---
title: Fixing a qubit channel by two Pauli operators
categories: [学术随笔 | Scholarly Notes]
tags: [量子计算 | Quantum Computation, 量子纠错 | Quantum Error Correction, 实践量子纠错探微 | Subtleties in Practical QEC, 量子信息 | Quantum Information]
math: true
---

A common argument in quantum error correction is like, "this series of operations maps a logical $Z$ to something, and a logical $X$ to something, so it acts like some particular channel at the logical level." However, the reason behind this argument is not that trivial.

The problem is equivalent to the following one: if a channel leaves $X$ and $Z$ invariant, can we conclude that it is the identity channel? Linearity alone is not enough, since $X$ and $Z$ do not form a complete basis of the operator algebra.

## A simple derivation

> **Lemma**
> 
> Let $\mathcal{E}$ be a channel (CPTP map) over a qubit. If Pauli operator $Z$ is invariant under $\mathcal{E}$, then the identity $I$ is also invariant.
{: .prompt-info }

> *Proof.* Consider pure states $(I\pm Z)/2$ under channel $\mathcal{E}$. The results are $(\mathcal{E}(I) \pm Z)/2$.
> Positivity and trace preserving of the channel then requires $\mathcal{E}(I) = I$. In fact, we have $\braket{0|\mathcal{E}(I)|0}\geq 1$ and $\braket{1|\mathcal{E}(I)|1}\geq 1$, but $\operatorname{tr} \mathcal{E}(I) = \operatorname{tr} I = 2$, so $\braket{0|\mathcal{E}(I)|0} = \braket{1|\mathcal{E}(I)|1} = 1$. 
> Furthermore, $0 \leq (\bra{0}+\lambda^{*} \bra{1})(\mathcal{E}(I)+Z)(\ket{0}+\lambda \ket{1})=2+(\lambda \braket{0|\mathcal{I}|1}+\text{c.c.})$ holds for any $\lambda$. So $\braket{0|\mathcal{E}(I)|1} = \braket{1|\mathcal{E}(I)|0} = 0$. This shows that $\mathcal{E}(I) = I$.

> **Lemma**
> 
> Let $\{K_{i}\}$ be the set of Kraus operators of a channel $\mathcal{E}$. If a pure state $\ket{\psi}$ is invariant under $\mathcal{E}$, then it is an eigenstate of all $K_{i}$ (with possibly different eigenvalues).
{: .prompt-info }

> *Proof.* By definition, $\sum_{i}K_{i}^{\dagger}K_{i}=I$, 
> so $\sum_{i}\braket{\psi|K_{i}^{\dagger}K_{i}|\psi} = 1$. 
> On the other hand, since $\ket{\psi}$ is a fixed point, we have $1 = \sum_{i}\braket{\psi|K_{i}|\psi}\braket{\psi|K_{i}^{\dagger}|\psi}=\sum_{i}|\braket{\psi|K_{i}|\psi}|^{2}$. 
> But according to Cauchy-Schwarz inequality, $|\braket{\psi|K_{i}|\psi}|^{2} \leq \|\ket{\psi}\|^{2} \|K_{i}\ket{\psi}\|^{2}$, and equality holds if and only if $K_{i}\ket{\psi}\propto \ket{\psi}$. 
> Summing over $i$ we see both sides equal to 1. So the equality condition holds, and that $\ket{\psi}$ is an eigenstate of all $K_i$.

> **Theorem**
> 
> Let $\mathcal{E}$ be a channel over a qubit. If Pauli operators $Z$ and $X$ are invariant under $\mathcal{E}$, then the channel is the identity channel.
{: .prompt-info }

> *Proof.* From invariance of $Z$ we have invariance of $I$ and hence of $(Z\pm I)/2$, which are two pure states $\ket{0}, \ket{1}$. 
> There two pure states are thus eigenstates of all the Kraus operators, so each $K_{i}$ is a diagonal matrix under computational basis. 
> But $X$ is also invariant, leading to $\ket{+}, \ket{-}$ being eigenstates of the Kraus operators. 
> So these Kraus operators can only be multiple of identity. 
> The CPTP channel is therefore identity.

## On complete positivity

Note that in the derivation above, the first lemma requires only positivity. It is the second lemma that requires complete positivity, which is encoded in the form of Kraus operator. We can see from another point of view why complete positivity is necessary for our result.

Linearity and trace preserving properties make a channel map the Block sphere to an ellipsoid (including degenerate forms like spheres or points). In fact, it can only map $(I + \boldsymbol{x}\cdot\boldsymbol{\sigma})/2$ to $(I + f(\boldsymbol{x})\cdot\boldsymbol{\sigma})/2$, where $f$ is a linear map. Now positivity requires the result state to be in the Bloch sphere. So with positivity (and linearity and trace preserving) alone, we can only say that a channel maps the Block sphere to an ellipsoid inside the sphere. 

This constraint cannot rule out the possibility that there is a channel that keeps the $x-z$ plane invariant, but squeezes along the $y$ direction. So with the first lemma, we cannot conclude that the channel we consider is identity.

To see how complete positivity matters explicitly, consider the Choi map.[^1] We have a channel $\mathcal{E}$ under which $I, X, Z$ are invariant operators. Suppose now $\mathcal{E}(Y)=\lambda Y$ with $\lambda \leq 1$. The Choi matrix is then

$$
C = (I\otimes \mathcal{E})(\ket{\Phi^{+}}\bra{\Phi^{+}}) = \frac{1}{4}(I+X_{1}X_{2}-\lambda Y_{1}Y_{2} + Z_{1}Z_{2}),
$$

where $\ket{\Phi^{+}} = (\ket{00}+\ket{11})/\sqrt{ 2 }$ is the Bell pair, whose density matrix is $(1+Z Z)(1+ X X)/4$. The matrix form is 

$$
C = \frac{1}{4} \begin{pmatrix} 2 & 0 & 0 & 1+\lambda \\ 0 & 0 & 1-\lambda & 0 \\ 0 & 1-\lambda & 0 & 0 \\ 1+\lambda & 0 & 0 & 2 \end{pmatrix}.
$$

Note that the central $2\times 2$ block have eigenvalues $\pm (1-\lambda)$. So $C$ is positive if and only if $\lambda = 1$. So squeezing along $y$ is impossible if $x-z$ plane has been fixed.

[^1]: If you know nothing about Choi map, it doesn't matter, because here we do not use the non-trivial part that complete positivity of a channel is *equivalent* to positivity of its Choi map; We only use the definition of complete positivity and call Choi's construction as a testing example.