---
layout: post
title: "8.3 Programming Techniques for Turing Machines"
category: "automata-theory"
order: "804"
math: true

---

- Our goal in subsequent sections is to convince you that a TM is exactly as powerful as a convenrional computer.
- We present several techniques that allow TMs to have additional features, but do not extend the power of the TM model.
- We shall also learn the "introspective" ability of TM machines, i.e., performing calculations that simulate another TM machine.

---

### Storage in the State

- We can use the finite control not only to represent a position in the "program" of the TM, but to hold a finite amount of data.
- The technique requires no extention to the TM model.
- We think each state as a tuple that have two components.
  - A control portion. This portion remembers the current state, i.e., what the original TM is doing.
  - A data portion. This portion remembers all data we want to store in the TM. It can have multiple elements of data.
- Therefore, the set of states $Q$ will become $Q \times \mathcal{D}_1 \times \mathcal{D}_2 \times \cdots \times \mathcal{D_n}$, where $\mathcal{D}_i$ is the set of all possible values of the $i$-th data elements we want to store.  
- To "read" the data, the transition function may behave differently on tuples that have the same control portion but different data portion.
- To "write" the data, the transition function can choose tuples with any possible value of the data portion.

---

### Multiple Stacks

- The tape of a TM can be thought as composed of serverl tracks.
- We think the tape symbol to be tuples.
- The tape alphabet becomes $\Gamma^n$ for an $n$-track TM.
  - Each track holds one tape symbol
- In each move, $n$ symbols are read and written simultaneously.
- Note: There is only one tape head that moves either left or right in a single move.

---

### Subroutines

- We can build a TM from a collection of interacting subroutines (other TMs).
- Say TM $A$ wants to call TM $B$ as a subroutine.
- We add directly the states and transition functions of $B$ to $A$.
- For the state where $A$ wants to call $B$, we let this state move to the initial state of $B$.
  - The "call" of subroutine $B$ occurs whenever there is a transition to its initial state.
- To allow $B$ to return control to $A$, we add another "return" state.
  - $B$ always moves to the return state when it halts.
  - The TM will move from the return state to another state that is under the caller's control.
- When $B$ is called multiple times,  the design above cannot remember the "return address".
- Therefore, we make copies of the subroutine for each call.

---

### Example for Subroutines

- We shall design a TM to implement the function of multiplication.
- The TM starts with $0^m10^n1$ on its tape, and ends with $0^{mn}$ on its tape.

The outline of the strategy is as follows.

- In general, the tape will have one nonblank string of the form $0^i10^n10^{kn}$ for some $k$.
- In one basic step, we change a 0 in the first group to $B$ and add $n$ 0's to the last group, giving a string of the form $0^{i-1}10^n10^{(k+1)n}$.
- When the all 0's in the first group are changed to blanks, we will then have copied the group of $n$ 0's to then end $m$ times.
- The final step is to change the 1's in $10^n1$ to blanks.

The heart of this algorithm is a subroutine which we shall call `Copy`.

- `Copy` converts an ID $0^{m-k}1q_10^n10^{(k-1)n}$ to ID $0^{m-k}1q_20^n10^{kn}$.
  - $q_1$ is initial state of the TM of `Copy`.
  - $q_2$ is the state where the TM eventually halts.
- We will only need single copy of TM of the subroutine `Copy` to implement multiplication.
  - Because every time `Copy` returns, the caller's control is the same, or the return address is the same.

