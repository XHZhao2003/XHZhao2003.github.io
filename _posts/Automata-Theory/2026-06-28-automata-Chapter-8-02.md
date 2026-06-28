---
layout: post
title: "8.1 Problems that Computers Cannot Solve"
category: "automata-theory"
order: "802"
math: true
---

- In this section, we provide an informal proof that one specific problem cannot be solved by computers.
- This problem is to determine, whether the first thing a C program prints is `hello world`.
- The difficulty of this problem is that, the program may take an unimaginably long time before making any output.
- Not knowing when, if ever, something will occur, is the ultimate cause of our inability to tell what a program does.

---

### An Example

- We all know this program exactly prints `hello world`.

  ```c
  main()
  {
    printf("hello, world\n");
  }
  ```

- However, there are other programs that also print `hello world`.

- A program may first try to solve the integer equation below where $n>2$, and if it finds a solution, then it prints `hello world` and exits.

	$$
  x^n + y^n=z^n
  $$


- The program will try to solve this equation just by simple searching.
- It takes human more than 300 years to prove that the equation has no integer solution, i.e., the simple program will not print `hello world`.
- We may let the program solve any problem that mathematicians have not yet been able to resolve, and print `hello world` if it succeeds.

---

### A Hypothetical Tester

- The proof of impossibility of making the "hello-world tester" is by contradiction.
- We assume there is a program $H$ to be the tester.
  - $H$ takes as input a program $P$ and input $I$;
  - and tells whether $P$ with input $I$ prints `hello world`.
  - That is, $H$ prints exactly `yes` or `no` for each circumstances.
- If $H$ really exists, then the problem of "hello-world testing" is said to be **decidable**. 
- Otherwise, the probelm is said to be **undecidable**.
- The formal definition of decidability will be introduced in Chapter 9.

---

### Proof by Contradiction

- We construct another program $H_1$ based on $H$.
  - $H_1$ also takes as input $P$ and $I$. 
  - $H_1$ first simulates $H$, and print its own results according to the result of $H$.
  - If $H$ prints `no`, then $H_1$ prints `hello world`.
  - If $H$ prints `yes`, then $H_1$ prints `yes`.

- Then we construct program $H_2$ based on $H_1$.
  - $H_2$ only takes as input a program $P$.
  - $H_2$ just simulates $H_1$ by using $P$ and $P$ as input for $H_1$, and prints whatever $H_1$ prints.
  - That is to ask, what $H_1$ will do on its own code.
- The contradiction will arise when we consider what $H_2$ will do on input $H_2$.
- On the one hand, $H_2$ cannot print `yes`.
  - If so, it means that $H_1$ says `yes` on $(H_2, H_2)$.
  - It further means $H$ will first print `hello world` on input $H_2$.
  - However, we assume it print `yes`.
- On the other hand, $H_2$ cannot print `hello world`.
  - If so, it means that $H_1$ says `hello world` on $(H_2, H_2)$.
  - It further means $H$ will not first print `hello world` on input $H_2$.
  - However, we assume it prints `hello world`.
- Therefore, we conclude that $H, H_1, H_2$ cannot exist.

> Note: The transformation above is essentially the insight that allowed Alan Turing to prove his undecidability result about Turing Machines.

### Why Undecidable Problems Must Exist

- Although it is tricky to prove the problem mentioned above is undecidable, it is quite easy to show that why almost all problems must be undecidable.
- Recall that  a "problem" is really membership of a string in a language.
- Let our alphabet be the ASCII alphabet.
- Then there will be only countable number of strings.
  - We can order them by length. For the same length, we order lexicographically.
  - That is, we can assign an integer for each string.
  - Any program can be considered as a string.
- However, there will be uncountable number of languages.
  - The set of languages is the power set of strings.
- As a result, we know there are infinitely fewer programs than there are problems.
- If we pick a language randomly, almost certainly is would be undecidable.
- However, we are only interested in those fairly simple, well-structured ones, which are often decidable.

---

### Reducing One Problem to Another

- If we want to know the decidability of another problem, we may also try to prove by contradiction that it is undecidable using similar techniques.
- However, once we have one problem that we know is undeciable, we can use this fact to show undecidability of other problems.
- Suppose $P_1$ is know to be undecidable, and $P_2$ is a new problem that we would like to prove is undecidable as well.
- We may invent a construction that converts instances of $P_1$ to instances of $P_2$ that have the same answer.
  - That is, any string in language $P_1$ is converted to some string in language $P_2$.
  - And ant string not in $P_1$ is converted to some string not in $P_2$.
- Once we have this construction, we can solve $P_1$ as long as we can solve $P_2$.
  - Given any instance of $P_1$, first convert it to an instance of $P_2$.
  - Test the instance of $P_2$ , and give the same answer for instance of $P_1$.
- We call this techique the **reduction** of $P_1$ to $P_2$.

---

### Example for Reduction

- Consider this problem. Does program $Q$ , given input $y$, ever call function `foo`?
- We construct a reduction of the hello-world problem $P_1$ to this problem $P_2$.
  -  If $Q$ has a function called `foo`, rename it and all its calls. Essentially the new program $Q_1$ does exactly what $Q$ does.
  - Add to $Q_1$ a function `foo` to get program $Q_2$. This function does nothing and is not called.
  - Modify $Q_2$ to remember the first 12 characters printed. And whenever it executes any output statement, it checks the characters already printed. If they are exactly `hello world`, call function `foo`.
- The resulting program $Q_3$ calls `foo` if and only if $Q$ first print `hello world`.











