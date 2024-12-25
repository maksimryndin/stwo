# CFFT

Circle FFT (CFFT) actually may look quite different from the previous section simplified algorithm but it draws same ideas.

* We work with bivariate polynomials (as opposed to univariate case where we work in *monomial basis*, i.e. \\(1, x, x^2, ...\\)) - rows of the matrix
* FFT domain is a circle group. At every recursive step the domain is a twin coset (i.e. a subgroup of the circle group). The group operation allows mapping 2-to-1. The squaring operation is \\(2x^2 - 1\\).
* Even and odd parts of polynomials are defined according to involution J

Likewise in a classical FFT, we split the univariate polynomial into even and odd parts, we can do the same in our polynomial space  \\(F[x, y]/(x^2 + y^2 - 1)\\) relying on the fact \\(y^2 = 1 - x^2\\).