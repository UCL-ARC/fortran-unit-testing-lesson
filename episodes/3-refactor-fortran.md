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

## The known refactorings

The next few sections will present some known refactorings.

We'll show before and after code, present any new coding techniques needed to do the refactoring, and describe [code smells](https://en.wikipedia.org/wiki/Code_smell): how you know you need to refactor.

### 1. Replace magic numbers with constants

**Smell**: Raw numbers appear in your code.

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### Before:

```f90
do i = 1, 100
    x = i * 3.141 / 100.0
    data(i) = sin(x)
end do
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### After:

```f90
do i = 1, resolution
    x = i * pi / real(resolution)
    data(i) = sin(x)
end do
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge

Replace all magic numbers in John's game of life code with constants.

:::::::::::::::::::::::: solution 

This can be achieved with the following diff

```diff
@@ -9,13 +9,14 @@ program game_of_life
     implicit none
 
     !! Board args
+    integer, parameter :: max_generations = 100, max_nrows = 100, max_ncols = 100
     integer :: nrow, ncol
     integer :: i, generation_number
     integer, dimension(:,:), allocatable :: current_board, new_board
 
     !! Animation args
     integer, dimension(8) :: date_time_values
-    integer :: mod_ms_step
+    integer :: mod_ms_step, ms_per_step = 250
     logical :: steady_state = .false.
 
     !! CLI args
@@ -58,14 +59,14 @@ program game_of_life
     read(input_file_io,*) nrow, ncol
 
     ! Verify the number of rows read from the file
-    if (nrow < 1 .or. nrow > 100) then
-        write (*,'(a,i6)') "nrow must be a positive integer less than 100 found ", nrow
+    if (nrow < 1 .or. nrow > max_nrows) then
+        write (*,'(a,i6,a,i6)') "nrow must be a positive integer less than ", max_nrows," found ", nrow
         stop 1
     end if
 
     ! Verify the number of columns read from the file
-    if (ncol < 1 .or. ncol > 100) then
-        write (*,'(a,i6)') "ncol must be a positive integer less than 100 found ", ncol
+    if (ncol < 1 .or. ncol > max_ncols) then
+        write (*,'(a,i6,a,i6)') "ncol must be a positive integer less than ", max_ncols," found ", ncol
         stop 1
     end if
 
@@ -87,10 +88,10 @@ program game_of_life
     call system ("clear")
 
     ! Iterate until we reach a steady state
-    do while(.not. steady_state .and. generation_number < 100)
+    do while(.not. steady_state .and. generation_number < max_generations)
         ! Advance the simulation in the steps of the requested number of milliseconds
         call date_and_time(VALUES=date_time_values)
-        mod_ms_step = mod(date_time_values(8), 250)
+        mod_ms_step = mod(date_time_values(8), ms_per_step)
 
         if (mod_ms_step == 0) then
             call run_next_iteration()
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

## References

- Martin Gardner, 1970. [The fantastic combinations of John Conway’s new solitaire game “life” by Martin Gardner](https://web.stanford.edu/class/sts145/Library/life.pdf). Scientific American, 223, pp.120–123.
