- What format is data saved in? (in memory and on disk)
- When does it move from memory to disk?
- Why can there only be one primary key per table?
- How does rolling back a transaction work?
- How are indexes formatted?
- When and how does a full table scan happen?
- What format is a prepared statement saved in?

In short, how does a database **work**?

I’m building a clone of [sqlite](https://www.sqlite.org/arch.html) from scratch in C in order to understand, and I’m going to document my process as I go.

# Table of Contents

- [Part 1 - Introduction and Setting up the REPL](https://cstack.github.io/db_tutorial/parts/part1.html)
- [Part 2 - World’s Simplest SQL Compiler and Virtual Machine](https://cstack.github.io/db_tutorial/parts/part2.html)
- [Part 3 - An In-Memory, Append-Only, Single-Table Database](https://cstack.github.io/db_tutorial/parts/part3.html)
- [Part 4 - Our First Tests (and Bugs)](https://cstack.github.io/db_tutorial/parts/part4.html)
- [Part 5 - Persistence to Disk](https://cstack.github.io/db_tutorial/parts/part5.html)
- [Part 6 - The Cursor Abstraction](https://cstack.github.io/db_tutorial/parts/part6.html)
- [Part 7 - Introduction to the B-Tree](https://cstack.github.io/db_tutorial/parts/part7.html)
- [Part 8 - B-Tree Leaf Node Format](https://cstack.github.io/db_tutorial/parts/part8.html)
- [Part 9 - Binary Search and Duplicate Keys](https://cstack.github.io/db_tutorial/parts/part9.html)
- [Part 10 - Splitting a Leaf Node](https://cstack.github.io/db_tutorial/parts/part10.html)
- [Part 11 - Recursively Searching the B-Tree](https://cstack.github.io/db_tutorial/parts/part11.html)
- [Part 12 - Scanning a Multi-Level B-Tree](https://cstack.github.io/db_tutorial/parts/part12.html)
- [Part 13 - Updating Parent Node After a Split](https://cstack.github.io/db_tutorial/parts/part13.html)
- [Part 14 - Splitting Internal Nodes](https://cstack.github.io/db_tutorial/parts/part14.html)
- [Part 15 - Where to go next](https://cstack.github.io/db_tutorial/parts/part15.html)

> “What I cannot create, I do not understand.” – [Richard Feynman](https://en.m.wikiquote.org/wiki/Richard_Feynman)