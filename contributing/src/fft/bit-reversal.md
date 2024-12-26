# Bit-reversal

One of the basic building blocks in FFT-like algorithms is grouping polynomials coefficients (and twiddles) accordingly to their monomials even and odd degrees. How can we do it in code? There is a general technique called *bit-reversal* applied to sequences of numbers of length \\(2^n\\). In the previous example we had \\(c_0, c_1, c_2, c_3\\) with indices \\(0, 1, 2, 3\\) and we wanted to obtain a *permutation* (i.e. the same elements but in a different order) \\(c_0, c_2, c_1, c_3\\) with indices \\(0, 2, 1, 3\\).

Let's write indices of the original sequence and the target permutation in a binary form (assuming 2 bits to store):

\_________________

0  | 00 | 0  | 00 |

\_________________

1  | 01 | 2  | 10 |

\_________________

2  | 10 | 1  | 01 |

\_________________

3  | 11 | 3  | 11 |

\_________________

Exercise: make the same table for the sequence \\(0, 1, .., 7\\). Note on the order of indices in even and odd groups of indices.

Let's think why it works. The logic is both simple and ingenious at the same time: having \\(n\\) bits to represent an unsigned integer of the index, we have the first \\(2^n-1\\) integers starting with 0 in their binary form and the second half \\(2^n-1\\) integers starting with 1. So when we reverse the bits, all even indices (originally ending with 0) go into the first half and all odd indices (originally ending with 1) go into the second half. The uniqueness of the indices and invertability of the bit reversal operation guarantee that it is indeed a permutation and at the same time an *involution* (i.e. an operation which is an inverse to itself - being applied an even number of times returns the original sequence).

We can find a naive straightforward implementation of `bit_reverse` in `crates/prover/src/core/utils.rs`[^fast-bit-reverse].

[^fast-bit-reverse]: there are optimized versions of the algorithm (e.g. articles like [this](https://folk.idi.ntnu.no/elster/pubs/elster-bit-rev-1989.pdf) or [this](https://arxiv.org/pdf/1708.01873)) which can be possibly explored to improve STWO performance.
