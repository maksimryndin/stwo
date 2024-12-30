# Circle


\\(S_1 = \\{(x, y) \in F^2: x^2 + y^2 = 1\\}\\)

Multiplicative group of Fp (excluding 0)

Exercise: write a small program to factor the size of a multiplicative group (2^31 - 1 - 1) into prime factors? What degree of 2 do you observe?

[^fields]: original STARKs used a big prime field (which you can see in some older tutorials devoted to STARKs). The *smooth* field allows for efficient modular arithmetics and at the same time is FFT-friendly (we will see later what it means). The key innovation behind Circle STARKs is the use of the circle which allowed to use so called Mersenne prime (a natural fit for `u32`) which is smooth in regards to field operations (modulo reduction being done via bit shifting -- todo - check) but is not FFT-friendly. Circle group over Mersenne prime field makes the latter FFT-friendly.