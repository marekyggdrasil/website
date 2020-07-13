---
layout: post
title: Satisfiability
tags: [Tutorial, Optimization, SAT, Computer Science, Formal Language Theory]
description: A brief introduction to variants of boolean satisfiability problems, including circuit satisfiability, reduction to maximum independent set and Tseytin transformation.
lang: en_US
lang-ref: satisfiability
image: /assets/figures/png/sat-drawing-mis-reduction.png
katex: true
gist: https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05
---

In mathematical logic satisfiability of an expression indicates if there exists an assignment of values, or simply, an input for which such expression yields a particular result. As I like to think about it, it is a logic generalization of the common concept of solving an equation. Determining if a formula is *satisfiable* is a decidable problem but computationally hard. In this article we are going to focus on *boolean satisfiability*, which to my knowledge, is the most studied problem in computer science. It is not a purely theoretical concept, it has industrial application and it has potential to inspire you for more projects and perhaps help you solve problems in projects you are already working on. We already did demonstrate solving graph problems [finds use in level design in games](/2020/04/22/vertex-cover-independent-set-game-level-design/), same could be true for *boolean satisfiability* or simply $$ \text{SAT} $$ problem.

An instance of such problem is a boolean formula expressions written in *conjunctive normal form* or a *clausal form* meaning it is a conjunction of clauses and each clause is a disjunction of literals.

$$ \Phi(x_1 \dots x_N) = \bigwedge\limits_{\mathfrak{C}_k \in \mathfrak{C}(\Phi)} ( \bigvee\limits_{l \in \mathfrak{C}_k} l ) \tag{SAT} $$

$$ \forall \mathfrak{C}_k \in \mathfrak{C}(\Phi), \forall l \in \mathfrak{C}_k, l \in \{x_n \vert 1 \leq n \leq N \} \cup \{ \neg x_n \vert 1 \leq n \leq N\} $$

A conjunction is logical `AND` operation denoted above as $$ \wedge $$ and disjunction is logical `OR` operation denoted as $$ \vee $$. Parentheses indicate *clauses* which consist of *literals* where a literal is a symbol pointing a variable or negation of variable. For the `CNF` expression $$ \Phi $$ I picked a set-theoretical description which denotes the set of clauses $$ \mathfrak{C}(\Phi) $$ in which clauses are sets of literals $$ \mathfrak{C}_k \in \mathfrak{C}(\Phi) $$. It is allowed to have more literals than variables.

The easiest way to get started is by playing with small examples. For instance, a $$\Phi$$ expression is an instance of $$ \text{3SAT} $$ problem when $$ \vert \mathfrak{C}_k \vert = 3, \forall \mathfrak{C}_k \in \mathfrak{C}(\Phi) $$. Such formula has exactly three literals per clause.

$$
\begin{aligned}
\Phi_1 &= (x_1 \vee \neg x_2 \vee x_3) \wedge (x_1 \vee x_2 \vee \neg x_3) \wedge (\neg x_1 \vee \neg x_2 \vee x_3)
\end{aligned}
$$

Let us computationally solve, personally I use the [MergeSat solver](https://github.com/conp-solutions/mergesat). First we write $$ \Phi_1 $$ in `.cnf` format which is accepted by the solver. Each line ends with `0`, negative numbers are negations, each line is a clause except for first line which indicates the `CNF` form with number of variables and number of clauses.

```cnf
p cnf 3 3
1 -2 3 0
1 2 -3 0
-1 -2 3 0
```

The `CNF` formula $$ \Phi_1 $$ is `satisfiable`, output from Mergesat indicating this as well as assignment of values to variables $$ x_1 = 1, x_2 = 0, x_3 = 0 $$ can be found in the [MergeSat output](https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05#file-phi1out-txt-L31-L32). Of course this does not have to be the only solution that satisfies $$ \Phi_1 $$. Now, for contrast, let us consider $$ \Phi_2 $$ which will involve two variables and four clauses.

$$
\begin{aligned}
\Phi_2 &= (x_1 \vee x_2) \wedge (x_1 \vee \neg x_2) \wedge (\neg x_1 \vee x_2) \wedge (\neg x_1 \vee \neg x_2)
\end{aligned}
$$

```cnf
p cnf 2 4
1 2 0
1 -2 0
-1 2 0
-1 -2 0
```

Unlike in case of the previous form, the $$ \Phi_2 $$ is not satisfiable which is indicated by the [MergeSat output](https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05#file-phi2out-txt-L25). It can also be deduced intuitively by looking at clauses of $$ \Phi_2 $$, they list all the possible assignments of values for two boolean variables making it not possible to satisfy a clause without violating another. It is also worth noting that the $$ \text{2SAT} $$ problem can be efficiently solved in polynomial time (linear) as it is a special case of $$ \text{SAT} $$ {% cite Krom1967 Aspvall1979 Even1976 --file references %}

For now let us ommit the `CNF` form and define the *circuit satisfiability problem* in a similar way, a boolean formula $$ \Phi_\text{circuit}(x_1 \dots x_N) $$ is a word of a formal context-free grammar of a form reassembling to the [syntax trees](https://en.wikipedia.org/wiki/Context-free_grammar#Derivations_and_syntax_trees) involving boolean operators $$\wedge$$, $$\vee$$ and $$\neg$$ with literals as terminal symbols. Formal language is defined by a quadruple $$(V, N, {, S})$$ where $$N$$ is a finite set of non-terminal symbols, $$V$$ is a finite set of terminal symbols, $$P$$ is a set of production rules and finally $$S$$ is the start symbol.

$$
\begin{aligned}
G &= (V, N, P, S) \tag{CSAT grammar} \\
N &= \{\mathfrak{S}\}  \\
V &= \{x_n \vert 1 \leq n \leq N \} \cup \{ \neg, \vee, \wedge, (, )\}\\
P &= \{\mathfrak{S} \rightarrow (\mathfrak{S} \wedge \mathfrak{S}), \mathfrak{S} \rightarrow (\mathfrak{S} \vee \mathfrak{S}), \mathfrak{S} \rightarrow \neg \mathfrak{S} \} \cup \{ \mathfrak{S} \rightarrow x_n \vert 1 \leq n \leq N \} \\
S &= \{\mathfrak{S}\}
\end{aligned}
$$

Worth noting that $$ \text{CSAT} $$ is a generalization of $$ \text{SAT} $$ meaning that every possible $$ \Phi(x_1 \dots x_N) $$ is a word of $$ \text{CSAT grammar} $$ and this is because every boolean formula in *conjunctive normal form* forms a valid syntax tree described by the $$ \text{CSAT grammar} $$.

{% include figure.html url="#" fll="/assets/figures/svg/sat-circ-trivial.svg" alt="simplest satisfiability circuits" %}

$$
\begin{aligned}
\Phi_3 &= ( x_1 \wedge x_2) \\
\Phi_4 &= ( x_1 \wedge \neg x_1)
\end{aligned}
$$

The circuit $$\Phi_3$$ is satisfied by an assignment of values to variables $$x_1 = x_2 = 1$$. In the case of $$\Phi_4$$ which involves only a single boolean variable $$x_1$$ and none of its values satisfies the circuit.

The $$ \text{SAT} $$ problem might be the most studied problem so far, the number of available solvers and heuristics is vast and it makes it attractive to formalize our formal satisfaction problems as $$ \text{SAT} $$ formulas and take advantage of decades of research and use those solvers to process them efficiently. What about the $$ \text{CSAT} $$? We made a formal argument that every $$ \text{SAT} $$ is $$ \text{CSAT} $$ but what about the other way around?

We could apply DeMorgan's Law and the distributive property to simplify formula generated by the $$ \text{CSAT grammar} $$ and eventually rewriting it into CNF form. This unforuntately could lead to exponential growth of formula size. This is where the discovery of Tseytin Transformation {% cite Tseitin1983 --file references %} which is an efficient way of converting *disjunctive normal form* `DNF` into *conjuctive normal form* `CNF`. We can imagine a sequence of algebraic transformations such as variables substitutions in order to translate an arbitrary $$ \Phi_\text{circuit}(x_1 \dots x_N) $$ into $$ \Phi(x_1 \dots x_N) $$. Problem with that is the number of involved variables would grow quadratically and this is an NP-Complete problem, every extra variable is increasing the number of required steps to solve by an order of magnitude. The advantage of Tseytin Transformation is that CNF formula grows linearly compared to input $$ \text{CSAT} $$ instance expression. It consists of assigning outputs variables to each component of the circuit and rewriting them according to the following rules of [gate sub-expressions](https://en.wikipedia.org/wiki/Tseytin_transformation#Gate_Sub-expressions).

$$
\begin{aligned}
A \cdot B = C &\equiv (\neg A \vee \neg B \vee C) \wedge (A \vee \neg C) \wedge (B \vee \neg C) \\
A + B = C &\equiv (A \vee B \vee \neg C) \wedge (\neg A \vee C) \wedge (\neg B \vee C) \\
\neg A = C &\equiv (\neg A \vee \neg C) \wedge (A \vee C) \tag{Tseytin}
\end{aligned}
$$

Where $$ A \cdot B = C $$, $$ A + B = C $$ and $$ \neg A = C$$ are $$ \text{AND} $$, $$ \text{OR} $$ and $$ \text{NOT} $$ classical gates with $$ C $$ as a single output. I will not provide a rigorous proof for those sub-expressions, rather provide some intuition, which is if we think of satisfying all the clauses then each of the gate sub-expressions becomes a logic description of the gate consisting of relevant cases, which are clauses, of how the inputs $$ A, B $$ affect the output $$ C $$. Let us analyse clasuses of the $$ \text{AND} $$ gate. Clause $$ (\neg A \vee \neg B \vee C) $$ indicates that if both $$ A $$ and $$ B $$ are `true`, then the only way to satisfy this clause is to assign $$ C $$ to `true` as well, which already seems to implement $$ \text{AND} $$ pretty well. The remaining case to ensure is enforcing $$ C $$ to `false` when at least one of inputs is `false` and this is done by clauses $$ (A \vee \neg C) \wedge (B \vee \neg C) $$.

Let us examine an [example](https://en.wikipedia.org/wiki/Tseytin_transformation#Simple_combinatorial_logic) of $$ \text{CSAT} $$ circuit converted into $$ \text{SAT} $$ problem using Tseytin transformation. The example circuit consists of three inputs $$ x_1, x_2, x_3 $$, a single output $$ y $$ and seven logic gates. Circuit takes the following form.

{% include figure.html url="#"
min-width="70%" fll="/assets/figures/svg/sat-circ.svg" alt="satisfiability circuit" %}

Note that one of the gates has two outputs. Formula corresponding to this circuit can be written in a predicate form which is recognizable by the $$ \text{CSAT grammar} $$ which is no longer in `CNF` form.

$$
\begin{aligned}
\Phi_5 &= ( ( ( \neg x_1 \wedge x_2 ) \vee ( x_1 \wedge \neg x_2 ) ) \vee ( \neg x_2 \wedge x_3 ) )
\end{aligned}
$$

Applying the Tseytin transformation to this circuit leads to expression with more variables but in *conjunctive normal form*. It requires to apply the gate sub-expressions to each of gates of the circuit leading us to following `CNF` form.

$$
\begin{aligned}
\Phi_\text{N1} &= (z_1 \vee x_1) \wedge (\neg z_1 \vee \neg x_1) \\
\Phi_\text{N2} &= (z_2 \vee x_2) \wedge (\neg z_2 \vee \neg x_2) \wedge (z_3 \vee x_2) \wedge (\neg z_3 \vee \neg x_2) \\
\Phi_\text{A1} &= (\neg z_1 \vee \neg x_2 \vee z_4) \wedge (z_1 \vee \neg z_4) \wedge (x_2 \vee \neg z_4) \\
\Phi_\text{A2} &= (\neg x_1 \vee \neg z_2 \vee z_5) \wedge (x_1 \vee \neg z_5) \wedge (z_2 \vee \neg z_5) \\
\Phi_\text{A3} &= (\neg z_3 \vee \neg x_3 \vee z_6) \wedge (z_3 \vee \neg z_6) \wedge (x_3 \vee \neg z_6) \\
\Phi_\text{R1} &= (z_4 \vee z_5 \vee \neg z_7) \wedge (\neg z_4 \vee z_7) \wedge (\neg z_5 \vee z_7) \\
\Phi_\text{R2} &= (z_7 \vee z_6 \vee \neg y) \wedge (\neg z_7 \vee y) \wedge (\neg z_6 \vee y)
\end{aligned}
$$

Each of $$ \Phi_\text{N1}, \Phi_\text{N2} \dots \Phi_\text{R2} $$ is an individual `CNF` form corresponding to particular gate of the circuit. The penalty for such convenient notation is in the extra variables $$ z_i $$ that have to be introduced. We can assemble those forms into $$ \Phi_4^\prime \equiv \Phi_4 $$ and the [cnf file](https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05#file-phi5-cnf) is also provided.

$$
\begin{aligned}
\Phi_5 &= \Phi_\text{N1} \wedge \Phi_\text{N2} \wedge \Phi_\text{A1} \wedge \Phi_\text{A2} \wedge \Phi_\text{A3} \wedge \Phi_\text{R1} \wedge \Phi_\text{R2} \wedge (y)
\end{aligned}
$$

This circuit outputs `true` if given one of the following four inputs $$(x_1, x_2, x_3) \in \{(0, 0, 1), (0, 1, 0), (1, 0, 0), (1, 0, 1), (0, 1, 1)\}$$. We can enumerate all the solutions by appending a clause that is invalid if given a previous solution until the formula becomes unsatisfiable. For instance if we append clauses $$ (x_1 \vee x_2 \vee \neg x_3) \wedge (x_1 \vee \neg x_2 \vee x_3) \wedge (\neg x_1 \vee x_2 \vee x_3) $$ we will get the following form.

$$
\begin{aligned}
\Phi_5^\prime &= \Phi_\text{N1} \wedge \Phi_\text{N2} \wedge \Phi_\text{A1} \wedge \Phi_\text{A2} \wedge \Phi_\text{A3} \wedge \Phi_\text{R1} \wedge \Phi_\text{R2} \wedge (y) \wedge  (x_1 \vee x_2 \vee \neg x_3) \wedge (x_1 \vee \neg x_2 \vee x_3) \wedge (\neg x_1 \vee x_2 \vee x_3)
\end{aligned}
$$

Which is satisfiable only by solutions $$(x_1, x_2, x_3) \in \{(1, 0, 1), (0, 1, 1)\}$$ as assignments $$\{(0, 0, 1), (0, 1, 0), (1, 0, 0)\}$$ have been excluded by the appended clauses. Following this logic the following formula includes clauses from all the solutions and is [unsatisfiable](https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05#file-phi5_5out-txt-L26). The [formula file](https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05#file-phi5-cnf) is also provided.

$$
\begin{aligned}
\Phi_5^{\prime\prime} = &\Phi_\text{N1} \wedge \Phi_\text{N2} \wedge \Phi_\text{A1} \wedge \Phi_\text{A2} \wedge \Phi_\text{A3} \wedge \Phi_\text{R1} \wedge \Phi_\text{R2} \wedge (y) \\
&\wedge  (x_1 \vee x_2 \vee \neg x_3) \wedge (x_1 \vee \neg x_2 \vee x_3) \wedge (\neg x_1 \vee x_2 \vee x_3) \wedge (\neg x_1 \vee x_2 \vee \neg x_3) \wedge (x_1 \vee \neg x_2 \vee \neg x_3)
\end{aligned}
$$



Let $$ \Phi(x_1 \dots x_N) $$ be an arbitrary $$ \text{SAT} $$ instance formula and we want to find an instance of $$ (\text{MIS}) $$ *maximum independent set* as defined in [Vertex Covers and Independent Sets in game level design](/2020/04/22/vertex-cover-independent-set-game-level-design/). We know both of those problems are NP-Complete thus by {% cite Kar72 --file references %} there must exist a transformation, or a *polynomial reduction* which transforms one instance into another in polynomial number of steps in terms of data size. As a brief reminder, the target instance of $$ (\text{MIS}) $$ takes form of a graph $$G = (E(G), V(G))$$ where $$ E(G) $$ is set of edges of $$G$$ and $$V(G)$$ is a set of vertices of $$G$$. We construct $$G$$ from $$ \Phi(x_1 \dots x_N) $$ according to the following definition.

$$
\begin{aligned}
G &= (E, V) \\
E &= \{\{l_\alpha, l_\beta\} \vert (\exists \mathfrak{C}_k \in \mathfrak{C}(\Phi), \{l_\alpha, l_\beta\} \subseteq \mathfrak{C}_k) \vee (\exists \mathfrak{C}_k, \mathfrak{C}_{k^\prime} \in \mathfrak{C}(\Phi), k \neq k^\prime, l_\alpha \in \mathfrak{C}_k, l_\beta \in \mathfrak{C}_{k^\prime}, l_\alpha = \neg l_\beta)\} \\
V &= \{l_{km} \vert l_{km} \in \mathfrak{C}_k, \forall \mathfrak{C}_k \in \mathfrak{C}(\Phi) \}
\end{aligned}
$$

If the *maximum independent set* of the graph described above is of size equal to number of clauses of the CNF formula $$ \Phi(x_1 \dots x_N) $$ then $$ \Phi $$ is satisfiable, i.e $$ \vert \mathfrak{C}(\Phi) \vert = V_\text{MIS}(G) $$. Reason for that is all the literals inside of clause are coupled together making it impossible to select more than one vertex without violating the *independent set* constraint. The lowest number of violations is zero and this corresponds to exactly one literal per clause.

{% include figure.html url="#"
min-width="80%" fll="/assets/figures/png/sat-drawing-mis-reduction.png" alt="Reduction from SAT to MIS" %}

The process illustrated on figure above consisists of assigning a graph vertex for each literal, then connecting the vertices corresponding to literals of same clasuses with an edge - this creates the *clause clusters*. Then we join with an edge all vertices corresponding to negation of corresponding variables, those we call *conflict links*. The graph obtained from the method above can be re-arranged for better aesthetics.

{% include figure.html url="#"
max-width="35%" fll="/assets/figures/png/sat-drawing-mis-graph.png" alt="Reduction from SAT to MIS" %}

If you read the [Vertex Covers and Independent Sets in game level design](/2020/04/22/vertex-cover-independent-set-game-level-design/) article you know that *maximum independent set problem* is semantically equivalent to *minimum vertex cover problem* thus this reduction can be treated as reduction to either of those problems. Moreover, if you checked the [using Quantum Adiabatic Optimization](/2020/05/22/quantum-adiabatic-optimization/) article you know that such problem could be solved using a quantum adiabatic processor, which already reveals one of possible options of solving $$ \text{SAT} $$ problems using a quantum machine.

We also can connect this problem to our [constraint programming](/2020/06/22/constraint-programming/) post. The $$ \text{CSAT} $$ is essentially a predicat and could implement a constraint. Thus an instance of $$ \text{CSP} $$ could be used to design a boolean circuit which is satisfied when the $$ \text{CSP} $$ instance is satisfied, then via Tseytin transformation we can convert it to `CNF` formula and even reduce to $$ \text{MIS} $$ instance. As long as the domains of variables of your $$ \text{CSP} $$ are boolean this should be relatively straightforward process, but once your constraints involve integer domain your circuits would start to look like computer processor circuits, they would involve arithmetic-logic components and grow beyond what is currently trackable.

Another interesting thing we can think about is, as the $$ \text{SAT} $$ and $$ \text{CSAT} $$ instances are described using a context-free grammar $$ \text{CSAT grammar} $$ determining if any $$ \Phi $$ is an instance of *boolean satisfiability* can be done efficiently by checking if $$ \Phi $$ is a word described by $$ \text{CSAT grammar} $$. Yet if we introduce a subtle change in the definition and claim that $$ \text{CSAT grammar} $$ describes only formulas $$ \Phi^\prime $$ which are satisfiable, suddenly checking if $$ \Phi $$ is a word of this grammar becomes computationally hard problem. My conclusion from this is quite direct and philosophical - that finding an example of a hard problem is equivalent of solving a hard problem.

This blog post barely scratches the surface of decades of research on the concept of *satisfiability* and logic programming, but it does provide the necessary definitions and gives us foundations to proceed to many other interesting concepts. One possible direction I consider at the moment of writing this post are hashing functions, cryptography and blockchains, another one is computational complexity theory and yet another one is formalizing satisfiability of circuits from category-theoretical perspective. If you have any preferences regarding the direction this blog goes or if you have any questions or find errors or typos please do not hesitate to let me know via Twitter or GitHub. All the `.cnf` files from this post are available in [this public gist repository](https://gist.github.com/marekyggdrasil/e82e851522be9195239aba88e1f34c05).

{% bibliography --file references --file wiki --cited %}
