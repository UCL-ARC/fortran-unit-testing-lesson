---
title: "Refactoring Fortran"
teaching: 
exercises: 
---

:::::::::::::::::::::::::::::::::::::: questions 

- What does good Fortran code look like?
- How do I refactor Fortran code to follow best practices?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Be able to spot bad practice within Fortran code.
- Understand why following best practice make Fortran more testable.

::::::::::::::::::::::::::::::::::::::::::::::::

Within Fortran projects, it is common to find many instances of bad practice which makes it difficult,
if not impossible to implement unit tests. Therefore, in many cases, the first step to writing unit tests
for a Fortran project is to refactor some section of the code into a more testable state which follows
best practice. Examples of what we mean by "bad practice" would be not limited to but could include...

- Using global variables.
- Large, multi-purpose procedures.
- Undocumented variables, procedures, modules and programs.

To demonstrate the benefits of refactoring Fortran and how it can be done, we're going to help John to
improve his Fortran implementation of the game of life. A copy of John's code can be found in the
exercises repo at [path/to/code]().

:::::::::::::::::::::::::::::::::::::::::::: spoiler
### Conway's Game of Life 

Conway's Game of life is a cellular automaton devised by the British mathematician John Horton Conway in 1970 (Gardner, 1970).

The universe of the Game of Life is an infinite, two-dimensional orthogonal grid of square cells, each of which is in one of two possible states, live or dead (or populated and unpopulated, respectively). Every cell interacts with its eight neighbours, which are the cells that are horizontally, vertically, or diagonally adjacent. At each step in time, the following transitions occur:

1. Any live cell with fewer than two live neighbours dies, as if by underpopulation.
2. Any live cell with two or three live neighbours lives on to the next generation.
3. Any live cell with more than three live neighbours dies, as if by overpopulation.
4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

See the [Wikipedia article](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) for more details.

::::::::::::::::::::::::::::::::::::::::::::::::::::

