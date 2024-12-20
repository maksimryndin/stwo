# CFFT

Circle FFT (CFFT) actually may look quite different from the previous section simplified algorithm but it draws same ideas.

* We work with bivariate polynomials (as opposed to univariate case)
* FFT domain is a circle group. At every recursive step the domain is a twin coset (i.e. a subgroup of the circle group). The group operation allows mapping 2-to-1.
* Even and odd parts of polynomials are defined according to involution J