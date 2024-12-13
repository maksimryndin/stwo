# Motivation


Let's consider first the idea of fast polynomial evaluation. We have 3-degree (cubic) polynomial 
\\(f(x) = c_0 + c_1 * x + c_2 * x^2 + c_3 * x^3\\) and we want to evaluate it at 4 points: 
\\(x_0, ..., x_3\\). A direct approach would involve \\(N^2\\) operations. 

Let's represent it in the matrix form:

$$
\begin{bmatrix}
f(x_0)\\\\
f(x_1)\\\\
f(x_2)\\\\
f(x_3)
\end{bmatrix} =
\begin{bmatrix}
1 & x_0 & x_0^2 & x_0^3\\\\
1 & x_1 & x_1^2 & x_1^3\\\\
1 & x_2 & x_2^2 & x_2^3\\\\
1 & x_3 & x_3^2 & x_3^3
\end{bmatrix}
\begin{bmatrix}
c_0\\\\
c_1\\\\
c_2\\\\
c_3
\end{bmatrix}
$$

Let's reorder a little the polynomial: 
\\(f(x) = c_0 + c_2 * x^2 + c_1 * x + c_3 * x^3 =  c_0 + c_2 * x^2 + x * (c_1 + c_3 * x^2) = f_e(x^2) + x * f_o(x^2)\\). 
We grouped even- and odd-degree monomials (terms \\(c_i * x^i\\)). In the matrix form 
it should look like (note that we reordered matrix columns and entries of the coefficients vector):

$$
\begin{bmatrix}
f(x_0)\\\\
f(x_1)\\\\
f(x_2)\\\\
f(x_3)
\end{bmatrix} =
\begin{bmatrix}
1 & x_0^2 & x_0 & x_0^3\\\\
1 & x_1^2 & x_1 & x_1^3\\\\
1 & x_2^2 & x_2 & x_2^3\\\\
1 & x_3^2 & x_3 & x_3^3
\end{bmatrix}
\begin{bmatrix}
c_0\\\\
c_2\\\\
c_1\\\\
c_3
\end{bmatrix}
= \begin{bmatrix}
1 & x_0^2\\\\
1 & x_1^2\\\\
1 & x_2^2\\\\
1 & x_3^2
\end{bmatrix}
\begin{bmatrix}
c_0\\\\
c_2\\\\
\end{bmatrix}
+
\begin{bmatrix}
x_0 & x_0^3\\\\
x_1 & x_1^3\\\\
x_2 & x_2^3\\\\
x_3 & x_3^3
\end{bmatrix}
\begin{bmatrix}
c_1\\\\
c_3
\end{bmatrix} =
$$

To extract a common \\(x\\) in the matrix form we can use an identity matrix:

$$
= \begin{bmatrix}
1 & x_0^2\\\\
1 & x_1^2\\\\
1 & x_2^2\\\\
1 & x_3^2
\end{bmatrix}
\begin{bmatrix}
c_0\\\\
c_2\\\\
\end{bmatrix}
+
\begin{bmatrix}
1 & 0 & 0 & 0\\\\
0 & 1 & 0 & 0\\\\
0 & 0 & 1 & 0\\\\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x_0\\\\
x_1\\\\
x_2\\\\
x_3
\end{bmatrix}
\begin{bmatrix}
1 & x_0^2\\\\
1 & x_1^2\\\\
1 & x_2^2\\\\
1 & x_3^2
\end{bmatrix}
\begin{bmatrix}
c_1\\\\
c_3
\end{bmatrix} =
$$

(Stop and verify the transformations here!). Hence we reduced the number of operations but the algorithmic complexity is still quadratic (verify). Can we come up with some clever divide-and-conquer algorithm? If we introduce a constraint that under squaring our 4 evaluation points become 2 other points, e.g. \\(x_0^2 = x_2^2\\) and \\(x_1^2 = x_3^2\\) (we could also require instead \\(x_0^2 = x_1^2\\) and \\(x_2^2 = x_3^2\\) or other combinations), so (up to a permutation of the points) we could have (horizontal line highlights the blocks):

$$
= \begin{bmatrix}
1 & x_0^2\\\\
1 & x_1^2\\\\
\hline
1 & x_0^2\\\\
1 & x_1^2
\end{bmatrix}
\begin{bmatrix}
c_0\\\\
c_2\\\\
\end{bmatrix}
+
\begin{bmatrix}
1 & 0 & 0 & 0\\\\
0 & 1 & 0 & 0\\\\
0 & 0 & 1 & 0\\\\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x_0\\\\
x_1\\\\
x_2\\\\
x_3
\end{bmatrix}
\begin{bmatrix}
1 & x_0^2\\\\
1 & x_1^2\\\\
\hline
1 & x_0^2\\\\
1 & x_1^2
\end{bmatrix}
\begin{bmatrix}
c_1\\\\
c_3
\end{bmatrix}
$$

The scheme allows us divide the problem of size N (in our case 4) into 2 subproblems of size N/2 (in our case 2) and so on, thus recursively finding the solution. 

At the current step we recurse 

At the final step we have an evaluation at one point. Let's picture the recursion tree:

```
             N
           /   \
         N/2   N/2
        / \    / \ 
            ...
    /\   /\   ... /\
   1  1 1  1     1  1
```

The height of the tree is \\(log_2 N\\) and we have \\(O(N)\\) operations at every tree level (\\(N\\) additions and \\(N\\) multiplications). So the total complexity is \\(O(N * log_2 N)\\). Take your time to appreciate the beauty of it!

Let's summarize on what has helped us here[^assumptions]:
1) The unlimited divisibility by 2 (so called "2-addicity") - we can decrease the problem of size \\(2^n\\) into 2 subproblems of size \\(2^{n-1}\\)
2) The half-reduction of evaluation domain by squaring operation (the original domain is mapped into half of it and this property is preserved along all recursion steps)

// crates/prover/src/core/poly/circle/domain.rs
```rust
#[test]
fn test_circle_domain_for_fft() {
    let log_size = 2; // N = 4
    let coset = Coset::half_odds(log_size);
    let domain = CircleDomain::new(coset);
    domain.iter().for_each(|circle_point| {
        println!("Square of {:?} is {:?}", circle_point, circle_point.mul(2))
    });
}
```

The squaring operation \\(\pi\\) is more exotic in the case of `CirclePoint`.

Now you should understand why we need `CircleDomain` - and you should be able to mostly understand *1 Introduction* of the paper.

Your may disagree that we enforced constraints on the evaluation domain (but our initial goal was to evaluate at some prescribed points). But actually what we are looking for here, is the inverse operation. We can define n-degree polynomials via its n+1 coefficients and equivalently via n+1 evaluations (2 points define a line c_0 + c_1 * x). So the inverse operation to our matrix multiplications could lead us to fast *interpolation*, i.e. finding the polynomial which goes through the points 

[^assumptions]: If under the assumption 2 we use roots of unity, we get a classical Radix-2 Cooley-Tuckey FFT algorithm and evaluations/interpolations over the real domain \\(R\\).