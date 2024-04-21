# Fiat-Shamir and Schwartz-Zippel for Faster Emulated Pairings

:warning: Please see the file [Fiat_Shamir_and_Schwartz_Zippel_for_Faster_Emulated_pairings.ipynb](Fiat_Shamir_and_Schwartz_Zippel_for_Faster_Emulated_pairings.ipynb) for well formatted equations and code.

:warning: Find working Python example in [the Google colab](https://colab.research.google.com/drive/1G2XPsA8k6LFcQlqOc2zKeHtagydYukI9#scrollTo=B3ohT_qlAwGd) at https://colab.research.google.com/drive/1G2XPsA8k6LFcQlqOc2zKeHtagydYukI9#scrollTo=B3ohT_qlAwGd.

:warning: Find working verifier in Cairo [shramee/cairo_bn](https://github.com/shramee/cairo_bn/blob/main/src/schwartz_zippel.cairo).

## 1. Introduction

In the write-up [Faster Extension Field multiplications for Emulated Pairing Circuits](https://hackmd.io/@feltroidprime/B1eyHHXNT#fn3), Feltroid Prime discusses using Schwartz Zippel lemma and Fiat Shamir heuristics to optimize Field multiplications (Sections 1, 2 and 3).

Starting with representing $𝔽_{p12}$ (and equivalently $𝔽_{p6}$) as direct extensions in section 1. Then representing multiplication operation as a polynomial in section 2 (equation 1). Finally using Fiat-Shamir and the Schwartz-Zippel lemma in the circuit to verify the computation.

We will expand on this to apply it to larger computations. To start with we will try to adapt this for exponentiation (equation 2), with a goal of replacing entire final exponentiation.

## 2. Challenges

### 2.1 Controling polynomial degree

As we try to prove the larger computations, exponentiation of an $𝔽_{p12}$ by n, the degree of ${Q(x)}$ grows linearly with n.
With a large degree comes increased calldata size and computation costs for hashing each coefficient for Fiat-Shamir. To make things easier we prefer an optimised version for larger computations (equation 3).

### 2.2 Computation of Fiat-Shamir, $z$

Details regarding the computation of Fiat Shamir hueristic are presented here, [Section 3 of FEFMEPC](https://hackmd.io/@feltroidprime/B1eyHHXNT#3-Using-Fiat-Shamir-and-the-Schwartz-Zippel-lemma-in-circuit). 
To have our evaluations mod ${P(x)}$ we need to reduce our Fiat-Shamir hash, $z$, so that computation of ${P(z)} doesn't require reduction.

With ${P(z)}$ for modulo, bitsize of `z` from Fiat Shamir needs to be reduced to $1/(deg P(x))$ of maximum bitsize available for a modulo operation.


### 2.3. Size of coefficients

The size of the coefficients in bits grows linearly with $n$, the exponent.
This can be managed easily by modding, but here's the challenge.
1. We need the coefficients modulo curve's prime field $p$. Otherwise the result is not usable for us in pairing.
2. The evaluation requires a very specific modulus based on result of Fiat-Shamir otherwise the equation doesn't hold. Let's say we need it mod $z_{P} = P(z)$.

So we have the prover provide the coefficients mod $z_{P}*p$
This brings the size within twice the permissible integers in the circuit. For typical multiplication operation on a simulated field, this is usually required and available.

## 3. Prover and verifier

This section expands upon the process described in [Section 3 of FEFMEPC](https://hackmd.io/@feltroidprime/B1eyHHXNT#3-Using-Fiat-Shamir-and-the-Schwartz-Zippel-lemma-in-circuit).

### 3.1 Computing Fiat Shamir heuristic

Prover and verifier both need to compute this, here's an outline of the process.

1. Hash all coefficients of $A$ and $R$ mod $p$.
2. Reduce the $z$ to make sure $z_{P} = P(z)$ doesn't require reduction. For example for BLS12-381, bit size of $z$ should be under $384 / 12 = 32$ bits. Or when using a $F_{P6}$ Torus $384 / 6 = 64$ bits. Assuming support for 384 bit integers.

### 3.2 Prover's flow

* Prover computes the coefficients for $R$ mod $p$.
* Prover computes $z$ (Section 3.1) and $z_{P} = P(z)$.
* Prover re-computes coefficients for $R$ mod $z_{P}*p$

### 3.2 Verifier's flow

* Receives the coefficients of polynomial $R$ for the computation. The coefficients are provided modulo $z_{P}*p$
* Verifier receives the double-precision coefficients and reduces them separately with $z_{P}$ for evaluation and with $p$ to use in the circuit.
* Verifier computes $z$ (Section 3.1) from coefficients reduced mod $p$. And also $z_{P} = P(z)$
* Verifier evaluates $a_z = A(z) mod z_{P}$ and computes $a_z^n mod z_{P}$.
* Now verifier computes $r_z = R(z)$ using coefficients of R mod $z_{P}$
* Verify that  $a_z = r_z$

> *__According to the Schwartz-Zippel Lemma, if R is incorrect, the evaluated polynomials will differ with a very high probability.__*  
&ndash; Stolen from https://hackmd.io/@feltroidprime/B1eyHHXNT
