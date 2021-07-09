---
layout: post
title: Topological Teleportation
tags: [Quantum, Fermions, Anyons, Tutorial, QuTip, Python, Topological Quantum Computing]
description: A tutorial on how to teleport the topological qubit from one Kitaev chain onto another by applying nothing else but braiding operation between the domain edges.
image: /assets/figures/png/topological-teleportation/thumbnail.png
lang: en_US
lang-ref: topological-teleportation
katex: true
---

Quantum teleportation {% cite PhysRevLett.70.1895 --file references %} has [already been covered](/2020/03/22/simulating-quantum-teleportation/) on this blog. This was the first tutorial I have written on this website. Right now we are going to re-visit this concept again from a slightly different perspective. Recently we have been talking a lot about the [Majorana zero modes](/2020/04/09/mzms/), creating them using the [topological phase transitions](/2021/05/09/braiding/) and performing quantum computation by braiding them defining the [Majorana qubits](/2021/06/09/majorana-qubits/). The question to ask now is, would it be possible to perform the quantum teleportation on such states defined on the Kitaev chains? In other words, could we teleport a state from one Kitaev chain onto another using nothing else but braids of their edges? Turns out yes!

It was June 5th, 2019. Very fun time as [Jonathan Dowling](https://en.wikipedia.org/wiki/Jonathan_Dowling) visited us in Shanghai to be a speaker at [ITU Workshop on Quantum Information Technology (QIT) for Networks](https://www.itu.int/en/ITU-T/Workshops-and-Seminars/2019060507/Pages/default.aspx) conference which we all attended. During the break time I was chatting with my PhD advisor [Tim Byrnes](https://nyu.timbyrnes.net/), we discussed teleporting states in the Kitaev chains and we got quite excited with this idea. It took us quite few months to develop a necessary theory to make it work, which eventually resulted the PRL publication {% cite PhysRevLett.126.090502 --file references %} which additionally has been highlighted [on the NYU Shanghai website](https://shanghai.nyu.edu/news/nyu-shanghai-scientists-develop-method-teleporting-quantum-states-using-majorana-fermions).

The key to perform the quantum teleportation is the ability to perform the entangling operations. As we have few entangling gates described in {% cite narozniak2020majorana --file references %} and also in [this tutorial](/2021/06/09/majorana-qubits/) we found that the inner braid equivalent to $$ \sqrt{X_1 X_2} $$ operation does produce the effect of quantum teleportation with slightly different classical correction.

We start by producing the entangled state. Working in the logical space we begin by creating entanglement between second and third logical qubits using inner braid which corresponds to $$ \sqrt{X_2 X_3} $$ operation

$$
\begin{aligned}
     \left \vert E \right>_{23} & = \sqrt{X_2 X_3} \left \vert 0 0_L \right>_{23} \\
     & =
     \frac{1}{\sqrt{2}} ( \left \vert 0 0_L \right>_{23} + i \left \vert 1 1_L \right>_{23}) .
\end{aligned}
$$

The first logical qubit contains the state to be teleported $$ \left \vert \psi \right> $$, which can be written as

$$
\begin{aligned}
    \left \vert \psi \right>_1  =  \alpha \left \vert 0_L \right>_1 + \beta \left \vert 1_L \right>_1 .
\end{aligned}
$$

Now after applying the $$ \sqrt{X_1 X_2} $$ gate we have

$$
\begin{aligned}
& \sqrt{X_1 X_2}  \left \vert \psi \right>_1   \left \vert E \right>_{23} \\
=  & \frac{1}{2}(\alpha \left \vert 000_L \right> + i \alpha \left \vert 011_L \right> + \beta \left \vert 100_L \right> + i \beta \left \vert 111_L \right> \\
& +
      i\alpha \left \vert 110_L \right> - \alpha \left \vert 101_L \right> + i \beta \left \vert 010_L \right> - \beta \left \vert 001_L \right> ) \\
= &\frac{1}{2}(\left \vert 00_L \right> (\alpha \left \vert 0_L \right> - \beta \left \vert 1_L \right>) + i \left \vert 01_L \right> (\beta \left \vert 0_L \right> + \alpha \left \vert 1_L \right>) \\
 & -\left \vert 00_L \right>(\beta \left \vert 0_L \right> - \alpha \left \vert 1_L \right>) + i \left \vert 01_L \right> (\alpha \left \vert 0_L \right> + \beta \left \vert 1_L \right>))
\end{aligned}
$$

from which we can deduce what corrections have to be applied depending on the measurements of the first two logical qubits. Under the form of logical circuit those $$ X $$ and $$ Z $$ corrections depending on measurements are

{% include figure.html max-width="70%" fll="/assets/figures/svg/topological-teleportation/topological-teleportation.svg" alt="" %}

The rightmost logical qubit is prepared in the logical $$ \left \vert +_L \right> $$ state to avoid getting entangled with the rest of the system in case of the $$ X $$-correction getting applied. The teleportation itself is performed entirely using the braiding operations, however classical correction as requires the logical $$ X $$-operation needs an extra qubit.

I did this tutorial slightly different than we described in the paper. Here we do not simply assume the existence of such ancilla Kitaev chain, we also prepare it. If we include the state preparation an extra ancilla topological qubit is required and one more inner braid

{% include figure.html max-width="70%" fll="/assets/figures/svg/topological-teleportation/blog_topological_teleportation_braiding_diagram.svg" alt="" %}

The above diagram contains classical correction operators which of course do not need to always be applied. To more formally define the measurement we will prepare appropriate projection operators. But first, a topological qubit in logical $$ Z $$-basis could be written as

$$
\begin{aligned}
\left \vert m_L \right> &= (1-m) \left \vert 0_L \right> + m \left \vert 1_L \right>
\end{aligned}
$$

where $$ m \in \{0, 1\} $$. For the final state after the teleportation $$ \left \vert \psi_f \right> $$ if the first topological qubit has been measured to be $$ m^{(1)} $$ and second $$ m^{(2)} $$ we could define the projection operator

$$
\begin{aligned}
\Pi_{m^{(1)}, m^{(2)}} &= \left \vert m^{(1)}_L \right> \left< m^{(1)}_L \right \vert \otimes \left \vert m^{(2)}_L \right> \left< m^{(2)}_L \right \vert \otimes I \otimes I \otimes I
\end{aligned}
$$

which gives us the post-teleportation and post-measurement state

$$
\begin{aligned}
 \left \vert \psi_{m^{(1)}, m^{(2)}} \right> &= \Pi_{m^{(1)}, m^{(2)}} \left \vert \psi_f \right>
\end{aligned}
$$

we could trace out logical qubits apart from the teleported logical qubit on the third side and get fidelity for each of the outcomes as

$$
\begin{aligned}
 f_{m^{(1)}, m^{(2)}} &= \left < \psi \right \vert \text{Tr}_{1, 2, 3, 4, 7, 8, 9, 10}(\left \vert \psi_{m^{(1)}, m^{(2)}} \right> \left < \psi_{m^{(1)}, m^{(2)}} \right \vert) \left \vert \psi \right> .
\end{aligned}
$$

This was for the case of $$ 5 $$ topological qubits each of length $$ L = 2 $$. The partial trace arguments would need to be adjusted for different system size. I have numerically simulated the above approach and feel free to review the [Python source code](https://github.com/marekyggdrasil/majorana/blob/40e12fa0f9ae7f22eebd11ead92ed15e7d97aefc/teleport.py). The bar plot comparing fidelities for the cases with and without classical correction are as follows

{% include figure.html
max-width="70%" fll="/assets/figures/png/topological-teleportation/outcomes.png" alt="Effect of classical correction on fidelities per outcome"
caption="Bar plot showing how unreliable teleportation is without the classical correction." %}

The exact distribution depends on the random state. There would always be one outcome with perfect fidelity as there is one outcome that does not require any classical correction.

Today we have simulated the topological teleportation by applying sequences of unitary braids on topological states. The full source code of this simulation is published under MIT licence [on GitHub](https://github.com/marekyggdrasil/majorana). If you find errors please tweet me and let me know.

{% bibliography --file references --file wiki --cited %}
