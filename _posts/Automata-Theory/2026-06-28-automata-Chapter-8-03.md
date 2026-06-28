---
layout: post
title: "8.2 The Turing Machine"
category: "Automata-Theory"
order: "803"
math: true
---

- The purpose of decidability theory is to provide a guidance to pragrammers about what they might or might not be able to accomplish through programming.
- Therefore, we need tools that allow us to prove everyday quesions decidable or not.
- However, the technology introduced in Section 8.1 does not translate easily to programs in unrelated domains.
  - For example, to determine whether a grammar is ambiguous.

- We will rebuild our theory of undecidability based on a very simple model of a computer, called Turing Machine.
- Using Turing Machine, we shall show in Section 9.4 that "Post's Correspondence Problem" is undecidable.
  - This problem makes it easy to show questions about grammars, such as ambiguity, to be undecidable.


---

### The Quest to Decide All Mathematical Questions

- The mathematician D. Hilbert asked at the turn of the 20th century, whether it was possible to find an algorithm for determining the truth of falsehood of any mathematical proposition.
- In particular, he asked if there was a way to determine any formula in the first-order predicate calculus, applied to intergers, was true.
  - The first-order predicate calculus of integers is sufficiently powerful to express statements like "this grammar is ambiguous" or "this program prints `hello world`".
  - The predicate calculus was not the only notion for "any possible computation".
  - Others include partial-recursive functions, and Turing Machines.
- All the serious proposals for a model of computation have the same power, i.e., they recognize the same languages.
- The **Church-Turing thesis** proposes the unprovable assumption that any general way to compute will allow us to compute only what what Turing machines or modern-day comptuers can compute.

---

### Notations for the Turing Machine

The Turing machine consists of

- A **finite control**, which can be in any of a finite set of states.
- A **tape**, divided into cells. Each cell can hold any one of a finite number of symbols.

The working procedure of a Turing machine is as follows.

- Initially, the **input**, which is a finite string of symbols from **input alphabet**, is placed on the tape. 

- All other tape cells, extending inifinitely to the left and right, hold a special symbol called **blank**. The blank is a **tape symbol** but not an input symbol.

- There is a **tape head** always positioned at one of the tape cells. The Turing machine is sad to be **scanning** that cell. Initially, the tape head is at the leftmost cell that holds the input.

- A move of the Turing machine is a funtion of the state and the tape symbol scanned. In one move, the machine will:

  - Change State. 

    > The next state optionally may be the same as the current state.

  - Write a tape symbol in the cell scanned. 

    > The written symbol replaces the existing symbol, and optionally may be the same.

  - Move the tape head left or right. 

    > We do not allow the head to remain stationary, and this does not constrain the ability of Turing Machine, because any sequence of moves with a stationary head could be condensed, along with the next tape-head move, into a single state change, a tape symbol, and a move left or right.

  ---


### Formal Definition of Turing Machines

  We describe a **Turing Machine** (TM) by the 7-tuple
$$
  M = (Q, \Sigma, \Gamma, \delta, q_0, B, F)
$$
  whose components have the following meanings:

  -  $Q$: The finite set of **states** of the finite control.
  - $\Sigma$: The finite set of **input symbols**.
  - $\Gamma$: The complete set of **tape symbols**; $\Sigma \subseteq \Gamma$.
  - $\delta: Q\times \Gamma \to Q \times \Gamma \times D$: The transition function, where $D$ is a direciton, either $L$ for left or $R$ for right.
  - $q_0$: The **start state**, a member of $Q$.
  - $B$: The **blank symbol**, $B \in \Gamma$ and $B \notin \Sigma$.
  - $F$: The set of **final** or **accepting states**, $F \subseteq Q$.

---

### Instantaneous Descriptions (IDs) for Turing Machines

- In order to describe formally what a TM does, we need notations for its configurations or **instantaneous descriptions**(IDs).
- Problem 1: A TM has an infinitely long tape, which is impossible to describe in principle.
  - After any finite number of moves, only a finite number of cells is visited.
  - There are infinite prefix and infinite suffix of cells that all hold blanks.
  - We only show the cells between the leftmost and rightmost nonblanks in an ID.
  - Special case: when the head is scanning a leading or trailing blank, some blanks should be included in the ID.
- Problem 2: To represent the finite control and tape-head position.
  - Embed the state in the tape, and place it immediately to the left of cell scanned.
  - States and tape symbols must have nothing in common, which can be ensured by simply renaming.

---

- We shall use the string $X_1X_2\cdots X_{i-1}qX_iX_{i+1}\cdots X_n$ to represent an ID in which

  - $q$ is the current state of TM.

  - The tape head os scanning the cell with $X_i$.

  - $X_1X_2\cdots X_n$ is the portion of tape between the leftmost and rightmost blank. For exception,  some prefix or suffix can be blank if $X_1$ or $X_n$ is being scanned. 

- We describe moves of a TM $M$ by the notation $\underset{M}{\vdash}$ , or just $\vdash$ when $M$ is understood.

- We use $\overset{\ast}{\vdash}$ to indicate zero, one or more moves of the TM $M$.

- Suppose $\delta(q, X_i)=(p, Y,L)$, then

  $$
  X_1X_2 \cdots X_{i-1}qX_i X_{i+1}\cdots X_n \vdash X_1X_2\cdots X_{i-2}pX_{i-1}YX_{i+1}\cdots X_n
  $$

  - If $i=1$, we have

    $$
    qX_1X_2\cdot X_n \vdash pBYX_2\cdots X_n
    $$

  - If $i = n$ and $Y = B$, we have

    $$
    X_1X_2\cdots X_{n-1}qX_n \vdash X_1X_2 \cdots X_{n-2}pX_{n-1}
    $$

- Suppose $\delta(q, X_i) = (p, Y, R)$, then

  $$
  X_1X_2\cdots X_{i-1}qX_iX_{i+1}\cdots X_n\vdash X_1X_2\cdots X_{i-1}YpX_{i+1}\cdots X_n
  $$

  - If $i = 1$ and $Y = B$, we have
    
    $$
    qX_1X_2\cdots X_N \vdash pX_2\cdots X_n
    $$

  - If $i = n$, we have
    
    $$
    X_1X_2\cdots X_{n-1}qX_n \vdash X_1X_2 \cdots X_{n-1}YpB
    $$
    

  ---

### Example 1 for Turing Machine

- Let us design a TM that accepts the language $\lbrace 0^n1^n \,\vert\,  n\geq 1 \rbrace$.
- We informally describe the behavior of the TM instead of giving every definition of states and transition function.
- Given a finite sequence of 0's and 1's on the tape, the TM will alternatively change a 0 to an $X$ and a 1 to a $Y$, until all 0's and 1's are matched.
  - It repeatedly changes a 0 to an $X$.
  - Then it moves to the right over whatever 0's and $Y$'s it sees, until it comes to a 1.
  - It changes the 1 to a $Y$.
  - Then it moves left over $Y$'s and 0's, until it finds an $X$.
  - Then it looks for a 0 immediately to the right, and if it finds one, repeats the process above.
- If the input is not  $0^\ast1^\ast$, then the TM will fail to have a next move and die.
- Only when the number of 0's and 1's are equal will the TM accept. 

---

### Example 2 for Turing Machine

- Turing's original view of his machine was a computer of integer-valued functions.
- Integers were represented by unary, and the machine computed by changing the length of the blocks of characters or constructing new characters.
- We show how a TM might compute **proper substraction** or **monus**
  - It is defined as $m\overset{\cdot}{-}n = \max(m-n, 0)$.
- This TM $M$ omits the accepting states, because it is not used to accept inputs.
- $M$ starts with a tape consisting of $0^m10^n$ surrounded by blanks.
- $M$ Halts with $0^{m\overset{\cdot}{-}n}$ on its tape, surrounded by blanks.

---

- $M$ repeatedly finds its leftmost remaining 0 and replaces it by a blank.
- It then searches right, looking for a 1.
- After finding a 1, it continues right, until it comes to a 0, and replace it by a 1.
- Then it returns left, looking for the leftmost 0, and repeat the process.

This repetition ends if either:

- All the $n$ 0's in $0^m10^n$ have been changed to 1's.
- The first m 0's have been changed to blanks.

---

### Definition of the Language of a Turing Machine

- Let $M = (Q, \Sigma, \Gamma, \delta, q_0, B, F)$ be a Turing Machine.

- $L(M)$ is the set of strings $w\in \Sigma^\ast$ such that $q_0w\overset{\ast}{\vdash}\alpha p \beta$ for some state $p\in F$ and any tape strings $\alpha, \beta$.

- The set of languages we can accept using a Turing machine is called the **recursively enumerable languages** or RE languages.

  > The term "recursive" as a synonym for "decidable" goes back to Mathematics as it existed prior to computers. Then, formalism based on recursion were commonly used as a notion of computation. In that sense, to say a problem was "recursive" had the positive sense of "it is sufficiently simple that it can be solved by a recursive function, and the function always finishes".
  >
  > The term "recursive enumerable" means that, there exists a function that could list all the members of a language in some order; that is, to enumerate them.  The languages that can have their members listed in some order are the same as the languages that are accepted by some TM, although that TM might run forever on inputs that it does not accept.

---

### Halting of Turing Machines

- A TM is said to **halt** if it enters a state $q$, scanning a tape symbol $X$, and there is no move available, i.e., $\delta(q, X)$ is undefined.

- If a TM accepts an input, we can always assume it halts.

  - We can make $\delta(q, X)$ undefined for all $q \in F$.
  - This will not change the language accepted.

- However, if a TM does not accept an input, it does not always halt.

  - Those languages with TMs that do halt, regardless of whether or not they accept, are called **recursive languages**.
  - Recursive languages are good models of "algorithms", because an algorithm should always ends or halts on any input.

  





