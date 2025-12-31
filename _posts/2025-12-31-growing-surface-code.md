---
title: Growing a surface code
categories: [学术随笔 | Scholarly Notes]
tags: [量子计算 | Quantum Computation, 量子纠错 | Quantum Error Correction, 实践量子纠错探微 | Subtleties in Practical QEC]
math: true
image: /assets/img/251231/growth-cover.svg
---

The ability to grow is a useful feature of surface codes. In 2002, [Dennis, Kitaev et al.](http://arxiv.org/abs/quant-ph/0110143) described an approach for growing a surface code by gradually adding qubits to edges. In the famous 2018 paper [A Game of Surface Codes](https://arxiv.org/abs/1808.02892), which focused on code deformation techniques, Daniel Litinski discussed instantaneous surface code growth along one edge. In this blog, we describe a method to grow a surface code in a single syndrome measurement. This method has no fundamental difference from known ones; actually, it is a combination of Litinski's instantaneous expansion in two directions, yet packed into one single step. However, it seems that this method is not made explicit in the literature.

See the figure below. To grow a distance-3 code to a distance-7 one, we first initialize the qubits to be added into the $\ket{0}$ and $\ket{+}$ states according to their positions, and then measure all the stabilizers in the larger new code. Quite simple!

![codegrow](/assets/img/251231/growth.svg){: w="700" }
_Figure 1. Growing a distance 3 surface code to a distance 7 one._

This arrangement is intended to preserve the logical operators $\overline{Z}$ and $\overline{X}$, and hence the encoded state. To see this, note that for the distance-3 code, the left edge supports a logical $\overline{Z}$, and the top edge supports a logical $\overline{X}$. Now consider the $Z$-string supported by the new left edge. If the distance-3 code is initially stabilized by its logical $\overline{Z}$ operator, then the uncoupled system is stabilized by the left $Z$-string. This $Z$-string commutes with all stabilizers to be measured, so the system is still stabilized by the left $Z$-string, which now becomes the logical $\overline{Z}$ operator of the new code. This shows that the grown distance-7 code is still stabilized by its logical $\overline{Z}$ operator. By a similar argument can we show that if the initial distance-3 code is stabilized by its logical $\overline{X}$ operator, then the grown distance-7 code is still stabilized by its (now longer) logical $\overline{X}$ operator. This shows that this growing process preserves the logical space.[^1] 

Fault-tolerance of this growing procedure is straightforward to see because error spreading during this procedure is the same as that during a standard syndrome measurement round of the larger code. There is one more subtlety here. As in the case of initializing a new surface code, if you want to fix the values of the checks, you will need more rounds of syndrome measurements. But you will not always need to do so. It is another really subtle problem when we need to fix values of the checks, and what cost we shall pay for not doing so.

[^1]: This follows from a lemma saying that if a CPTP map preserves $X$ and $Z$, it preserves all states. This lemma is not entirely trivial, as linearity is not enough to prove this.