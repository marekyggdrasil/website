---
layout: post
title: Geometric Phase for a Qubit
tags: [Quantum, Tutorial, QuTip, Python]
description: Numerical simulation of a Berry phase of a qubit.
lang: en_US
lang-ref: qubit-berry-phase
image: /assets/figures/png/qubit-berry-phase/res_qubit.png
katex: true
gist: https://gist.github.com/marekyggdrasil/66a5b0d822eb60b27b68144045613bcd
---

In Physics, the term *phase* is extremely abused. There could be different *phases of matter* such as gas phase, liquid phase, solid phase. Changes between them are *phase transitions*. There also is an idea of a *phase space* which describes a state of a dynamical system (coordinates are positions and momentum of bodies). The meaning of this term is not very consistent as more generally, a *phase* represents a state of some periodic function, in Physics that would be a phase of a wave for instance. This leads us to the Mathematical meaning of this concept of a *phase* which is a complex number of norm $$1$$ taking a form $$e^{i\phi}$$. In the set of $$\mathbb{C}$$ phase turns out to be a constant, thus when a physicist tries to sound fancy (phancy!) would say "up to a phase" and what he means is "up to a constant" in $$\mathbb{C}$$.

The geometric phase is (yet another) concept of a phase which results from adiabatic changes in the system which return the system into its initial state and pull out this global phase factor known as geometric phase, Pancharatnam–Berry phase, Pancharatnam phase, or Berry phase. The reason why it has so many names is it was independently discovered by Kato {% cite Kato1950 --file references %} (yet I never heard anyone calling it *Kato phase*), Pancharatnam {% cite Pancharatnam1956 --file references %}, Longuet-Higgins {% cite LH1958 --file references %} and (the obvious favorite) Sir Michael Berry {% cite Berry1984 --file references %}.

In this short tutorial we are going to derive the Berry phase of a qubit after adiabatically traversing a cyclic path on the surface of the Bloch sphere. We are going to derive the expected Berry phase, numerically simulate it and compare the two. This is going to be an adiabatic evolution in a same sense as we did it in [Quantum Adiabatic Optimization](/2020/05/22/quantum-adiabatic-optimization/) tutorial.

Let us begin by theoretically deriving the value of such phase. First let us consider a quantum state of a qubit in the Bloch sphere coordinates

{% include figure.html url="#" fll="/assets/figures/svg/qubit-berry-phase/Bloch_sphere.svg" alt="" %}

assuming the angle $$\phi=0$$ is fixed such state would take then the following form

$$
\begin{aligned}
\left \vert \theta, \phi \right> &= \cos(\theta/2) \left \vert 0 \right> + e^{i\phi} \sin(\theta/2) \left \vert 1 \right >.
\end{aligned}
$$

It is straightforward to [define this state using QuTip](https://gist.github.com/marekyggdrasil/66a5b0d822eb60b27b68144045613bcd#file-run-py-L24) (without $$\phi$$-angle dependence)

```python
psi0 = np.cos(theta/2.)*basis(2, 0)+np.sin(theta/2.)*basis(2, 1)
```

Let that be our initial state for the adiabatic time evolution. [We already know](/2020/05/22/quantum-adiabatic-optimization/) implementing the adiabatic evolution requires us to provide the Hamiltonian operator at an arbitrary step of the time evolution in order to let the ground state follow it. The Bloch sphere Hamiltonian for the qubit described in the $$Z$$-basis would take the form

$$
\begin{aligned}
H(\theta,\phi) &= \cos(-\phi)\sin(\theta)\sigma^x + \sin(-\phi)\sin(\theta)\sigma^y + \cos(\theta)\sigma^z
\end{aligned}
$$

which again is not difficult to implement in QuTip, for instance for varying $$\theta$$ we can implement the time evolution [like this](https://https://gist.github.com/marekyggdrasil/66a5b0d822eb60b27b68144045613bcd#file-qm-py-L6-L16)

```python
def sweepPhaseDown(psi0, theta_min, theta_max, phi, tau):
    def angle(t):
        return (1-(t/tau))*theta_max + (t/tau)*theta_min
    res = 300
    times = np.linspace(0., tau, res)
    opts = Options(store_final_state=True)
    H = [
        [sigmax(), (lambda t, args: np.cos(-phi)*np.sin(angle(t)))],
        [sigmay(), (lambda t, args: np.sin(-phi)*np.sin(angle(t)))],
        [sigmaz(), (lambda t, args: np.cos(angle(t)))]]
    return mesolve(H, psi0, times, options=opts)
```

you may prefer to use `lambda` functions if your linter allows it! Note that we keep $$-\phi$$ angle in the Hamiltonian which could be simplified using even/odd trigonometric identities, yet I think it is better to keep it this way to indicate the direction of $$\phi$$-angle on the Bloch sphere.

So we have the initial state and the Hamiltonian. We know how to evolve it. Now we need something to compare it to, so let us derive the Berry phase. This is done by taking the derivative of $$ \left \vert \theta, \phi \right> $$ state with respect to the angles $$ \theta $$ and $$ \phi $$

$$
\begin{aligned}
\left < \theta, \phi \right \vert \frac{d}{d\theta} \left \vert \theta, \phi \right> &= 0 \\
\left < \theta, \phi \right \vert \frac{d}{d\phi} \left \vert \theta, \phi \right> &= i \sin(\theta / 2)^2.
\end{aligned}
$$

The Berry phase factor

$$
\begin{aligned}
\gamma &= i \int_{\theta_i}^{\theta_f} \left < \theta, \phi \right \vert \frac{d}{d\theta} \left \vert \theta, \phi \right> \,d\theta + i \int_{\phi_i}^{\phi_f} \left < \theta, \phi \right \vert \frac{d}{d\phi} \left \vert \theta, \phi \right> \,d\phi \\
&= -(\phi_f - \phi_i) \sin(\theta / 2)^2
\end{aligned}
$$

and after the adiabatic time evolution is over the final state will take the form

$$
\begin{aligned}
\left \vert \psi_f \right> &= e^{i \Gamma} e^{i \gamma} \left \vert \psi_0 \right >
\end{aligned}
$$

where $$ e^{i \gamma} $$ is the Berry phase factor and $$ e^{i \Gamma} $$ is the dynamic phase factor resulting from the adiabatic time-evolution. We will permit ourselves to ignore $$ \Gamma $$ by keeping the overall time-evolution length $$ \tau $$ an even multiple of $$ \pi $$ as defined [here](https://gist.github.com/marekyggdrasil/66a5b0d822eb60b27b68144045613bcd#file-run-py-L11).

Now let us consider few cases to be numerically tested. For the first plot, we are going to perform a series of numerical time-evolutions initiated in a state $$ \left \vert \theta, \phi \right> = \left \vert \theta, 0 \right> $$. Each of those time evolutions is going to have different initial value of the angle $$ \theta $$. All of those time-evolutions will perform a full revolution around the Bloch sphere by adiabatically varying the angle $$ \phi $$ from $$ 0 $$ to $$ 2 \pi $$. The plot below shows the numerically measured value of gemetric phase (dashed line) as well as theoretically predicted value $$ e^{i \gamma} $$ (solid line). The imaginary part is represented by orange and real part by blue.

{% include figure.html url="#"
min-width="60%" fll="/assets/figures/png/qubit-berry-phase/res_qubit.png" alt="" %}

The test consists of another series of numerical time-evolutions initiated in $$ \left \vert \theta, \phi \right> = \left \vert 0, 0 \right> $$. This time the protocol is slightly different. The $$ \theta $$ angle is increased until it reaches $$ \theta_f = 3 \pi / 4 $$. Then we vary the angle $$ \phi $$ from $$ 0 $$ to $$ \phi_f $$ and it is the $$ \phi_f $$-angle that distinguishes the numerical runs. Each of the runs is finalized by decreasing $$ \theta $$ back to $$ 0 $$ which returns the state back to the North Pole of the Bloch sphere.

{% include figure.html url="#"
min-width="60%" fll="/assets/figures/png/qubit-berry-phase/res_qubit_2_1.png" alt="" %}

The final test consists of three steps just as the previous one, only this time the $$ \phi_f = 3 \pi \ 4 $$ is fixes and we vary the $$ \theta_f $$ for each of the adiabatic runs.

{% include figure.html url="#"
min-width="60%" fll="/assets/figures/png/qubit-berry-phase/res_qubit_2_2.png" alt="" %}

It is visible that that the dash line does not follow solid line closely for the imaginary part for the time-evolution that takes $$ \theta_f $$-angle all the way down to the South Pole of the Bloch sphere. I am not sure how to formally prove it, yet my intuition suggests that is the most error-sensitive case as variation of $$ \phi $$ angle while on the South Pole is not reflected by any change of position on the sphere. Moreover, keep in mind state is valid as long as its norm is $$ 1 $$ thus it is the squared value of each of those components that has to add up to $$ 1 $$. They do seem off by a lot, yet when we square them one gets very close to $$ 0 $$ and another very close to $$ 1 $$, together they approximate to $$ 1 $$.

That would be it. I hope you feel more comfortable with this *phancy* jargon of "phase". As usual the code is available in [this public gist repository](https://gist.github.com/marekyggdrasil/66a5b0d822eb60b27b68144045613bcd) and if you have suggestions how to include the dynamical phase in this simulation please feel free to Tweet me and contribute!

{% bibliography --file references --file wiki --cited %}
