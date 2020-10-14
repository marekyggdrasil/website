---
layout: post
title: Jordan-Wigner Transformation
tags: [Quantum, Fermions, Bosons, Anyons, Tutorial, QuTip, Python]
description: Coding and testing fermion and boson operators with QuTip in Python. Implementing Jordan-Wigner transformation.
lang: en_US
lang-ref: jordan-wigner-transformation
image:
katex: true
gist: https://gist.github.com/marekyggdrasil/ed36ee96444e445da1dcd59e40802c1d
---

In this article we are going to simulate different kinds of indistinguishable particles and introduce that concept briefly in case if you are not yet familiar with them. We will also write few words about the *second quantization* formalism in $$1$$D to be able to formally represent quantum states involving those particles. For fermions we will use Jordan-Wigner transformation, for bosons we will use native operators already available in QuTip. In the end you should be able define operators that act on fermions or bosons in QuTip, we will test their properties using `pytest` testing framework and eventually as we conclude, we will briefly discuss the idea of test driven science.

For a starter, brief introduction to indistinguishable particles. Let $$\left\vert \psi_\alpha \psi_\beta \right>$$ be a quantum state representing a system holding two indistinguishable particles $$\alpha$$ and $$\beta$$. Their indistinguishability implies that

$$
\begin{aligned}
\left \vert \left< \psi_\beta \psi_\alpha \vert \psi_\alpha \psi_\beta \right> \right \vert^2 &= 1.
\end{aligned}
$$

Note that particles $$\alpha$$ and $$\beta$$ are indistinguishable from measurement perspective and what vanishes during quantum measurement is the phase factor. In such case exchanging particles should allow a phase factor without violating their indistinguishability. To express that let $$U$$ be an operator that exchanges two particles

$$
\begin{aligned}
U \left \vert \psi_\alpha \psi_\beta \right> &= e^{i\phi} \left \vert \psi_\beta \psi_\alpha \right>.
\end{aligned}
$$

The value of $$\phi$$ characterizes groups of indistinguishable particles. For *bosons* phase factor is $$\phi = 0$$ and for *fermions* it is $$\phi = \pi$$. Particles that have any different value of $$\phi$$ are commonly referred to as *anyons*. Let us change the notation a bit to something more workable computationally, for this let us use the second quantization formulation which allows us to describe quantum many-body states from perspective of occupation number. For this we consider the Fock state basis, consisting of single particle states occupancies. For example, a Fock state for a quantum $$4$$-body system in a vacuum would be $$\left \vert 0000 \right>$$ and if last site is occupied by a single particle we would write it as $$\left \vert 0001 \right>$$ and so on.

Such Fock states are manipulated using creation and annihilation operators. For fermions we write them as $$a^\dagger_j$$ and $$a_j$$. For bosons $$b^\dagger_j$$ and $$b_j$$. The subscript $$j$$ represents the lattice site index. For example we could place fermions on the edges using $$a^\dagger_1 a^\dagger_4\left \vert 0000 \right> = \left \vert 1001 \right>$$. Values in the ket represent occupancy number. Note that this notation does not really distinguish *what* occupies the site so let us just assume those were *fermionic sites* and from now on let us try to just define if site is meant for bosons or fermions. We should try to avoid mixing them up. The properties of indistinguishable particles (i.e. what phase factor their exchange pulls out) is entirely described by their operators, thus for fermions we have

$$
\begin{aligned}
\{a_n, a_{n^\prime}\} &= 1 \\
\{a_n, a_{n^\prime}^\dagger\} &= \delta_{nn^\prime} \\
a_n^2 &= 0
\end{aligned}
$$

and for bosons

$$
\begin{aligned}
[b_n, b_{n^\prime}] &= 0 \\
[b_n, b_{n^\prime}^\dagger] &= \delta_{nn^\prime}.
\end{aligned}
$$

the above properties are true in the infinite-dimensional Hilbert space. Here we consider numerical simulation so our Hilbert space is fixed to hold maximum $$N_p$$ bosons per site and commutation relation becomes

$$
\begin{aligned}
[b_n, b_{n^\prime}] &= 0 \\
[b_n, b_{n^\prime}^\dagger] &= \delta_{nn^\prime} [1 - \frac{N_p + 1}{N_p!} (b_n)^N_p (b_n^\dagger)^N_p]
\end{aligned}
$$

from {% cite Somma_2003 --file references %}.

Where $$[A, B] = AB - BA$$ is the *commutator* and $$\{A, B\} = AB + BA$$ is the *anticommutator*. Also note how bosonic operators do not square to zero!

To use those operators in our simulations we need to translate them into spin-$$\frac{1}{2}$$, we do not worry about bosonic operators because QuTip already provides [creation](http://qutip.org/docs/latest/apidoc/functions.html#qutip.operators.create) and [annihilation](http://qutip.org/docs/latest/apidoc/functions.html#qutip.operators.destroy) operators for $$N$$-level Fock basis for bosons. For fermions (unfortunately) we cannot be so lazy and we need to make them ourselves and for this we need the famous Jordan-Wigner transformation

$$
\begin{aligned}
a_j &= (\prod_{k=1}^{j-1} \sigma^\alpha_k)(\sigma^\beta_j + i \sigma^\gamma_j) \\
a_j^\dagger &= (\prod_{k=1}^{j-1} \sigma^\alpha_k)(\sigma^\beta_j - i \sigma^\gamma_j)
\end{aligned}
$$

I do not like to be attached to any particular basis so I generalized the operators for Jordan-Wigner transformation to Pauli operators labeled $$\sigma^\alpha$$, $$\sigma^\beta$$ and $$\sigma^\gamma$$. The choice of those operators would affect the spin-$$\frac{1}{2}$$ representation of the vacuum state and could potential mix-up the creation and annihilation operators. The choice of $$\sigma^\alpha$$ determines how vacuum is represented, vacuum state would be the state that has $$+1$$ eigenvalue of the $$\sigma^\alpha$$-operator. The occupied state would be one that has $$-1$$ eigenvalue. Regarding the choice of remaining operators, if $$\sigma^\beta \sigma^\gamma = i\sigma^\alpha$$ then $$a$$ and $$a^\dagger$$ will represent *annihilation* and *creation* respectively (as expected). On the other hand choosing $$\sigma^\beta \sigma^\gamma = -i\sigma^\alpha$$ would swap their interpretations.

Let us implement the Jordan-Wigner transformation that works for an arbitrary choice of those operators for a $$N$$-body system. First some spin-$$\frac{1}{2}$$ operators.

```python
import numpy as np
from qutip import qeye, sigmax, sigmay, sigmaz, tensor

def Is(i): return [qeye(2) for j in range(0, i)]
def Sx(N, i): return tensor(Is(i) + [sigmax()] + Is(N - i - 1))
def Sy(N, i): return tensor(Is(i) + [sigmay()] + Is(N - i - 1))
def Sz(N, i): return tensor(Is(i) + [sigmaz()] + Is(N - i - 1))
```

Also, as usual, product, sum and power operators that act on generic `Object`-type which is very useful for defining quantum mechanical operators using QuTip.

```python
def osum(lst): return np.sum(np.array(lst, dtype=object))
def oprd(lst): return np.prod(np.array(lst, dtype=object))
def opow(op, N): return oprd([op for i in range(N)])
```

Using these we can defined Jordan-Wigner operators.

```python
def a_(N, n, Opers=None):
  Sa, Sb, Sc = Sx, Sy, Sz
  if Opers is not None:
    Sa, Sb, Sc = Opers
  return oprd([Sa(N, j) for j in range(n)])*(Sb + 1j*Sc)/2.

def ad(L, n, Opers=None):
  Sa, Sb, Sc = Sx, Sy, Sz
  if Opers is not None:
    Sa, Sb, Sc = Opers
  return oprd([Sa(N, j) for j in range(n)])*(Sb - 1j*Sc)/2.
```

We could have just use `a_.dag()` for the definition of `ad` as they only differ by a sign but I find it beneficial to sometimes follow the definitions strictly even if they differ only by a sign. Benefit of that is it helps to correlate things for those who read this article and see those things for the first time. We also need identity operator, but not just any identity... We need identity that matches the QuTip's quantum object type of Hilbert space of $$N$$ $$2$$-level particles.

```python
def I(N): return osum([Sz(N, i)*Sz(N, i) for i in range(N)])/N
```

Lets write a little test script that checks basic properties of fermions and bosons. It must cover all possible choices of Jordan-Wigner operators (work in any basis) for fermions and for bosons we expect it to cover cases of different number of possible levels. Lets start by making the necessary imports.

```python
import pytest
import math
import itertools

from qm import Sx, Sy, Sz
from qm import a_, ad, b_, bd, I
from qm import opow
from qm import commutator, anticommutator
```

Now parameters, we want to cover cases of $$N = 2 \dots 4$$ particles and in case of bosons $$N_p = 2 \dots 4$$ levels. Using the `itertools` library we prepare all possible permutations of spin operators for Jordan-Wigner and provide them with string type label for test naming.

```python
Ns = [2, 3, 4]
Nps = [2, 3, 4]
jws = itertools.permutations([(Sx, 'X'), (Sy, 'Y'), (Sz, 'Z')])
```

Prepare products of those parameters to cover all cases for fermions and bosons as well as pre-generate the test names. Fermion tests will be dynamically labeled using `fermionTestName` function that accepts parameters and returns test case name as a string.

```python
def fermionTestName(param):
    N, jw = param
    (_, la), (_, lb), (_, lc) = jw
    return 'N={0},JW={1}'.format(str(N), la + lb + lc)

fermion_params = list(itertools.product(Ns, jws))
fermion_names = [fermionTestName(param) for param in fermion_params]

boson_params = list(itertools.product(Ns, Nps))
boson_names = ['N={0},Ns={1}'.format(str(N), str(Ns)) for N, Ns in boson_params]
```

Finally, let us implement test for fermion operators. It is a literal implementation of fermion operator properties stated above. Note how zero operator is prepared by multiplying identity by constant zero

```python
@pytest.mark.parametrize('N,jw', fermion_params, ids=fermion_names)
def testFermions(N, jw):
    (Sa, _), (Sb, _), (Sc, _) = jw
    Opers = Sa, Sb, Sc
    zero = 0.*I(N)
    # test all the pairs
    for n in range(N):
        a_n = a_(N, n, Opers=Opers)
        adn = ad(N, n, Opers=Opers)
        for np in range(N):
            a_np = a_(N, np, Opers=Opers)
            adnp = ad(N, np, Opers=Opers)
            assert anticommutator(a_n, a_np) == zero
            if n == np:
                assert anticommutator(a_n, adnp) == I(N)
            else:
                assert anticommutator(a_n, adnp) == zero
        assert a_n*a_n == zero
        assert adn*adn == zero
```

we proceed in the same way to design test for bosons, following the finite Hilbert space boson commutation relations stated above

```python
@pytest.mark.parametrize('N,Np', boson_params, ids=boson_names)
def testBosons(N, Np):
    zero = 0.*b_(N, Np, 0)*bd(N, Np, 0)
    # test all the pairs
    for n in range(N):
        b_n = b_(N, Np, n)
        bdn = bd(N, Np, n)
        for np in range(N):
            b_np = b_(N, Np, np)
            bdnp = bd(N, Np, np)
            # test anticommutation properties
            assert commutator(b_n, b_np) == zero
            LHS = commutator(b_n, bdnp)
            RHS = zero
            if n == np:
                NpF = math.factorial(Np)
                RHS = (1. - ((Np+1)/NpF)*opow(bdn, Np)*opow(b_n, Np))
            assert LHS == RHS
```



```console
$ python -m pytest tests.py -v
============================= test session starts ==============================
platform darwin -- Python 3.5.3, pytest-4.2.0, py-1.8.0, pluggy-0.12.0 -- /Users/marek/.pyenv/versions/3.5.3/bin/python
cachedir: .pytest_cache
metadata: {'Platform': 'Darwin-18.6.0-x86_64-i386-64bit', 'Python': '3.5.3', 'Packages': {'pluggy': '0.12.0', 'py': '1.8.0', 'pytest': '4.2.0'}, 'Plugins': {'steps': '1.6.4', 'timeout': '1.3.3', 'html': '1.22.1', 'metadata': '1.8.0', 'dynamodb': '1.2.0', 'flaky': '3.6.1'}}
rootdir: /Users/marek/Development/bloggists/jordan-wigner, inifile:
plugins: steps-1.6.4, timeout-1.3.3, dynamodb-1.2.0, flaky-3.6.1, html-1.22.1, metadata-1.8.0
collected 27 items                                                             

tests.py::testFermions[N=2,JW=XYZ] PASSED                                [  3%]
tests.py::testFermions[N=2,JW=XZY] PASSED                                [  7%]
tests.py::testFermions[N=2,JW=YXZ] PASSED                                [ 11%]
tests.py::testFermions[N=2,JW=YZX] PASSED                                [ 14%]
tests.py::testFermions[N=2,JW=ZXY] PASSED                                [ 18%]
tests.py::testFermions[N=2,JW=ZYX] PASSED                                [ 22%]
tests.py::testFermions[N=3,JW=XYZ] PASSED                                [ 25%]
tests.py::testFermions[N=3,JW=XZY] PASSED                                [ 29%]
tests.py::testFermions[N=3,JW=YXZ] PASSED                                [ 33%]
tests.py::testFermions[N=3,JW=YZX] PASSED                                [ 37%]
tests.py::testFermions[N=3,JW=ZXY] PASSED                                [ 40%]
tests.py::testFermions[N=3,JW=ZYX] PASSED                                [ 44%]
tests.py::testFermions[N=4,JW=XYZ] PASSED                                [ 48%]
tests.py::testFermions[N=4,JW=XZY] PASSED                                [ 51%]
tests.py::testFermions[N=4,JW=YXZ] PASSED                                [ 55%]
tests.py::testFermions[N=4,JW=YZX] PASSED                                [ 59%]
tests.py::testFermions[N=4,JW=ZXY] PASSED                                [ 62%]
tests.py::testFermions[N=4,JW=ZYX] PASSED                                [ 66%]
tests.py::testBosons[N=2,Ns=2] PASSED                                    [ 70%]
tests.py::testBosons[N=2,Ns=3] PASSED                                    [ 74%]
tests.py::testBosons[N=2,Ns=4] PASSED                                    [ 77%]
tests.py::testBosons[N=3,Ns=2] PASSED                                    [ 81%]
tests.py::testBosons[N=3,Ns=3] PASSED                                    [ 85%]
tests.py::testBosons[N=3,Ns=4] PASSED                                    [ 88%]
tests.py::testBosons[N=4,Ns=2] PASSED                                    [ 92%]
tests.py::testBosons[N=4,Ns=3] PASSED                                    [ 96%]
tests.py::testBosons[N=4,Ns=4] PASSED                                    [100%]

========================== 27 passed in 4.72 seconds ===========================
```

We tested commutation / anticommutation properties of different kinds of indistinguishable particles. We defined QuTip operators for fermions using Jordan-Wigner operators. We have not simulated any physical system although using those operators we can start working with different physical systems in 1D geometry. Jordan-Wigner transformation although primarily designed for 1D systems, it can be extended to 2D {% cite Azzouz1993 --file references %}.

I am strongly attached to the idea of testing operator properties. Writing `pytest` tests takes huge amount of my daily research work but it is a good time investment. I am a huge advocate of [test-driven development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development) and I brought those habits with me to research work as I career switched from software engineering. Research is primarily exploration of the unknown. Doing research requires exploring ideas which have never been tried before. In case of numerical simulations it remains true. I do not believe it would be possible to get right results if at least one operator is not working properly. I observe people working with tools like Mathematica notebooks or Jupyter, often looping around being blocked on same bug resulting from some old state being preserved as one of the cells was not re-run after changes were made. I do large part of my numerics with `pytest`, whenever I test a hypothesis literally everything is being tested so that if I temporarily change something to try an idea test and if that breaks anything then `pytest` will catch it immediately.

For the complete source code please consult the [gist repository](https://gist.github.com/marekyggdrasil/ed36ee96444e445da1dcd59e40802c1d).

{% bibliography --file references --file wiki --cited %}
