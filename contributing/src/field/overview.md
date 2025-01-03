# Field

All operations with numbers are performed over some prime *field*. 

> Field is an algebraic structure over a set of some objects (*field elements* or *felts* for short) with defined operations of addition and multiplication closed over the set (i.e. adding or multiplying two felts results in the felt belonging to the same set); every *non-zero* element has a multiplicative inverse. 

Examples of fields:
* the set of real numbers with addition and multiplication \\((\mathbb{R}, +, \cdot)\\)
* the set of complex numbers with addition and multiplication \\((\mathbb{C}, +, \cdot)\\)
* the set of modular some prime modulo

Let's focus on the latter example of a field as it is much more natural fit for current CPU architectures is a finite prime field, Fp. (A prime number p has no divisors except 1 and itself). All operations in such a field are done modulo p so the set of possible felts is {0, 1, ..., p-1}.

Why fields and not real numbers? Real numbers involve rounding errors due to their representation in a computer. Moreover, the result of an operation (addition/multiplication) is unbounded for real numbers.

Primes of the form \\(p = 2^{n+1} - 1\\)[^mersenne] where  \\(n >= 1\\) are called *Mersenne primes*. In the binary form they are just \\(n\\) `1`s. The smallest is 3 (`n = 1`); \\(2^{31} - 1\\) fits `u32` and \\(2^{61} - 1\\) fits `u64` and there're others less relevant here.

Exercise: benchmark the modulo reduction operation `%` and the specific one implemented for Mersenne prime.

Exercise: Can \\(n+1\\) in the definition of Mersenne prime \\(p = 2^{n+1} - 1\\) be a composite number? (You may find useful [the factorization](https://proofwiki.org/wiki/Difference_of_Two_Powers))

Exercise: Show that from \\(p = 2^{n+1} - 1\\) it follows that \\(p \equiv 3 \textrm{mod}\ 4\\).

There is [a mathematical fact](https://en.wikipedia.org/wiki/Quadratic_reciprocity#q_=_%C2%B11_and_the_first_supplement) that prime fields with \\(p \equiv 3 \textrm{mod}\ 4\\) do not have such a field element \\(x\\) that \\(x^2 \equiv 1  \textrm{mod}\ p\\). So the polynomial f(x) = x^2 + 1 is *irreducible* (it means it cannot be factored )

In the basics chapter we considered polynomials in real values, i.e. \\(x \in \mathbb{R}) and addition/multiplication operations are defined in a familiar way as we think about numbers. But as we already discussed fields and we know that \\(\mathbb{R}\\) is a *field*, what happens when we replace \\(\mathbb{R}\\) with something more exotic like prime integer fields?

[^mersenne]: We use a notational convention of the paper and we narrow down the definition of CFFT-friendly prime which is \\(2^{n+1} * t - 1\\) for \\(n, t >= 1\\) - see *2 Notations and definitions* of the paper. For Mersenne primes you would usually see \\(M_n = 2^n - 1\\).