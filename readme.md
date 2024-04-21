# Fiat-Shamir and Schwartz-Zippel for Faster Emulated Pairings

:warning: Please check out the file [Fiat_Shamir_and_Schwartz_Zippel_for_Faster_Emulated_pairings.ipynb](Fiat_Shamir_and_Schwartz_Zippel_for_Faster_Emulated_pairings.ipynb) for code and up to date notes.

### 1. Introduction

In the write-up [Faster Extension Field multiplications for Emulated Pairing Circuits](https://hackmd.io/@feltroidprime/B1eyHHXNT#fn3), Feltroid Prime discusses using Schwartz Zippel lemma and Fiat Shamir heuristics to optimize Field multiplications (Sections 1, 2 and 3).

Starting with representing $𝔽_{p12}$ (and equivalently $𝔽_{p6}$) as direct extensions in section 1. Then representing multiplication operation as a polynomial in section 2 (equation 1). Finally using Fiat-Shamir and the Schwartz-Zippel lemma in the circuit to verify the computation.

> ${A(x)*B(x) = Q(x)*P(x) + R(x)}$ &emsp; &emsp; &emsp;${(1)}$

We will expand on this to apply it to larger computations for greater cost savings.
