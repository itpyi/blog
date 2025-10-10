---
title: 如何评价微软发布的拓扑量子芯片？
categories: [科普]
tags: [量子计算 | Quantum Computation]
math: true
description:  
---

本文首发于[全球首款拓扑架构量子芯片问世，这一芯片有哪些技术突破？会带来哪些实际变化？ - Schroenberg 的回答 - 知乎](https://www.zhihu.com/question/12784048864/answer/105968265677)

## 太长不看版

微软推出了一个大新闻 [Introducing Microsoft Majorana 1](https://news.microsoft.com/azure-quantum/)，一篇 Nature [实验论文](https://www.nature.com/articles/s41586-024-08445-2) 和一个[路线图](https://arxiv.org/abs/2502.12252)。

大新闻只体现肺活量，不用太在意，重要的是路线图和论文。简而言之，这份路线图即使实现了最后一步也不会有任何实际应用价值，而实验论文甚至还没有完全实现路线图的第一步。拓扑量子计算近年来命运多舛，但有人投钱继续探索也不是坏事。

## 声明

不是拓扑量子计算领域专家，也没有仔细阅读实验论文和路线图的全文，如有错误，欢迎方家批评指正。

## 潜在的意义和积极影响

目前的量子计算还处在不同技术路线各有优劣的摸索阶段。主流的有基于中性原子的、基于离子阱的和基于超导的，微软力推的基于拓扑物态的量子计算是一条和前三者迥异的路线，其区别要之有二：

- 如果都能实现且优化得很好，前三者仍然过于容易受到环境的影响而出错，而拓扑量子计算天然地不易受到环境影响而出错。
- 前三者在物理上都已经毋庸置疑地实现了，尽管还很稚嫩，而后者依赖的物理材料是否存在仍存争议，即使微软这篇实验论文也没有给出无可否认的证据。

区别 1 可以用经典通信来类比，前三者的脆弱程度相当于远程用电磁波通信，即俩人发微信。由于电磁波在大气中容易受到各种干扰，所以要可靠通信就必须主动采用纠错码来保护，也就需要付出更多成本。拓扑量子计算的稳定程度相当于写好信再亲自带给对方，其物理介质非常稳定，虽然也存在信被烧毁或者偷走的可能。

因此，要实现大规模计算，前三者必须采用纠错码主动纠错，而后者自身硬件就稳定很多，当然也可以采用纠错码进一步提升其稳定性。这是拓扑量子计算吸引力之所在。

当然，拓扑量子计算依赖于拓扑物态，需要在真实材料中找到马约拉那零能模（MZM）或者其他任意子，这是凝聚态实验中一个尚未被普遍认可其已经解决的问题（微软这篇实验论文也不能确证 MZM 的存在，详后）。在学界普遍承认该问题已经解决之前，任何有关拓扑量子计算的宣称都不能尽信。

容错和拓扑量子计算的奠基人之一 Alexei Kitaev 曾表示，当下人们造量子计算机的努力就像 Charles Babbage 时代（19 世纪）的人试图用机械结构造出现在的经典计算机，量子计算机也许需要一些全新的技术（[Alexei Kitaev, Theoretical Physicist](https://heritageproject.caltech.edu/interviews-updates/alexei-kitaev)）。 在他眼里，拓扑物态正是一种可能。所以沿着这一路线的探索无疑是有益的。

## 微软实验论文和路线图与公众预期的距离

微软的路线图提出了四步：

> Our roadmap encompasses four generations of devices: a single-qubit device that enables a measurement-based qubit benchmarking protocol; a two-qubit device that uses measurement-based braiding to perform single-qubit Clifford operations; an eight-qubit device that can be used to show an improvement of a two-qubit operation when performed on logical qubits rather than directly on physical qubits; and a topological qubit array supporting lattice surgery demonstrations on two logical qubits.

- 一个单量子比特的仪器，可以做基于测量的量子比特标定。即做一个拓扑量子比特，并且做一些反对易的量子测量，根据测量结果的统计学性质推定是否真的做出了一个量子比特。
- 一个双量子比特的仪器，基于测量做单比特 Clifford 门。
- 一个 8 量子比特的仪器，在其基础上做两个逻辑比特的量子纠错。
- 一个拓扑量子阵列，在其上做量子纠错，尤其是多逻辑比特测量。

（当前，原子、离子和超导平台大约在第 3 步左右，由此可见拓扑量子计算仍然任重道远。）

这一路线图不知何故，并未提及非 Clifford 门如何实现，而仅能实现 Clifford 门和 Pauli 测量的量子计算机，其能力不会超越经典计算机。这是我最感困惑的地方。

微软的实验论文大概对应路线图的第一步，他们希望做出一个拓扑量子比特，采用仅基于测量进行操作的架构，而文章实现了这个架构中的一项关键技术，并很可能实现了 MZM。为什么说“很可能”？论文作者倒是实在，他们指出

> These measurements do not, by themselves, determine whether the low-energy states detected by interferometry are topological. However, our data tightly constrain the allowable energy splittings in models of trivial Andreev states.
> 
> 仅凭这些测量，并不能推定干涉仪探测到的低能态是拓扑的。另一种可能是平凡相的 Andreev 态，而我们的数据对这种可能的模型所能预言的能级劈裂给出了很强的限制，
即几乎但并不完全能排除对实验数据的另一种解释。

至于第一步中重要的“标定”，文章似乎并未提及。