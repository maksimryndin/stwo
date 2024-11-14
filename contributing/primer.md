# CSTARK primer

## First steps

Clone the repo if not yet and try to run `./poseidon_benchmark.sh` in your shell. Everything should go smooth and you will see logs in the stdout. You would see terms like "Trace", "Interpolation", "Commitment", "Composition", "FRI". No worries if they are too vague now.

Check out the contents of `./poseidon_benchmark.sh` - which compilation features are used and what test is run. Let's check the contents of `test_simd_poseidon_prove`. Note from the name that this example already uses [SIMD instructions](https://github.com/rust-lang/portable-simd/blob/master/beginners-guide.md). As the name of the test also contains "prove", let's stop here a little. In ZK it is common to distinguish between proofs in a rigorous math sense and *arguments of knowledge* (probabilistically true). And that there are a *prover* (who produces a proof or an argument) and a *verifier* (who checks the proof/argument for truth).

> [Poseidon2](https://eprint.iacr.org/2023/323.pdf) is a hash function specifically designed to be used in proof systems (to have as least constraints as possible - more on it later - *arithmetization-oriented hash functions* or *circuit-friendly*). If we would use a more usual hash function like SHA256, it's representation involves much more gates to prove it (so the proving overhead is very significant). Hash functions are heavily used (especially in the web3 context) and are computations-heavy. So they are a natural target for optimized circuits and a natural target for benchmarks (as Poseidon2 hash function is widely used in Ethereum???).

Basically, we are re-running the same experiment as in [this video](https://x.com/StarkWareLtd/status/1807776563188162562) (the number of proofs in the video is 64 which means that LOG_N_INSTANCES=8 and each proof proves 2^16 poseidon hashes). In our benchmark LOG_N_INSTANCES=18.

Let's try to focus on a big picture for now. So we have two parties and two routines `prove_poseidon` and `verify`.

## Prover (`prove_poseidon`)

There are some precomputation steps (`SimdBackend::precompute_twiddles`) and protocol setup (a channel for non-interactivity, a polynomial commitment scheme) we will return to them later. Now let's focus on the *trace*.

### Trace (`gen_trace`)

So our goal here is to prove the computation of a poseidon2 hash function. The appoach taken in many proof systems is to define a machine with T registers (which track the computation state), execute the computation and record the computation trace as a journal of state changes (say N). So we have a matrix NxT of N rows (state changes) and T columns (registers which store state variables).

Let's look at the signature of `gen_trace`:

```rust
pub fn gen_trace(
    log_size: u32,
) -> (
    ColumnVec<CircleEvaluation<SimdBackend, BaseField, BitReversedOrder>>,
    LookupData,
)
```

It takes `log_size` - a logarithm of a number of rows `N`
The trace matrix is represented by the [`ColumnVec`](crates/prover/src/core/mod.rs) (length of a number of rows `N` - `2^log_size`) of columns [`Column`](crates/prover/src/core/backend/mod.rs). The trait implementation is parametrized by a backend (`SimdBackend`) and a field (`BaseField`). Let's start with the latter.

All operations with numbers are performed over some *field*. 

> Field is an algebraic structure over a set of some objects (*field elements* or *felts* for short) with defined operations of addition and multiplication closed over the set (i.e. adding or multiplying two felts results in the felt belonging to the same set); every non-zero element has a multiplicative inverse. For example, fields generalize the set of real numbers (R) and allow to deduce some general laws (e.g. the associative law) for felts of some general nature.

Why fields and not real numbers? Real numbers involve rounding errors due to their representation in a computer. Moreover, the result of an operation (addition/multiplication) is unbounded. 

Much more natural for current CPU architectures is a finite prime field, Fp. (A prime number p has no divisors except 1 and itself). All operations in such a field are done modulo p so the set of possible felts is {0, 1, ..., p-1}.

> Refresher on [The Integers mod n](http://abstract.ups.edu/aata/groups-section-mod-n-sym.html)


Mersenne primes (primes of the form `2^n - 1`) are of binary-friendly structure. E.g. modulo reductions (`n % p` where n is an integer) can be done using addition and bit operations (check [`impl M31`](crates/prover/src/core/fields/m31.rs)) as division operations are costly.  Mersenne prime `2^31 - 1` (usually denoted as M31) perfectly fits `u32` which means it can be efficiently operated by both CPU and GPU (as a 4-byte unsigned integer). [`BaseField`](crates/prover/src/core/fields/m31.rs) is M31 (in the current implementation it is a wrapper around `u32` with some field-specific methods).

In the `prove_poseidon` we can see `SimdBackend`. So the idea is to provide different backends ([cpu](./crates/prover/src/core/backend/cpu/), [simd](./crates/prover/src/core/backend/simd/), [GPU](https://github.com/NethermindEth/stwo-gpu)) to basic field operations. For example, in our the first method is [`precompute_twiddles`](./crates/prover/src/core/backend/simd/circle.rs) of the trait [`PolyOps`](./crates/prover/src/core/poly/circle/ops.rs).

> In our SIMD poseidon benchmark columns are vectors of `PackedBaseField` - Rust stdlib [`Simd`](https://doc.rust-lang.org/stable/std/simd/struct.Simd.html) structure which allow to operate with `[u32; N_LANES]` felts at once ([`N_LANES`](crates/prover/src/core/backend/simd/m31.rs) is 2^4, 16*32 is 512, in Intel it is AVX512). So basically trace columns are matrices themselves of N rows (as the trace) and 16 columns. But for the next part it is easier to think of it as vectors of length `N * 16` where we move by 16 rows at once.

We won't go into details how Poseidon2 works (see `apply_external_round_matrix`, `apply_internal_round_matrix`, `apply_m4` and the relevant paper). We just treat it as some permutations to the state.

In the main loop of `gen_trace` for every row (remember that we step by 16 simultaneous rows due to SIMD) and for every instance (out of 8) in row (columns ranges are relative to each instance)
1. copy the initial state values into the first 0..16 trace columns (16 values)
2. apply 4 full rounds of hash function logic to the state (16 values) and copy these updated state values into the next 16..80 trace columns (4*16 values)
3. apply 14 partial rounds of hash function logic to the first element of the state (1 value) and copy these updated state values into the next 80..94 trace columns (14 values)
4. apply 4 full rounds of hash function logic to the state (16 values) and copy these updated state values into the next 94..158 trace columns (4*16 values)

As we have 8 (`N_INSTANCES_PER_ROW`) instances in a row and 158 (`N_COLUMNS_PER_REP`) state registers per instance, then totally we have 1264 felts (`N_COLUMNS`) in a row of the trace. As a reminder, every felt consists of 16 `u32` values (as it is a simd version).

After the loop we see the evaluation of the trace over [`CircleDomain`](crates/prover/src/core/poly/circle/domain.rs).

The input to the poseidon2 hash function matches the expected output iif the algebraic intermediate representation (AIR) polynomial constraints are satisfied.

### Circle curve

Integrity of the original computation is transformed into proving that the AIR constraints are satisfied. The process of reducing the computation trace to the set of the polynomial constraints is named *arithmetization*.

So we treat each column of the computation trace as values of a polynomial over the `CircleDomain`.

When we define constraints, we can move both along the columns and along the rows.
Then every constraint (which may include values of several polynomials) is representated as a rational function (think of moving along the rows).
The variables are column polynomials.

If we have a solution to the system of constraints (i.e. we know column polynomials), it means that we now the solution of the original computation program. But we can say that we have a solution but provide another object instead of polynomials (e.g. rational fractions). So we also need to prove that solution consists of polynomials.

So we treat the trace columns as an image (values) of a polynomial fi defined of the domain over the unit circle. Informally, fi: CircleDomain -> Column
As a quick refresher on domain and image of the mapping(function) see [Cartesian Products and Mappings](http://abstract.ups.edu/aata/ssets-ection-sets-and-equivalence-relations.html).

The circle curve is defined by the equation x^2 + y^2 = 1, where x and y come from the finite prime field. At the current stage of the proof the field is M31. The circle curve is represented by [`struct CirclePoint`](crates/prover/src/core/circle.rs) and 

The circle curve can be treated as a group.

> Group  is an algebraic structure (less structured that Field) which has only one "addition" operation (as opposed to Field) which is associative (you can place brackets as you wish) and closed over the set of its elements. An identity element is defined (think of it as 0 for addition) and every element of the group has an inverse element (if we "add" a group element with its inverse, we get the identity element).

The "addition" operation for the circle curve group is defined by [`Add`](crates/prover/src/core/circle.rs) trait implementation for `CirclePoint<F>`. Identity and inverse elements are implemented as `zero()` and `conjugate(&self)` methods of `CirclePoint<F>` accordingly.

Moreover, the circle curve group is a [cyclic group](http://abstract.ups.edu/aata/cyclic-section-cyclic-subgroups.html), i.e. all its elements can be obtained from a single one, called *generator*.

Circle points are place of over the circle. And if we consider a zero as a start, then we can index 

> [Groups](http://abstract.ups.edu/aata/groups-section-defnitions.html)
> [Subgroups](http://abstract.ups.edu/aata/groups-section-subgroups.html)
> [Coset](http://abstract.ups.edu/aata/cosets-section-cosets.html)

If we follow the nested structs of `CanonicCoset::new(log_size).circle_domain()`, we come up to [`Coset`](crates/prover/src/core/circle.rs).

Coset is a set which is obtained 


CircleDomain
CanonicCoset









Every column polynomial is actually evaluated over a larger domain (called evaluation domain). The evaluation is refered to as trace Low Degree Extension (LDE). Extension domain points don't belong to the original trace domain (i.e. the trace domain is disjoint with the added points but at the same time the trace domain is a subset of the evaluation domain). The ratio between the evaluation domain and the trace domain is a *blowup* factor. (LDE is a Reed-Solomon codeword of the trace).


The goal of using such a small prime field is efficiency (CPU is optimized word-sized operations). But for the security we need a large field (so we use extension fields).

How to compute the trace LDE?
First, the interpolation polynomial for every trace colum (trace domain) is calculated with IFFT.
Second, the interpolated polynomial is evaluated on the evaluation domain with FFT.

To obtain the extension domain points which should not coincide with those which are in the trace domain, a non-unit coset of the multiplicative subgroup of size (blowup * N).

The prover commits to the trace LDE with Merkle tree.

As a constraint is composed of our column polynomials (every of each is of degree < N and evaluated over the extended domain), the constraint itself should be of low degree.
So the composition of all constraints polynomials should also be of low degree.
The verifier supplies the random weight of the composition.
The prover interpolates this Composition Polynomial and commits to it.
Now the evaluation happen over the extended field (due to the security issue).
What is the extension field? (complex numbers as an example), improves security.

The evaluation of the composition polynomial are again encoded as constraints (differences between column polynomial values and composition polynomial values divided by roots???).
The new constraints should be of low degree.

Low degree testing is achieved with FRI (Fast Reed-Solomon Interactive Oracle Proof of Proximity)
Recursive procedure of halving the polynomial degree (interleaved with random challenges from the verifier) (TODO an alternative to FRI is STIR https://www.notion.so/nethermind/Stwo-and-variants-132360fc38d080bca3e2fa8c4f8f9079)

One of the first struct you see is [`PcsConfig`](crates/prover/src/core/pcs/mod.rs) which stands for `Polynomial Commitment Scheme configuration`. 

What is a *commitment*? For example, you want play a guess-a-number game with your friend. Your friend comes up with a number and you try to guess it. How can you enforce your friend to stay honest and not to change the number after you have answered? Both of you can agree on a collision-resistant hash function (say, sha256) and the following protocol:
1. (commit phase) Your friend comes up with a number `x`, gets the hash `sha256(x||r)` of the number `x` concatenated with a random value (salt) `r` and tells you a hash of it `0x...` and `r` while keeping the number secret, thus *committing* to the number `x`.
2. (check phase) You guess the number and your friend (the *commiter*) *reveals* the number `x`. You can understand also you're right even without asking your friend - just calculating the hash of it concatenated with `r` and comparing it with the provided one (thus removing the *interaction*).

Why do we need a salt `r`? At least, due to two reasons:
1) It helps to produce different hash values for the same input `x` at different game rounds (i.e. it prevents you from learning the patterns and based on that manipulating - *non-malleability*)
2) It doesn't allow you to collect the history to organize lookup tables and brute-force

Our basic commitment scheme:
1) the committer cannot open the commitment to another value `x'` (*binding*)
2) the commitment itself doesn't reveal any information about the value `x` (*hiding*) - your friend doesn't want you to somehow deduce the value from the commitment.
3) the commitment guarantees that the committer knew the value of `x` at the commitment time (*extractability*)

A clever generalization of our basic commitment scheme is a Merkle tree (usually a binary one) (from a single value `x` to a vector of values). Moreover, the Merkle commitment scheme allows to cheaply open a part of the message so it also generalizes a check procedure for a single value.

partially updatable (O(log2|m|)), if no privacy is ok - then no salts (no need for hiding, only data integrity)






Hash function as a heuristic for ROM (Random Oracle Model).
(Perfect) completeness
STARK is a SNARG (succinct non-interactive argument) in the ROM with a knowledge soundness (an honest prover also convinces that he knows the witness). As we use a hash function, we have no secret randomness (i.e. it is a public-coin protocol).

public coin - roughly speaking the verifiers challenges are random and independent from each other, and public (no hidden secret randomness)

Fiat-Shamir transform - to use the random oracle (heuristically represented by a cryptographically secure hash function) and to provide a challenges from the verifier as an output from the oracle from an input which includes all the interaction up to the random oracle call and a random salt (due to the similar reasons as for the hash commitment scheme above) (basically, thus the prover commits to all the prover's messages so far sent and salts). Thus to forge the prove basically requires the prover to be able to find collisions in the hash function implementation. The verifier uses the same oracle to obtain its "random" challenges.

Hashing the whole previous history of interactions increases the input to the hash function significantly, so the hash chain is actually built. The hash includes the previous hash (Merkle-Damgard construction)


# Task

Implement a Poseidon2 benchmark with the CPU backend and compare the performance with the simd version.


# References
1. [Beginner's Guide To SIMD](https://github.com/rust-lang/portable-simd/blob/master/beginners-guide.md) (accessed in November 2024)
2. [Portable SIMD Programming in Rust](https://calebzulawski.github.io/rust-simd-book/) (accessed in November 2024)
3. [Poseidon2: A Faster Version of the Poseidon Hash Function](https://eprint.iacr.org/2023/323) (accessed in November 2024)
4. [Building Cryptographic Proofs from Hash Functions](https://snargsbook.org/) (accessed in November 2024)
5. [ethSTARK Documentation](https://eprint.iacr.org/2021/582) (accessed in November 2024)
6. [Hardware Acceleration for Zero Knowledge Proofs](https://www.paradigm.xyz/2022/04/zk-hardware) (accessed in November 2024)
7. [Exploring circle STARKs](https://vitalik.eth.limo/general/2024/07/23/circlestarks.html) (accessed in November 2024)
8. [Why I’m excited by Circle STARK and Stwo](https://elibensasson.blog/why-im-excited-by-circle-stark-and-stwo/) (accessed in November 2024)
9. [Abstract Algebra: Theory and Applications](http://abstract.ups.edu/aata/aata-toc.html) (accessed in November 2024)