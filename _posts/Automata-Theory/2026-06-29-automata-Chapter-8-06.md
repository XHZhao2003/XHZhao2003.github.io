---
layout: post
title: "8.5 Restricted Turing Machine"
category: "automata-theory"
order: "806"
math: true
---

- In this section, we shall consider some examples of restrictions on the TM that also have the same language-recognizing power.

---

### Turing Machines With Semi-infinite Tapes

**Theorem 8.12**: Every language accepted by a TM $M_2$ is also accepted by a TM $M_1$ with the following restrictions.

- $M_1$'s head never moves left of its initial position.

  > The tape of $M_1$ is thus called semi-infinite, as it has a left end.

- $M_1$ never writes blanks.

**Proof**:

- For the second restriction, we just create a new tape symbol $B'$ that functions as a blank, and replace all $B$ by $B'$.
- For the first restriction, we will create two tracks on $M_1$'s semi-infinite tape.
  - The first track represents the cells of the original TM that are at or to the right of the initial head position, i.e. $X_0, X_1, \cdots$.
  - The second track represents the positions left of the initial position, but in reverse order, i.e. $X_{-1}, X_{-2}, \cdots$.
  - The first symbol in the second track is a placeholder $\ast$.

- Design the state set as $Q_1 = \lbrace q_0, q_1 \rbrace\, \cup\,\big(Q_2\times \lbrace U, L \rbrace\big)$.
  - $q_0$ is the initial state of $M_1$. $q_1$ is just another state. $Q_2$ is the state set of $M_2$.
  - $U$ means the head of $M_2$ is at or to the right of its initial position. 
  - $L$ means the head of $M_2$ is to the left of its initial position. 
- Design the tape alphabet as $\Gamma_1 = \Gamma_2 \times \Gamma_2$.
  - It's natural since we have a 2-track tape.
  - Additionally, for all $X \in \Gamma_2$, there is a pair $(X, \ast)$ in $\Gamma_1$ where $\ast$ is a new tape symbol. This is for the case where the tape write the initial position.
- The input symbols in $\Sigma_1$ are those pairs with an input symbol of $M_2$ in the first component and a blank in the second component.
- The blank symbol in $B_2 = (B_1, B_1)$.
- The transition funciton is as follows.
  - The first move is always $\delta_1(q_0, (a, B)) = (q_1, (a, \ast), R)$ for any $a\in \Sigma$.
    - It puts a $\ast$ in the second track's leftmost cells and moves right.
  - The second move is always $\delta_1(q_1, (X, B)) = ((q_2, U), (X, B), L)$ for any $X \in \Gamma_2$.
    - $q_2$ is the initial state of $M_2$
    - And it moves back to the initial position.
  - Then, for each move of $M_2$, $M_1$ have enough information to simulate it.
- The accepting states is $F_1 = F_2 \times \lbrace U, L \rbrace$.

---

### Multistack Machines

- We now consider models that are generalizations of PDAs.
- First, we consider what happens when a PDA has multiple stacks.
- A deterministic $k$-stack PDA is most the same as a DPDA. But a move of a $k$-stack PDA is based on
  - The state of the finite control.
  - The input symbol read, which cannot be $\epsilon$.
  - The top stack symbol on each of its stacks.
- In a move, the multistack machine can
  - Change to a new state.
  - Replace the top symbol of each stack with a string of zero of more stack symbols. Replacement for each stack can be different.
- We add one assumption for simplicity: there is a special endmarker $\$$ that appears only at the end of the input and is not part of input.

---

**Theorem 8.13**: If a language $L$ is accepted by a TM $M$, then $L$ is accepted by a two stack machine $S$.

**Proof**:

- The essential idea is that two stacks can simulate one TM tape, using the same strategy in proof of Theorem 8.12.
- $S$ begins with the endmarker on each stack.
- The input $w$ is first put in the first stack, i.e. $w\$$. $S$ pops each symbol and pushes into the second stack to get $\$w^R$, with the left end of $w$ at the top.
  - The first stack is for the positions left to the tape head.
  - The second stack is for the positions at or right to the tape head.
- Then for any move of $M$, $S$ has enough information to simulate.
- $S$ accepts if the new state of $M$ is accepting.

---

### Counter Machines

A counter machine may be thought of in two ways:

- The counter machine has the same structure as the multistack machine, but inplace of each stack is a counter. 
- Counters hold nonnegative integers, but the finite control can only distinguish between zero and nonzero values. 
- That is, the move of the counter machine depends on its state, input symbol, and whether the counters are zero. 
- In one move, the counter machine changes state, and adds or subtracts 1 from any its counters independently. But the counters are not allowed to be negative.

The other way is to regard it as a restricted multistack machine. The restrictions are as follows:

- There are only two stack symbols $Z_0$ (the endmarker) and $X$.
- $Z_0$ is initially on each stack.
- We may replace $Z_0$ only by strings of the form $X_iZ_0$ for some $i \geq 0$.
- We may replace $X$ only by $X_i$ for some $i \geq 0$.

---











