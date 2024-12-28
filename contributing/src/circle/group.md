# Group

The circle curve [can be treated](https://www.researchgate.net/publication/371339788_Reed-Solomon_codes_over_the_circle_group) as a group.

> Group  is an algebraic structure (less structured that Field) which has only one "addition" operation (as opposed to Field) which is associative (you can place brackets as you wish) and closed over the set of its elements. A neutral element is defined (think of it as 0 for addition) and every element of the group has an inverse element (if we "add" a group element with its inverse, we get the identity element).

The "addition" operation for the circle curve group is defined by 

\\((x_0, y_0) \cdot (x_1, y_1) = () \\)

and implemented by trait [`Add`](crates/prover/src/core/circle.rs) for `CirclePoint<F>`. You may have some confusion here as I do because the operation clearly resembles the mulitplication of complex numbers (compare!) and even uses the same multiplication dot symbol in the paper but let's persuade ourselves with the following: a group as an algebraic strucure has only a single binary operation so we just default using \\(+\\)" sign.


A neutral and inverse elements are implemented as `zero()` and `conjugate(&self)` methods of `CirclePoint<F>` accordingly (see also [`Neg`](crates/prover/src/core/circle.rs) trait implementation for `CirclePoint<F>`). The inverse operation is defined by 

\\(J(x, y) := (x, -y)\\) which is an *involution* (an operation which is inverse to itself).

Moreover, the circle curve group is a [cyclic group](http://abstract.ups.edu/aata/cyclic-section-cyclic-subgroups.html), i.e. all its elements can be obtained from a single one, called *generator*, by applying the group's operation (some number of times).

Circle points are placed of over the circle. And if we consider a zero as a start, then we can index

Exercise: Verify that the neutral element (1, 0) of the Circle group under group operation applied to it any positive number of times produces itself. Explain why it happens.

How can we find the generator of the Circle group? Basically, the generator is a `CirclePoint` 

We already know that the multiplicative subgroup should have all the elements of the Circle group and be *cyclic*. Being a cyclic group for the multiplicative group means that there exists such an element \\(g\\) (called a *generator*) which when raised to some power gives any other element of the group.

So the generator is a `CirclePoint` and the power to get some element of the circle group is a `CirclePointIndex`.

Let's find out the generator for `M31` field with a brute-force procedure:

crates/prover/src/core/circle.rs
```rust
// other imports assumed
use crate::core::fields::m31::{M31, P};
use super::M31_CIRCLE_GEN;

fn find_y(x: M31) -> M31 {
    // Exercise: Is the y coordinate
    // satisfying x^2 + y^2 = 1 unique?
    for y in (0..P).rev() {
        let y = M31::from(y);
        if x * x + y * y == M31::one() {
            return y;
        }
    }
    panic!("cannot find y-coordinate for x={x:?}");
}

fn find_m31_circle_generator() -> CirclePoint<M31> {
    // start with 2 as (1, 0) is the neutral element
    // which only produces itself on repeated group operation
    'generator_x: for x in 2..P {
        // The circle equation x^2 + y^2 = 1
        // determines the y-coordinate
        let x = M31::from(x);
        let y = find_y(x);
        // The potential generator
        // for every power should produce some non-identity
        // element of the group (otherwise after such a power
        // all futher elements are identity elements)
        let point = CirclePoint { x, y };
        let mut acc = point.clone();
        println!("checking {point:?} as a potential generator");
        for _index in 0..P - 1 {
            // 0..(2 ** 31 - 2) (or 2 ** 31 - 2 field elements exluding the neutral one)
            // apply group operation to generator `_index` times
            acc = acc + point;
            if acc == CirclePoint::zero() {
                continue 'generator_x;
            }
        }
        // Exercise: what is the result of acc + point
        // after the loop?
        return point;
    }
    panic!("cannot find circle group generator");
}

#[test]
fn circle_group_generator() {
    assert_eq!(M31_CIRCLE_GEN, find_m31_circle_generator());
}
```

Exercise: is the generator unique? If not, can you explain why and find other generators?