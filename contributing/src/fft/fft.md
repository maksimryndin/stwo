# FFT

So our idea is to evaluate polynomials represented by columns of the trace - i.e. for every column i we need to find such a polynomial P that it goes through all the points (talk about x and y of the points here).
Approach 1 - system of linear equations and Gauss elimination
Approach 2 - Lagrange interpolation.
Approach 3 - IFFT

First, let start with 2 forms of polynomial representation. For the polynomial 

x = MX, where X is an ith column of the trace, a row of matrix M is row numeration of the trace. If we approach the problem

FFT as a divide and conquer algorithm, master theorem, Cooley-Tuckey, DFT,
change of basis (Vandermonde matrix)

DFT[x(k)]
IDFT[X(k)] for the sequence of evaluations X(0), X(1), ..., X(N)

NTT is a DFT over a finite field of prime size (DFT is defined over the complex numbers or complex field)
NTT beacuse of the field arithmetic, doesn't involve any rounding errors.

Coefficient representation -- evaluation --> Value representation
Value representation -- interpolation --> Coefficient representation

We can evaluate with
* solving a system of linear equations
* lagrange interpolation
* DFT

The key idea behind FFT is to reuse calculations (compare with Karatsuba multiplication).

