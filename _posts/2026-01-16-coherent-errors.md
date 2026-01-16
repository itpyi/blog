---
title: Can quantum error correcting codes handle coherent errors?
categories: [学术随笔 | Scholarly Notes]
tags: [量子计算 | Quantum Computation, 量子纠错 | Quantum Error Correction, 实践量子纠错探微 | Subtleties in Practical QEC]
math: true
# image: /assets/img/251231/growth-cover.svg
---
Analysis of quantum error correcting (QEC) codes is usually based on random Pauli errors, which form decoherence channels. This sometimes causes confusion on whether QEC codes can protect against "coherent" errors, such as a small unitary evolution on the qubits. In this blog post, we show that in general, people do not need to worry about coherent errors.

## Linearity of correctable errors

A common theoretical tool for analysing error correction is to separate errors as correctable and uncorrectable for a given code. This analysis does not depend on the error model. After that, when an error model is given, one further analyses the proportion of uncorrectable errors in the error model, which gives an estimation of the logical error rate. 

Consider the first step of separating errors into two classes. A class of error is said to be correctable if there is a recovery channel $\mathcal{R}$ (practically speaking, a decoder) that cancels the effect of the error. A subtlety here is that the error is unknown, so the recovery channel should be independent of the errors. Therefore, it only makes sense to talk about correctable and uncorrectable errors for a given code *and* a given recovery channel. (Since correctablity depends on the recovery channel, needless to say that this channel should be designed based on the knowledge of the error model in practice. This does not contradict our logic here of *analysing* correctable errors.)

An important property of correctable errors for a code and a channel is that they form a linear space of linear maps. That is, if $E_{1}$ and $E_{2}$ are correctable (acting as $\rho \mapsto E \rho E^{\dagger}$) by $\mathcal{R}$, then any linear combination of $E_{1}$ and $E_{2}$ is correctable by $\mathcal{R}$. It is this linear property that allows us to analyse Pauli errors only, since they form a basis of the operator algebra. By analysing Pauli errors, we can figure out the correctable space of error operators. Then even if the actual errors are coherent, one can expand them over the Pauli basis and analyse the weight of correctable and uncorrectable errors. Usually, local Pauli errors are correctable. So as long as the coherent errors are still local, uncorrectable terms can only be present with an exponentially low weight. We will demonstrate this picture by some examples later.

As a theoretical side comment, note that there is a maximal space of correctable errors for which there is an error-independent recovery channel. This can be understood roughly by the classical picture: if an error cannot make two codewords looks the same, then it can always be corrected. This leads to the Knill-Laflamme (K-L) condition for QEC. For pure theoretical analysis, one often refer to the operator space satisfying the K-L condition as correctable errors. One may also talk about correctability of error channels rather than a space of linear maps, for which a similar K-L condition is of vital importance. We would like to remind the readers that all these viewpoints share the same mechanism, but can be applied to different situations. One should be clear about what is being talking about.

## Emergence of decoherence

In this section, we show that incoherent error models generally describe the real case. Decoherence emerges from different mechanisms.

### Interaction with the environment

Interaction with the environment is the most natural way decoherence emerges from unitary evolution. This is purely fundamental quantum mechanics. A major error source in our quantum computer is that it interacts with the environment: they undergo a combined unitary evolution. However, we are not able to track the quantum state of the environment, so we just lose a portion of the information stored and processed in our quantum computer. The lost information leaks into the atoms and electrons of the experimental devices, the air, ourselves and eventually the whole universe. Then we humble human beings can only "trace out" the environment and face a "mixed state". This is effectively described by an incoherent error channel.

### Randomisation

Another source of decoherence is randomisation. Another error source in our quantum computer is the fluctuation of control signals. You may want to rotate a qubit around the Pauli $Z$ axis by $\pi/2$, but actually your laser or microwave lasts for a little longer or shorter, leading to a rotation by $\pi/2 + \theta$, where $\theta$ is a small random variable. However, as long as the distribution of $\theta$ is symmetric around 0 (an even function), the rotation is randomised to be a Pauli $Z$ error. One can see this by a simple calculation,

$$ 
\begin{aligned}
\int \mathrm{d}\theta\, f(\theta)\exp \left( -\frac{\mathrm{i}\theta Z}{2}  \right) \rho \exp \left( \frac{\mathrm{i}\theta Z}{2}  \right) &= \int \mathrm{d}\theta\, f(\theta) \left( \cos\frac{\theta}{2} - \mathrm{i}\sin \frac{\theta}{2} Z  \right) \rho \left( \cos \frac{\theta}{2} + \mathrm{i} \sin \frac{\theta}{2}Z  \right) \\
&= \Braket{\cos^{2} \frac{\theta}{2}}_{f} \rho + \Braket{\sin^{2} \frac{\theta}{2}}_{f} Z \rho Z.
\end{aligned}
$$

The cross term vanishes since $f(\theta)\sin (\theta/2) \cos (\theta/2)$ is an odd function.

We are actually taking an ensemble average in the above treatment, which describes what one observes by repeating the experiment many times. However, we usually run a quantum algorithm once with QEC, hoping that it can output the correct result. During this single-run computation, such errors happen from time to time. What is relevant to us is actually a time average. For our treatment to be valid, there is actually a hidden assumption that a time average can be described by an ensemble average. This is actually the central foundation of statistical mechanics. We believe this to be true if the correlation time of the fluctuation is much smaller than the timescale of our experiment.
### Projective measurements

Coherence can also be killed by projective measurements. Below we give two examples on how this happens.

Consider a single $Z$ rotation as in the previous section, but with a fixed yet unwanted angle. This can be a result of inaccuracy in the calibration of control signals. After that an $X$-check is measured ideally. Let the projector for the $X$ check be $\Pi_{\pm}$. Note that $Z \Pi_{\pm} = \Pi_{\mp} Z$ since $Z$ anticommutes with $X$. The quantum state with the measurement outcome becomes 

$$
\begin{aligned}
& \sum_{m=\pm}\left(\Pi_{m} \left( \cos\frac{\theta}{2} - \mathrm{i}\sin \frac{\theta}{2} Z  \right) \rho \left( \cos \frac{\theta}{2} + \mathrm{i} \sin \frac{\theta}{2}Z  \right) \Pi_{m}, m\right) \\ 
= {} & \sum_{m} \left(\cos^{2} \frac{\theta}{2} \Pi_{m}\rho\Pi_{m} + \sin^{2} \frac{\theta}{2} \Pi_{m} Z \rho Z \Pi_{m} + \left( \mathrm{i}\sin \frac{\theta}{2} \cos \frac{\theta}{2} Z \Pi_{-m} \rho \Pi_{m} + \mathrm{h.c.} \right),m\right). 
\end{aligned}
$$

If the initial state is an eigenstate of the $X$ check with either eigenvalue (in fact, it suffices if $\rho$ commutes with $\Pi_{+}$), then the cross term vanishes. This error, after measurement, reduces to a Pauli $Z$ error with probability $\sin^{2}\theta/2$. Note that here we only consider such errors on a single data qubit. We will discuss more complicated cases in the next section. 

Our second example concerns the coherent error during syndrome measurement. Consider the situation of measuring all $Z$ checks to prepare a surface code from a product state $\ket{+}^{\otimes n}$, by the following circuit. 

![circuit](/assets/img/260116/260116-meas.svg){: w="350" }
_Figure 1. Circuit for $Z$ syndrome measurement._

If $\theta = \pi/2$, then before measuring the ancilla qubit, the system is stabilized by $U X_{0} U^{\dagger} = Y_{0}Z_{1}Z_{2}Z_{3}Z_{4}$. Consider the error that $\theta = \pi/2 + \delta$ uniformly, that is, there is an overrotation by $\delta$ everywhere. It looks like a complicated coherent error $\exp(\mathrm{i}\delta Z_{0}Z_{1}\cdots Z_{4}/2)$ happening everywhere uniformly, but a similar calculation shows that after measuring $Y$ on the ancillas, it becomes an independent random readout error[^1] with probability $\sin^{2}\delta/2$: the system is projected to the $Z_{1}\cdots Z_{4}=m$ eigenspace, but the measurement outcome is $-m$, for $m = \pm 1$. Unlike the previous example, here this coherent error occurs uniformly across the system. We will also comment further on this point in the next section.

## When coherence survives

There are situations that inaccuracy in the calibration leads to coherent errors, and that these coherent errors cannot be killed by measurement. Most of the time, failure in killing the coherence by measurement emerges from degeneracy of quantum codes. 

In the previous section, we see that overrotation at one site can be safely turned to Pauli errors. However, overrotation at multiple sites can make a difference. A minimal example is a uniform overrotation on all data qubits, $U(\theta) = \prod_{i}\exp(\mathrm{i}\theta Z_{i}/2)$. Expanding this operator over the Pauli basis, we have

$$
U(\theta) = \cos^{n} \frac{\theta}{2} \left( I + \mathrm{i}\tan \frac{\theta}{2} \sum_{i} Z_{i} - \tan^{2} \frac{\theta}{2} \sum_{i,j}Z_{i}Z_{j} + \cdots \right).
$$

We see immediately that for small $\theta$, the weight for uncorrectable high weight errors is very low. This is what we have noticed when discussing the linearity of correctable errors. If the state before this error is in the code space, ideal syndrome measurements then kill all the cross terms in $U(\theta)\rho U^{\dagger}(\theta)$ except those whose left and right $Z$-strings yield the same syndrome, i.e., terms like $Z(l_{1}) \rho Z(l_{2})$ survive if and only of $Z(l_{1})Z(l_{2})$ is a stabilizer. (Here $l$ is an $n$-bit string and $Z(l)$ is the corresponding Pauli string which contains Pauli $Z_{i}$ if the $i$'th bit of $l$ is 1.) As a comparison, incoherent $Z$ errors only contain terms like $Z(l)\rho Z(l)$. The difference here is that due to the existence of $Z$-stabilizers, equivalently, due to the degeneracy of quantum codes, some cross terms survive the syndrome measurement. This will not change the fact that long errors are exponentially unlikely, but will affect the distributions and hence the performance of the code. For this case with surface codes, it was shown by [Bravyi et al.](https://arxiv.org/abs/1710.02270) in 2017 that there is still a threshold, but performance below threshold is worse. Later in 2022 this surface code threshold was further studied by statistical mechanics mapping ([Venn et al.](https://arxiv.org/pdf/2211.00655)), with $\sin^{2} \theta_{\text{th}}/2 \approx 0.18$. Note that this $\sin^{2} \theta_{\text{th}}/2$ value is the effective Pauli error rate for single-site coherent error. In this view, this threshold is even better than the $p_{\text{th}}=0.11$ value for incoherent errors.

Another consequence of the surviving coherent error is that after error correction, the logical error may remains coherent. For incoherent errors, the quantum state after correction is usually a mixing of a state without logical errors and states with logical errors. But for coherent errors, the post-correction state can be the original state evolved by a logical unitary. One should not draw a conclusion from here that incoherent errors can be cleanly corrected at some possibility and coherent errors can never be cleanly corrected. All we can get from a quantum computer is the final logical measurement outcomes. In this view, there is no difference whether the logical error is coherent or not. 

Nevertheless, there is still a difference between coherent and incoherent errors considering temporal accumulation. A single step unitary error of rotating by a small $\theta$ can lead to a readout error with probability $O(\theta^{2})$, which is equivalent to an incoherent error with probability $p=O(\theta^{2})$. For $n$ steps of such errors, the incoherent errors accumulate to $np$, while the coherent errors accumulate to a rotation of $n \theta$, equivalent to an incoherent error with probability $n^{2}p$. This resembles the difference between moving along one direction and a 1D random walk. Suppose the coherent error is random in the sign of $\theta$ in each step, then $n$ steps of such errors average to a rotation of $\sqrt{n}\theta$, equivalent to incoherent error rate $np$. This view also shows how randomisation effectively decoheres the error.

## Conclusion on practical purposes

For practical purposes, the conclusion of this blog is as follows. One generally does not need to worry about coherent errors. They are very unlikely to totally break fault-tolerance that holds for incoherent errors. However, coherence can make a difference sometimes, depending on the devices and codes used. It is therefore helpful to do some optimization to decoders or calibrations to achieve a better performance.

There are other interesting theoretical problems concerning coherent errors, such as quantum phases of topological codes under coherent errors. But this is beyond the scope of this post.


[^1]: There is a [published paper](https://arxiv.org/abs/2301.12859) claiming that this error during state preparation is harmful, and even breaks the existence of a threshold, indicating that surface code preparation is sensible to coherent errors. From our analysis here, it is clear that their error model is equivalent to independent readout errors, which can be handled by repeated measurements—a standard QEC technique. Their conclusion is likely incorrect and misleading. 
