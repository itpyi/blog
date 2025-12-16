---
title: Is it necessary to perform virtual Pauli gates before applying T gate?
categories: [学术随笔 | Scholarly Notes]
tags: [量子计算 | Quantum Computation, 量子纠错 | Quantum Error Correction]
math: true
# description: 
---

## The problem

It is widely said that in a stabilizer-code-based error-corrected circuit, all Pauli corrections can be traced within software until a logical non-Clifford gate is to be applied. A common misunderstanding as I have observed is that all Pauli corrections, as well as logical Pauli gates, should be applied before a logical non-Clifford gate is to be applied. In this blog I will point out why this is not true, and what is the more precise statement.

This problem is closely related to the Gottesman-Knill theorem, stating that a quantum circuit with only Clifford gates, Pauli measurements and stabilizer state inputs can be efficiently simulated by a classical computer. This theorem is built upon the same reason as the validity of tracing all Pauli corrections and logical Pauli gates in a software through a Clifford circuit--that Clifford gates map Pauli to Pauli by conjugation. But one may ask why it is necessary condition for the theorem that the input should be stabilizer states. A most confusing yet fruitful example is the gate teleportation realization of a $T$ gate. 

## An example: gate teleportation

The gate teleportation gadget for a $T$ gate is shown in the figure below. Here we do not care about whether the circuit is protected by any code.

![circuit_gt](/assets/img/251216/gate_tele.svg)
_Figure 1. Circuit for teleportation of a $T$ gate_

The circuit contains only Clifford gates and Pauli measurements, but effectively implements a $T$ gate. However, here is some subtlety on the Clifford correction conditioned on the measurement. Since projector along $Z$ commutes with a controlled gate, the circuit above is equivalent to the following, which contains a controlled-$S$ gate (non-Clifford!) explicitly.

![circuit_gt_ct](/assets/img/251216/gate_tele_ct.svg)
_Figure 2. Circuit for teleportation of a $T$ gate, with controlled-$S$ gate explicitly_

Actually, it contains two sources of non-Cliffordness: the input and the controlled-$S$ gate. Nevertheless, the "amount" of non-Cliffordness is still one $T$ gate, since its net effect is a $T$ gate. We can perform the following transformation on the circuit to see this directly (note that $TXT^{\dagger} = SX$, and three CNOT gates composes to a SWAP).

![circuit_eq](/assets/img/251216/gate_tele_eq.svg)
_Figure 3. Transformation of the teleportation circuit_

To make the problem simpler, let's neglect the controlled- gate first as the figure below.

![circuit_nofb](/assets/img/251216/gate_tele_nofd.svg)
_Figure 4. Circuit without feedback_

In the following, we will analyze how the non-stabilizerness makes classical simulation difficult.

First, a direct application of $T$ conjugates $Z$ to itself, and $X$ to $SX=(X+Y)/\sqrt{2}$. After this, you should trace two terms for one $X$ before $T$. This implies that the complexity of simulating a Clifford+T circuit grows exponentially with the number of $T$ gates.

Second, for the simplest case (no feedback or controlled gate), a trace-back stabilizer analysis shows $ZI \leftarrow ZI, XI \leftarrow XX$ and $ZZ \leftarrow IZ, XZ \leftarrow YY$. This shows that, on the first line, $Z$ is mapped to $Z$ since $ZI$ is invariant, and $X$ is mapped to $(X+(-1)^{m}Y)/\sqrt{2}$ since $X_{1}(X_{2}+Y_{2})/\sqrt{2}$ is mapped to $(XI+YZ)/\sqrt{2}$. This shows an effect of $T^{\pm 1}$: a single term stabilizer $X$ splits into two terms.

Finally, for the feedback case (which is simpler than the controlled-$S$ case since no more non-Clifford gate is needed), the $S^m$ correction cancels the dependence of the measurement outcome, recovering precisely the effect of a $T$ gate. To see this, note that $S X S^{\dagger}=Y, SYS^{\dagger}=-X$.

## Virtual Pauli gates through gate teleportation gadget

The analysis above seems to indicate that you cannot pass an $X$ through the $T$-gadget without increasing the complexity of classical simulation. After all, if you can simulate the circuit classically, you will not bother to build a quantum computer. However, the subtlety comes from the fact that you are not simulating the whole circuit classically, but you are actually running a quantum circuit and just leaving some trivial part to a classical computer. So we must do something on the quantum computer here. But what we do needs not be executing all the logical Pauli gates before this gadget. We only need to read out the measurement result correctly.

Before delving into the discussion, I would like to emphasize that here we are analyzing the evolution of virtual gates rather than stabilizers. In the stabilizer case, a $Z_{2}$ measurement destroys an $X_{1}X_{2}$ stabilizer. But such measurements has no effect on the virtual gate acting on another qubit. That is, if you have virtual $X_{1}X_{2}$ before measuring $Z_2$, you will still have a virtual $X_1$ after the measurement, no matter the measurement outcome.

Let's consider the original gadget with measurement and feedback. A virtual Pauli $Z_{1}$ is always safe from any viewpoint. Let's consider what we should do with a virtual Pauli $X_{1}$. It will propagate through the CNOT gate to $X_{1}X_{2}$, inverting the measurement result. So in our classical control system, if there is an unapplied virtual Pauli $X_{1}$, we should interpret the measurement outcome as its inverse. This will affect the feedback operator we perform. Since the feedback gate is Clifford, we can furthermore pass $X_{1}$ though this gate. 

There is another equivalent viewpoint. See the table below. If we do not adjust the measurement outcome and use the original feedback, the circuit will have the same effect as a $T$ gate, with the virtual gate left being $SX$ (first two rows). However, if we reinterpret the outcome and adjust the feedback, we will have effectively apply a gate after the $T$ gate (the rule is: $U_{\text{adj-fb}} = U_{\text{eff}}U_{\text{ori-fb}}$). This effective gate will be $S^{\pm 1}$, which brings the left virtual gate back to Pauli gates. This gives the same result as the Clifford analysis on virtual gates. 

| bare outcome | original feedback | adjusted feedback | effective gate | virtual gate left |
| ------------ | ----------------- | ----------------- | -------------- | ----------------- |
| 0            | $I$               | $I$               | $I$            | $SX$              |
| 1            | $S$               | $S$               | $I$            | $SX$              |
| 0            | $I$               | $S$               | $S$            | $SXS^{\dagger}=Y$ |
| 1            | $S$               | $I$               | $S^{\dagger}$  | $SXS = X$         |

One may then be confused by the observation that the Clifford analysis, unlike the effective gate analysis, seems to be independent on the input state. If the input state is not the magic state but say $\ket{0}$, the gadget measures $Z_{1}$ and applies $S_{1}$ conditioned on the measurement outcome. The analysis still works in this case. So it seems that the classical computer feels no effect from the magic state. This is not a bug. This is exactly what we want. The classical computer *should* feel no effect of non-stabilizerness, otherwise the complexity will blow up. More directly speaking to the problem, the non-stabilizerness is injected into the state by our real operations on the quantum circuit, including the adjusted feedback, which needs not be captured by our virtual gate tracing system. The classical computer only traces a part of the circuit, namely the virtual Pauli gates, which is far from the full knowledge of the evolved state in the circuit. So it is reasonable to transfer anything non-Pauli to the real quantum circuit.

We can take a closer view on how the non-Pauli virtual gates are transferred into the real quantum circuit. From the first viewpoint of Clifford analysis, the key point is that we are tracing one measurement outcome each time, based on the output from the real quantum circuit. We are tracing a collapsed state after measurement rather than a measurement-and-feedback channel. If we do the latter, we will have to trace two branches of the virtual Pauli gates, which is equivalent to tracing the two branches of an $X$ passing directly through a $T$ gate. But the measurement outcome is known in each run of the circuit, so tracing one branch is enough.

From the second viewpoint of the effective gates given by the adjusted feedback, the mechanism is that we transfer a non-Pauli factor in $SX$ to the quantum circuit when adjusting the feedback. It will be more transparent if we consider a direct application of a  gate. If there is a virtual $X$ before this $T$ gate, you can do any one of the following things which are trivially equivalent:
- Apply the $X$ before $T$ and the left virtual gate is trivial;
- Apply an $SX$ after $T$ and the left virtual gate is trivial;
- Apply an $S$ after $T$ and the left virtual gate is $X$.

The third seems silly, but it aligns with what we do in the gate-teleportation gadget (with a slight difference that in the gadget sometimes the effective gate is $S^\dagger$). In the gadget, it becomes not that silly, because compared to the other two, it requires no more quantum operations. 

In short, the effectiveness of this method of reinterpreting measurement outcome and adjusting feedback can be explained from two viewpoints. One is the additive viewpoint of Clifford analysis, from which measurement and feedback allows us to trace only one branch of $TXT^{\dagger} = (X+Y)/\sqrt{2}$. The other is the multiplicative viewpoint of effective gates, from which an effective gate from the adjusted feedback brings $TXT^{\dagger} = SX$ back to Pauli.

## Error correction through gate teleportation gadget

Finally let's consider error correction. During an error correction procedure, we usually trace Pauli corrections in a classical computer. These are physical-level virtual Pauli gates. Therefore, for the same reason as the Clifford analysis for logical-level virtual Pauli gates, what we need is to interpret the logical $Z$ measurement result correctly. This requires that the decoding algorithm should output the predicted logical measurement outcome at the time of the $T$ gadget, but there is no need to actually correct the errors by applying these Pauli gates. 

One may also worry about the case where a logical $T$ is implemented transversally, rather than by gate teleportation. In such a case there is still no need to realize the virtual Pauli corrections. One may worry that the virtual corrections will be no longer Pauli after a transversal implementation of a logical $T$ gate. This is true, but the errors they correct will be projected to Pauli errors again after a new round of syndrome measurement. The syndrome data before and after the logical gate will be viewed as a whole during decoding. 

There are infinitely many ways to implement a logical non-Clifford gate. But the guiding principle of stabilizer code error correction is that logical operations are transparent to the error-correcting subroutine, while only errors that drive the system out of the code space matter. The requirement of decoding in time by the gate teleportation gadget is merely because the measurement outcome is needed for a non-Pauli feedback, which may be expensive to postpone.