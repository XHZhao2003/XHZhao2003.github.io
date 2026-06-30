---
layout: post
title: "8.4 Extentions to the Basic Turing Machine"
category: "automata-theory"
order: "805"
math: true

---

- We shall introduce serveral extensions to the basic Turing Machine model.
- However, they add no language-defining power to the basic model.

---

### Multitape Turing Machines

- A multitape TM has a finite number of tapes, as well as tape heads.

- Initially, the input is placed on the first tape. All other tapes hold blanks.

- The tape head of the first tape is at the left end of the input. It does not matter where other heads are placed initially.

- In a move:

  - The control enters a new state according to the current state and all symbols scanned by each of the tape heads.

  - On each tape, write a new tape symbol.

  - Each of the tape heads makes a move independently.

    > The difference between a Multitape TM and a Multistack TM lies here. A multitape TM extends the TM model but a multistack does not.

- We allow stationary move in multitape TM. The equivalence has been discussed.

---

### Equivalence of TM and Multitape TM

- Surely, multitape TM's accept all the recursively enumerable languages.
  - An one-tape TM is a multitape TM by definition.
- The theorem we will prove shows that multitape TM cannot accept languages other than RE languages.

**Theorem 8.9**: Every language accepted by a multitape TM is recursively enumerable.

**Proof**:

- Suppose $M$ is a $k$-tape TM with $L=L(M)$.
- We simulate $M$ with a multistack and one-tape TM $N$ whose tape has $2k$ tracks.
  - Half the tracks holds the tapes of $M$.
  - The other half of the tracks each hold only a single marker to indicate where the head of the corresponding tape is currently located.
- The problem is that, while $k$ heads in $M$ moves independently, there is only one head in $N$. Although we "mark" the positions of heads using $k$ extra tracks, we always need to start from the single head and find all these marks.
  - $N$ must remember how many markers are to the left or right of its own head.
  - $N$ can store this count in finite control, as discussed in Section 8.3.
  - With this information, in each move, $N$ can visit all markers on its tape, simulate all tapes of $M$, and update the stored count.
  - Finally, $N$ should move on the tape so as to satisfy its own tape head move. 
- The accepting states of $N$ are those that record $M$'s state as one of the accepting states of $M$.

---

### A Reminder About Finiteness

- In constructing a multistack TM to simulate a multitape TM, one may propose the following.
  - Store all $k$ positions of markers as integers in the finite control.
  - Carelessly, he may argue that after any $n$ moves, the head only has to store finite possibilities of integer values.
- The problem is that, while the positions are finite at any time, the complete set of positions at any time is infinite.
- A value that is finite at any time is different from a value that is finite.

---

### Running Time and Time complexity

- We not introduce two important concepts, the **running time** and the "**time complexity**" of a Turing Machine.
- The **running time** of TM $M$ on input $w$ is the number of steps that $M$ makes before halting.
- The **time complexity** of TM $M$ is the function $T(n)$ that is the maximum, over all inputs $w$ of length $n$, of the running time of $M$ on $w$.
  - For TMs that do not halt on all inputs, $T(n)$ may be infinite.

---

### Time Complexity of the Many-Tapes-to-One Construction

- The construction above seems clumsy, as the one-tape TM takes much more running time than the multitape TM.

- However, we will show the amounts of times taken by two TMs are commensurate, as it only increases polynomially.

  >We shall see in Chapter 10 that the difference between polynomial time and higher growth rates in running time is really the divide between what we can solve by computer and what is in practice not solvable.

**Theorem 8.10**: The time take by the one-tape TM $N$ of Theorem 8.9 to simulate $n$ Moves of the $k$-tape TM is $O(n^2)$.

**Proof**:

- After $n$ moves of $M$, the tape head markers can have separated by only no more than $2n$ cells.
- Assume $N$ first locates the leftmost marker. 
- It has to move no more than $2n$ cells right, to find all the markers.
- For $k$ markers, it needs no more than $2k$ steps to wipe out old markers and write new markers.
- $N$ may make excursions leftward in this process, but no more than $2n$ steps.
- Therefore, it takes no more than $4n+2k$ steps to simulate a single step of $M$.
- It takes $O(n^2)$ steps to simulate $n$ steps of $M$.

---

### Nondeterministic Turing Machines

- A **nondeterministic Turing Machine** (NTM) differs from the TM only at the transition function.
- Similar to nondeterministic versions of other automata, the transition function becomes $\delta: Q\times \Gamma \to 2^{Q\times \Gamma \times D}$.
- The NTM can choose at each step any subset of triples to be the next move.
- The strings accepted by an NTM is those that there exists some sequence of choices of moves that eventually accepts the string.

---

**Equivalence of NTM and DTM**

- The NTM's accept no languages not accepted by a TM (or DTM if we need to emphasize it is deterministic).

**Theorem 8.11**: If $M_N$ is a NTM, then there is a DTM $M_D$ such that $L(M_N)=L(M_D)$.

**Proof**:

- The overall strategy is to let $M_D$ to explores all IDs that $M_N$ can reach.
- $M_D$ will have two tapes.
  - One tape holds a queue of ID's of $M_N$, among which there is an ID marked as the "current" ID pointed by the tape head. The successors of current ID are to be discovered.
  - Another tape will be used as a buffer.
- To process the current ID:
  - $M_D$ examines state and tape symbol. The knowledge of what moves $M_N$ will make is built into $M_D$. If there is any accepting state, $M_D$ discovers and accepts.
  - If there $M_D$ discovers no accepting state, assuming there are $k$ possible moves, $M_D$ will copy the current ID on the second tape; make $k$ copies of the that ID at end of the first tape; and modify each of the $k$ copies according different one of $k$ choices of move.
  - $M_D$ will mark the next ID as current ID and repeat this process.
- The time complexity of $M_D$ will grow exponentially.
  - Suppose $m$ is the maximum number of choices of $M_N$.
  - After $n$ moves of $M_N$, it can reach up to $1+m+m^2+\cdots+m^n \leq nm^n$ IDs.
  - All IDs that can reached by $M_N$ will be discovered by $M_D$, so $L(M_N)=L(M_D)$.













