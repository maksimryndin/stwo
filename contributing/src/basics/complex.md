# Complex numbers

Before digging into complex numbers, let's disuss a more general idea of *extension* in math. E.g. if we work with natural numbers (0, 1, 2, 3, ...)[^natural] \\(a, b \in \mathbb{N}\\) with addition operation defined (i.e. we get another natural number \\(c = a + b\\)) and we would like to define a subtraction (i.e. \\(c = a - b\\)) then it is not enough to have only natural numbers to deal with situations like \\(3 - 4\\). 

Let's think of single-coordinate vectors \\((a,)\\) and \\((b,)\\) existing on a line with *direction* which is evenly ticked with some basic measure of length (which we call \\(1\\)).

-----|-----|-----|-----|-----|-----|----->

So if we think about \\(3 - 4\\) as steps it means that we went forward \\(3\\) steps from original point \\(0\\) and went backward \\(4\\) and we are now \\(1\\) step before our original point. Introduction of vectors allowed us to bring a *direction* and the *extended* interpretation of \\(-\\) operation as a change of direction (besides merely diminishing a "magnitude"). And if we amplify the notion of step in \\(3 - 4\\) as \\(-1 \cdot (1,) = 3 \cdot (1,) - 4 \cdot (1,)\\), we can say that we expressed the vectors in terms of the *basis* unit vector \\((1,)\\).

So we *extend* the set of natural numbers \\(\mathbb{N}\\) to integers \\(\mathbb{Z}\\) by thinking of them as single-coordinate vectors (which have not only a magnitude but also a direction).

> We preserved all the elements of the original set \\(\mathbb{N}\\) and operations over them (namely, addition), i.e. are preserving the mathematical structure, while introducing new operations on an extended set \\(\mathbb{Z}\\) thus allowing us to deal with new situations like \\(3 - 4\\).


Now assume, we want to solve equations like \\(x^2 + 1 = 0, x \in \mathbb{R}\\) or in general finding square roots of negative real numbers like \\(\sqrt{-15}\\). It is evident that we cannot get the solution in terms of real numbers and we have to *extend* real numbers into something more "complex" (pun intended). We want the new superset \\(\mathbb{C}\\) of numbers (i.e. real numbers are included as a subset and preserve all operations over them) such that for all its elements common algebraic properties hold:

* Associativity: \\(z_0 + (z_1 + z_2) = (z_0 + z_1 ) + z_2\\) and \\(z_0 \cdot (z_1 \cdot z_2) = (z_0 \cdot z_1 ) \cdot z_2\\)
* Commutativity: \\(z_0 \cdot z_1 = z_1 \cdot z_0\\)
* Distributivity: \\(z_0 \cdot (z_1 + z_2) = z_0 \cdot z_1 + z_0 \cdot z_2\\)

where \\(z_0, z_1, z_2 \in \mathbb{C}\\).

It seems that real numbers fill the line without "holes" ([the completeness axiom](https://en.wikipedia.org/wiki/Completeness_of_the_real_numbers)) and so single coordinate vector representation is complete in this sense (sorry for such a frivolous treatment of serious math concepts). Probably we can extend the idea to the 2d space?


^
|
|
|
|
|
|
------------------->

So a complex number \\(z \in \mathbb{C}\\) can be represented as a pair of real numbers \\((x, y)\\). And analogous to our extension of natural numbers into integers we introduce basis vectors. One of them is already familiar \\(\mathbb{1} = (1, 0)\\) and the second (as we have a 2d space) is \\(\mathbb{i} = (0, 1)\\), an *imaginary* unit. 

We "imagine" that there is such a number \\(\mathbb{i}\\) that \\(\mathbb{i}^2 = -1\\) so it solves the equation \\(x^2 + 1 = 0\\).

If we express \\(z \in \mathbb{C}\\) in terms of basis unit vectors \\(\mathbb{1}\\) and \\(\mathbb{i}\\): \\(z = x \cdot \mathbb{1} + y \cdot \mathbb{i}\\) (so called *algebraic representation* of a complex number, \\(\mathbb{1}\\) is omitted), we can come up with the following *definitions* (thus the complex numbers are defined with such operations) of  

* addition \\(z_0 + z_1 = (x_0 + y_0 \cdot \mathbb{i}) + (x_1 + y_1 \cdot \mathbb{i}) = (x_0 + x_1) + (y_0 + y_1) \cdot \mathbb{i}\\) so we get a complex number \\((x_0 + x_1, y_0 + y_1)\\)
* multiplication \\(z_0 \cdot z_1 = (x_0 + y_0 \cdot \mathbb{i}) \cdot (x_1 + y_1 \cdot \mathbb{i}) = x_0x_1 + (x_0y_1 + y_0x_1) \cdot \mathbb{i} + y_0y_1\mathbb{i}^2\\) so we get a complex number \\((x_0x_1 - y_0y_1, x_0y_1 + y_0x_1)\\)

where \\(z_0 = (x_0, y_0)\\) and \\(z_1 = (x_1, y_1)\\), \\(\mathbb{i}^2 = -1\\)

Exercise: verify associativity, commutativity and distributivity of the defined addition and multiplication operations.

As such definitions of addition and multiplication ensure that algebraic properties of associativity, commutativity and distributivity hold, the set of real numbers \\(\mathbb{R}\\) is included in the form \\((x, 0)\\) (the x-axis of the complex plane), then with the numbers \\(z \in \mathbb{C}\\) we achieve our goal of having solutions for 

What is a unity element in case of complex numbers?

Exercise: find coordinate formulas for a zero element and a multiplicative inverse of the complex number (i.e. )

[^natural]: There is [an ongoing dispute](https://en.wikipedia.org/wiki/Natural_number#:~:text=In%20mathematics%2C%20the%20natural%20numbers,%2C%203%2C%20...%20.) either to include or exclude 0 (usually depends on the application). Here we treat them as values of unsigned integers (focusing on engineering applications).