# Field



Primes of the form \\(p = 2^{n+1} - 1\\)[^mersenne] where  \\(n >= 1\\) are called Mersenne primes. In the binary form they are just \\(n\\) `1`s. The smallest is 7 (`n = 2`), \\(2^{31} - 1\\) fits `u32` and \\(2^{61} - 1\\) fits `u64`.



Exercise: benchmark the modulo reduction operation `%` and the specific implemented for Mersenne prime.

Exercise: Can \\(n+1\\) in the definition of Mersenne prime \\(p = 2^{n+1} - 1\\) be a composite number? (You may find useful [the factorization](https://proofwiki.org/wiki/Difference_of_Two_Powers))

Exercise: Show that from \\(p = 2^{n+1} - 1\\) it follows that \\(p \equiv 3 \textrm{mod}\ 4\\).

There is [a mathematical fact](https://en.wikipedia.org/wiki/Quadratic_reciprocity#q_=_%C2%B11_and_the_first_supplement) that prime fields with \\(p \equiv 3 \textrm{mod}\ 4\\) do not have such a field element \\(x\\) that \\(x^2 \equiv 1  \textrm{mod}\ p\\). So the polynomial f(x) = x^2 + 1 is *irreducible* (it mean it cannot be factored )

[^mersenne]: We use a notational convention of the paper and we narrow down the definition of CFFT-friendly prime which is \\(2^{n+1} * t - 1\\) for \\(n, t >= 1\\) - see *2 Notations and definitions* of the paper. For Mersenne primes you would usually see \\(M_n = 2^n - 1\\).