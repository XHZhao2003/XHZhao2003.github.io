---
layout: post
title: "8.6 Turing Machines and Computers"
category: "automata-theory"
order: "807"
math: true
---

- We will show that Turing machines and the common sort of computer that we use daily, accepts the same languages -- the RE languages.
- The notion of "a common computer" is not well defined mathematically, and the arguments in the section are necessarily informal.

---

### Simulating a Turing Machine by Computer

- Given a Turing machine $M$, we can write a program that acts like $M$.
  - States will be encoded as strings.
  - The program uses a transition table to simulate transitions.
  - Tape symbols are also encoded as strings.
- The problem is, the length of the tape is infinite, but the memory of computers is finite.
  - If storage devices are finite, then computers will only be a finite automaton.
  - However, common computers have swappable storage devices.
  - For example, hard disks can be removed and replaced by empty but otherwise identical disks.
  - We can have a human operator to swap disks whenever the TM machine wants to move to positions on tapes that are mapped to other disks.
  - All involved disks should be arranged in order, to simulate the linear tape.
  - We may assume there are infinite disks that the human operator can buy.

---

### Simulating a Computer by a Turing Machine

- We will answer two questions:
  - Are there any things a common computer can do while a Turing machine cannot?
  - Are there any things a common computer can do much faster than a Turing machine.
- The answer will be: a TM can simulate a TM sufficiently fast that "only" a polynomial separates the running time of the computer and TM on given problems.

---

We first give a realistic but informal model of how a typical computer operates.

- The storage of a computer consists of indefinitely long sequence of **words**, each with an **address**.
  - Address will be consecutive numbers 0, 1, 2 and so on.
  - Words in real computers will be of 32 or 64 bits long, but we do not limit the length of a word.
- The program is stored in some of the words of memory.
  - Words each represent a simple instruction.
  - Indirect addressing is permitted, so one instruction could use the content of another word as the the address of the word to which the operation is applied.
- Each instruction involves a limited number of words, and changes the value of at most one word.
- The operations that instruction may perform are not restricted.
  - The relative speed of different operations on different words will not be taken into account.
  - It does not matter if we are only interested the language-recognizing abilities.
  - Even if we are interested in the running time, the relative speeds differences are "only" a constant factor and not important.

---

The TM simulating a computer will have several tapes.

- The first tape represents the entire memory of the computer.

  - The words are encoded with their addresses in the following way

    $$\$ 0 \ast w_0 \#1\ast w_1 \#10\ast w_2 \# 11 \ast w_3 \cdots$$

  - The $\ast$'s are end-markers of words.
  - The \$'s are end-marker of the memory.
  - The \#'s are used to separate the words and their addresses.
  - Addresses are encoded in binary.
  - Registers, caches, main memory and disks of a real computer can be mapped properly onto this tape.

- The second tape is the instruction counter.

  - It holds one integer in binary, which represents one of memory locations on tape 1.
  - The value store in this location will be interpreted as the next instruction.

- The third tape holds a memory address or the contents of that address after the address has been located on tape 1.

  - This is for the addressing process.
  - The desired address is copied onto tape 3.
  - Then, the TM compares it with addresses on tape 1 until a match is found.
  - The contents of this address is then copied onto the third tape.

- The TM may use other tapes for computer's input file or for scratch.

---

The TM will simulate the instruction cycle of the computer as follows:

- Search the first tape for the instruction that matches the instruction number on tape 2.
- Examine the found instruction value. 
  - We assume the first few bits represent the action to be taken (e.g. copy, add, branch) and the remaining bits represents addresses that are involved in the action.
- Perform addressing. 
  - Copy addresses onto tape 3, search tape 1, and copy values onto tape 3.
- Execute the instruction. We cannot go into all possible instructions, but we will do with the basic and important ones:
  - Copy the value to some other address.
    - Go back to tape 1 to locate that address.
    - Copy the value into that address.
    - If more space is needed, or the new value uses less space, we will change the available space by shifting over.
      - Copy the entire nonblank content of tape 1 onto scratch tape.
      - Write the new value onto tape 1
      - Recopy the scratch tape back to tape 1, immediately right to the new value.
  - Add the value to the value of some other address.
    - Go back to tape 1 to locate that address.
    - Perform a binary addition of the value of that address and the value on tape 3.
    - If the result needs more space, shift contents on tape 1 to create enough space.
  - Take a value as an address and jump to it.
    - Simply copy tape 3 to tape 2.
- If the performed instruction is not a jump, add 1 to tape 2 and begin the instruction cycle.

---

### The Running Times of Computers and Turing Machines

**Theorem 8.17**: The Turing machine describe above can simulate $n$ steps, in $O(n^3)$ of its own steps, of a computer that

- has only instructions that increase the word length by at most 1
- has only instructions that a multitape TM can perform on words of length $k$ in $O(k^2)$ steps or less.

---

- The problem we want to handle by restriction 1 is the multiplication instruction.
  - Multiply a word by itself will double its length.
  - This will cause the length of words to grow exponentially.
  - In reality, this case will cause overflow and values are truncated.
  - We only allow words to increase its length by 1 at a time.
- The second restriction deals with all possible computer instructions that we do not cover.

---

**Proof**:

- The first tape starts with only the program of constant length $c$, occupying $d$ words.
- The occupied storage on tape 1 after $n$ steps is $O(n^2)$.
  - After $n$ steps, the computer cannot have created any words longer than $c+n$.
  - And the number of words are no more then $d+n$.
- Therefore, the execution of each instruction takes $O(n^2)$ steps.
  - Addressing, shifting over all needs only $O(n^2)$ time.
- Therefore, simulating $n$ steps needs $O(n^3)$ steps.

---

- Finally, we note that the simulation above use a multitape TM.
- We know that a basic TM can simulate $n$ steps of a multitape TM in $O(n^2)$ steps.
- Therefore, we can simulate a computer with a basic TM also in polynomial time.

**Theorem 8.18**: A computer of the type describe in Theorem 8.17 can be simulated for $n$ steps by one-tape Turing machine using at most $O(n^6)$ steps of the Turing machine.



















