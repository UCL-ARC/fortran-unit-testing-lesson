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

:::::::::::::::::::::::::::::::::::::::::::: callout

### Checking we haven't broken anything

To ensure we don't break anything during our refactoring we need to have some way to test our code.
Since we don't have any automated tests in place we will need to do this manually. Firstly, let's
generate a starting state which we know to be correct.

```sh
cd episodes/7-refactoring-fortran/challenge
cmake -B build
cmake --build build
./build/game-of-life ../models/model-1.dat > initial-state.out
```

Then, whenever we make a change, we can test if the code still works as expected

```sh
cmake --build build
./build/game-of-life ../models/model-1.dat > new-state.out
diff initial-state.out new-state.out
```

If there are no differences, we can assume we haven't broken anything.

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
--- a/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
+++ b/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
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

### 2. Change of variable name

**Smell**: Code needs a comment to explain what it is for.

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### Before:

```f90
a = a + b*dt
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### After:

```f90
velocity = velocity + acceleration * dt
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge

Update any poorly named variables in John's code to have clear names
which make it clear what they are.

:::::::::::::::::::::::: solution 

This can be achieved with the following diff

```diff
--- a/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
+++ b/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
@@ -11,7 +11,7 @@ program game_of_life
     !! Board args
     integer, parameter :: max_generations = 100, max_nrows = 100, max_ncols = 100
     integer :: nrow, ncol
-    integer :: i, generation_number
+    integer :: row, generation_number
     integer, dimension(:,:), allocatable :: current_board, new_board
 
     !! Animation args
@@ -21,7 +21,7 @@ program game_of_life
 
     !! CLI args
     integer                       :: argl
-    character(len=:), allocatable :: cli_arg_temp_store, input_fname
+    character(len=:), allocatable :: cli_arg_temp_store, input_filename
 
     !! File IO args
     character(len=80) :: text_to_discard
@@ -31,8 +31,8 @@ program game_of_life
     ! Get current_board file path from command line
     if (command_argument_count() == 1) then
         call get_command_argument(1, length=argl)
-        allocate(character(argl) :: input_fname)
-        call get_command_argument(1, input_fname)
+        allocate(character(argl) :: input_filename)
+        call get_command_argument(1, input_filename)
     else
         write(*,'(A)') "Error: Invalid input"
         call get_command_argument(0, length=argl)
@@ -45,12 +45,12 @@ program game_of_life
 
     ! Open input file
     open(unit=input_file_io,   &
-         file=input_fname, &
+         file=input_filename, &
          status='old',  &
          IOSTAT=iostat)
 
     if( iostat /= 0) then
-        write(*,'(a)') ' *** Error when opening '//input_fname
+        write(*,'(a)') ' *** Error when opening '//input_filename
         stop 1
     end if
 
@@ -75,8 +75,8 @@ program game_of_life
 
     read(input_file_io,'(a)') text_to_discard ! Skip next line
     ! Populate the boards starting state
-    do i = 1, nrow
-        read(input_file_io,*) current_board(i, :)
+    do row = 1, nrow
+        read(input_file_io,*) current_board(row, :)
     end do
 
     close(input_file_io)
@@ -114,17 +114,17 @@ contains
 
     !> Evolve the board into the state of the next iteration
     subroutine run_next_iteration()
-        integer :: i, j, sum
+        integer :: row, col, sum
         character(nrow) :: output
 
         ! Clear the terminal screen
         call system("clear")
 
         ! Draw the current board
-        do i=1, nrow
+        do row=1, nrow
             output = ""
-            do j=1, ncol
-                if (current_board(i,j) == 1) then
+            do col=1, ncol
+                if (current_board(row,col) == 1) then
                     output = trim(output)//"#"
                 else
                     output = trim(output)//"."
@@ -134,34 +134,34 @@ contains
         enddo
 
         ! Calculate the new board
-        do i=2, nrow-1
-            do j=2, ncol-1
+        do row=2, nrow-1
+            do col=2, ncol-1
                 sum = 0
-                sum = current_board(i, j-1)   &
-                    + current_board(i+1, j-1) &
-                    + current_board(i+1, j)   &
-                    + current_board(i+1, j+1) &
-                    + current_board(i, j+1)   &
-                    + current_board(i-1, j+1) &
-                    + current_board(i-1, j)   &
-                    + current_board(i-1, j-1)
-                if(current_board(i,j)==1 .and. sum<=1) then
-                    new_board(i,j) = 0
-                elseif(current_board(i,j)==1 .and. sum<=3) then
-                    new_board(i,j) = 1
-                elseif(current_board(i,j)==1 .and. sum>=4)then
-                    new_board(i,j) = 0
-                elseif(current_board(i,j)==0 .and. sum==3)then
-                    new_board(i,j) = 1
+                sum = current_board(row, col-1)   &
+                    + current_board(row+1, col-1) &
+                    + current_board(row+1, col)   &
+                    + current_board(row+1, col+1) &
+                    + current_board(row, col+1)   &
+                    + current_board(row-1, col+1) &
+                    + current_board(row-1, col)   &
+                    + current_board(row-1, col-1)
+                if(current_board(row,col)==1 .and. sum<=1) then
+                    new_board(row,col) = 0
+                elseif(current_board(row,col)==1 .and. sum<=3) then
+                    new_board(row,col) = 1
+                elseif(current_board(row,col)==1 .and. sum>=4)then
+                    new_board(row,col) = 0
+                elseif(current_board(row,col)==0 .and. sum==3)then
+                    new_board(row,col) = 1
                 endif
             enddo
         enddo
 
         ! Check for steady state
         steady_state = .true.
-        do i=1, nrow
-            do j=1, ncol
-                if (.not. current_board(i, j) == new_board(i, j)) then
+        do row=1, nrow
+            do col=1, ncol
+                if (.not. current_board(row, col) == new_board(row, col)) then
                     steady_state = .false.
                     exit
                 end if
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

### 3. Wrap program functionality in procedures

**Smell**: Logic is repeated outside a procedure.

**Smell**: Loops appear outside a procedure.

**Smell**: Lots of inline comments requited to explain what is happening in the main program.


:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### Before 

```f90
program my_matrix_prog
    use process_marices_mod, only : process_matrices
    implicit none

    character(len=200) :: temp_string
    character(:), allocatable :: filename


    print *, 'Enter input filename:'
    read (*,*) temp_string
    filename = trim(temp_string)

    call process_matrices(filename)

end program my_matrix_prog
```

::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler

#### After

```f90
program my_matrix_prog
    use process_marices_mod, only : process_matrices
    implicit none

    character(:), allocatable :: filename

    call read_filename(filename)
    call process_matrices(filename)

contains

    subroutine read_filename(filename)
        character(:), allocatable, intent(out) :: filename

        character(len=200) :: temp_string

        print *, 'Enter input filename:'
        read (*,*) temp_string

        filename = trim(temp_string)
    end subroutine read_filename

end program my_matrix_prog
```

::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

#### Challenge

Update John's code to reduce the responsibilities of any procedures to one

:::::::::::::::::::::::: solution 

This can be achieved with the following diff

```diff
--- a/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
+++ b/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
@@ -10,24 +10,17 @@ program game_of_life
 
     !! Board args
     integer, parameter :: max_generations = 100, max_nrows = 100, max_ncols = 100
-    integer :: nrow, ncol
-    integer :: row, generation_number
+    integer :: nrow, ncol, generation_number
     integer, dimension(:,:), allocatable :: current_board, new_board
-
-    !! Animation args
-    integer, dimension(8) :: date_time_values
-    integer :: mod_ms_step, ms_per_step = 250
     logical :: steady_state = .false.
 
+    !> Whether to animate the board
+    logical, parameter :: animate = .true.
+
     !! CLI args
     integer                       :: argl
     character(len=:), allocatable :: cli_arg_temp_store, input_filename
 
-    !! File IO args
-    character(len=80) :: text_to_discard
-    integer :: input_file_io
-    integer :: iostat
-
     ! Get current_board file path from command line
     if (command_argument_count() == 1) then
         call get_command_argument(1, length=argl)
@@ -43,77 +36,116 @@ program game_of_life
         stop
     end if
 
-    ! Open input file
-    open(unit=input_file_io,   &
-         file=input_filename, &
-         status='old',  &
-         IOSTAT=iostat)
-
-    if( iostat /= 0) then
-        write(*,'(a)') ' *** Error when opening '//input_filename
-        stop 1
-    end if
-
-    ! Read in current_board from file
-    read(input_file_io,'(a)') text_to_discard ! Skip first line
-    read(input_file_io,*) nrow, ncol
+    call read_model_from_file()
 
-    ! Verify the number of rows read from the file
-    if (nrow < 1 .or. nrow > max_nrows) then
-        write (*,'(a,i6,a,i6)') "nrow must be a positive integer less than ", max_nrows," found ", nrow
-        stop 1
-    end if
+    call find_steady_state()
 
-    ! Verify the number of columns read from the file
-    if (ncol < 1 .or. ncol > max_ncols) then
-        write (*,'(a,i6,a,i6)') "ncol must be a positive integer less than ", max_ncols," found ", ncol
-        stop 1
+    if (steady_state) then
+        write(*,'(a,i6,a)') "Reached steady after ", generation_number, " generations"
+    else
+        write(*,'(a,i6,a)') "Did NOT Reach steady after ", generation_number, " generations"
     end if
 
-    allocate(current_board(nrow, ncol))
-    allocate(new_board(nrow, ncol))
-
-    read(input_file_io,'(a)') text_to_discard ! Skip next line
-    ! Populate the boards starting state
-    do row = 1, nrow
-        read(input_file_io,*) current_board(row, :)
-    end do
-
-    close(input_file_io)
+    deallocate(current_board)
+    deallocate(new_board)
 
-    new_board = 0
-    generation_number = 0
+contains
 
-    ! Clear the terminal screen
-    call system ("clear")
+    !> Populate the board from a provided file
+    subroutine read_model_from_file()
+        !> A flag to indicate if reading the file was successful
+        character(len=:), allocatable :: io_error_message
+
+        ! Board definition args
+        integer :: row
+
+        ! File IO args
+        integer :: input_file_io, iostat
+        character(len=80) :: text_to_discard
+
+        input_file_io = 1111
+
+        ! Open input file
+        open(unit=input_file_io,   &
+            file=input_filename, &
+            status='old',  &
+            IOSTAT=iostat)
+
+        if( iostat == 0) then
+            ! Read in board from file
+            read(input_file_io,'(a)') text_to_discard ! Skip first line
+            read(input_file_io,*) nrow, ncol
+
+            ! Verify the number of rows and columns read from the file
+            if (nrow < 1 .or. nrow > max_nrows) then
+                allocate(character(100) :: io_error_message)
+                write (io_error_message,'(a,i6,a,i6)') "nrow must be a positive integer less than ", max_nrows, " found ", nrow
+            elseif (ncol < 1 .or. ncol > max_ncols) then
+                allocate(character(100) :: io_error_message)
+                write (io_error_message,'(a,i6,a,i6)') "ncol must be a positive integer less than ", max_ncols, " found ", ncol
+            end if
+        else
+            allocate(character(100) :: io_error_message)
+            write(io_error_message,'(a)') ' *** Error when opening '//input_filename
+        endif
+
+        if (.not. allocated(io_error_message)) then
+
+            allocate(current_board(nrow, ncol))
+
+            read(input_file_io,'(a)') text_to_discard ! Skip next line
+            ! Populate the boards starting state
+            do row = 1, nrow
+                read(input_file_io,*) current_board(row, :)
+            end do
 
-    ! Iterate until we reach a steady state
-    do while(.not. steady_state .and. generation_number < max_generations)
-        ! Advance the simulation in the steps of the requested number of milliseconds
-        call date_and_time(VALUES=date_time_values)
-        mod_ms_step = mod(date_time_values(8), ms_per_step)
+        end if
 
-        if (mod_ms_step == 0) then
-            call evolve_board()
-            call check_for_steady_state()
-            current_board = new_board
-            call draw_board()
+        close(input_file_io)
 
-            generation_number = generation_number + 1
+        if (allocated(io_error_message)) then
+            write (*,*) io_error_message
+            deallocate(io_error_message)
+            stop
         end if
+    end subroutine read_model_from_file
 
-    end do
+    !> Find the steady state of the Game of Life board
+    subroutine find_steady_state()
 
-    if (steady_state) then
-        write(*,'(a,i6,a)') "Reached steady after ", generation_number, " generations"
-    else
-        write(*,'(a,i6,a)') "Did NOT Reach steady after ", generation_number, " generations"
-    end if
+        !! Animation args
+        integer, dimension(8) :: date_time_values
+        integer :: mod_ms_step
+        integer, parameter :: ms_per_step = 250
 
-    deallocate(current_board)
-    deallocate(new_board)
+        allocate(new_board(size(current_board,1), size(current_board, 2)))
+        new_board = 0
 
-contains
+        ! Clear the terminal screen
+        if (animate) call system ("clear")
+
+        ! Iterate until we reach a steady state
+        steady_state = .false.
+        generation_number = 0
+        mod_ms_step = 0
+        do while(.not. steady_state .and. generation_number < max_generations)
+            if (animate) then
+                ! Advance the simulation in the steps of the requested number of milliseconds
+                call date_and_time(VALUES=date_time_values)
+                mod_ms_step = mod(date_time_values(8), ms_per_step)
+            end if
+
+            if (mod_ms_step == 0) then
+                call evolve_board()
+                call check_for_steady_state()
+                current_board = new_board
+                if (animate) call draw_board()
+
+                generation_number = generation_number + 1
+            end if
+
+        end do
+    end subroutine find_steady_state
 
     !> Evolve the board into the state of the next iteration
     subroutine evolve_board()

```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

### 4. Break large procedures into smaller units

**Smell**: A function or subroutine no longer fits on a page in your editor.

**Smell**: Multiple dummy arguments are updated (i.e. multiple `intent(out)` arguments).

**Smell**: A line of code is deeply indented.

**Smell**: A piece of code interacts with the surrounding code through just a few variables.

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### Before:

```f90
module process_marices_mod
    implicit none
    real, allocatable :: A(:,:), B(:,:), C(:,:)

contains
    subroutine process_matrices(filename)
        character(len=*), intent(in) :: filename
        integer :: n, iostat, i, j, k
        integer :: unit
        real :: trace

        open(newunit=unit, file=filename, status='old', action='read', iostat=iostat)
        if (iostat /= 0) then
            print *, 'Error opening file: ', trim(filename)
            stop
        end if

        read(unit, *, iostat=iostat) n
        if (iostat /= 0) stop 'Error reading matrix size.'

        allocate(A(n,n), B(n,n))

        print *, 'Reading matrix A (', n, 'x', n, ')'
        do i = 1, n
            read(unit, *, iostat=iostat) (A(i,j), j=1,n)
            if (iostat /= 0) stop 'Error reading matrix A.'
        end do

        print *, 'Reading matrix B (', n, 'x', n, ')'
        do i = 1, n
            read(unit, *, iostat=iostat) (B(i,j), j=1,n)
            if (iostat /= 0) stop 'Error reading matrix B.'
        end do

        close(unit)

        C = 0.0
        do i = 1, n
            do j = 1, n
                do k = 1, n
                    C(i,j) = C(i,j) + A(i,k) * B(k,j)
                end do
            end do
        end do

        n = size(C, 1)
        trace = 0.0
        do i = 1, n
            trace = trace + C(i,i)
        end do

        print *, 'Trace of matrix C = ', trace
    end subroutine process_matrices
end module process_marices_mod
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### After:

```f90
module process_marices_mod
    implicit none
    real, allocatable :: A(:,:), B(:,:), C(:,:)

contains

    subroutine read_matrices_from_file(filename)
        character(len=*), intent(in) :: filename
        integer :: n, iostat, i, j
        integer :: unit

        open(newunit=unit, file=filename, status='old', action='read', iostat=iostat)
        if (iostat /= 0) then
            print *, 'Error opening file: ', trim(filename)
            stop
        end if

        read(unit, *, iostat=iostat) n
        if (iostat /= 0) stop 'Error reading matrix size.'

        allocate(A(n,n), B(n,n))

        print *, 'Reading matrix A (', n, 'x', n, ')'
        do i = 1, n
            read(unit, *, iostat=iostat) (A(i,j), j=1,n)
            if (iostat /= 0) stop 'Error reading matrix A.'
        end do

        print *, 'Reading matrix B (', n, 'x', n, ')'
        do i = 1, n
            read(unit, *, iostat=iostat) (B(i,j), j=1,n)
            if (iostat /= 0) stop 'Error reading matrix B.'
        end do

        close(unit)
    end subroutine read_matrices_from_file

    subroutine multiply_matrices()
        integer :: i, j, k, n
        n = size(A, 1)

        allocate(C(n,n))

        C = 0.0
        do i = 1, n
            do j = 1, n
                do k = 1, n
                    C(i,j) = C(i,j) + A(i,k) * B(k,j)
                end do
            end do
        end do
    end subroutine multiply_matrices

    subroutine display_trace()
        integer :: i, n
        real :: trace

        n = size(C, 1)
        trace = 0.0
        do i = 1, n
            trace = trace + C(i,i)
        end do

        print *, 'Trace of matrix C = ', trace
    end subroutine display_trace
end module process_marices_mod
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge

Update John's code to reduce the responsibilities of any procedures to one

:::::::::::::::::::::::: solution 

This can be achieved with the following diff

```diff
--- a/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
+++ b/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
@@ -94,7 +94,10 @@ program game_of_life
         mod_ms_step = mod(date_time_values(8), ms_per_step)
 
         if (mod_ms_step == 0) then
-            call run_next_iteration()
+            call evolve_board()
+            call check_for_steady_state()
+            current_board = new_board
+            call draw_board()
 
             generation_number = generation_number + 1
         end if
@@ -113,27 +116,9 @@ program game_of_life
 contains
 
     !> Evolve the board into the state of the next iteration
-    subroutine run_next_iteration()
+    subroutine evolve_board()
         integer :: row, col, sum
-        character(nrow) :: output
 
-        ! Clear the terminal screen
-        call system("clear")
-
-        ! Draw the current board
-        do row=1, nrow
-            output = ""
-            do col=1, ncol
-                if (current_board(row,col) == 1) then
-                    output = trim(output)//"#"
-                else
-                    output = trim(output)//"."
-                endif
-            enddo
-            print *, output
-        enddo
-
-        ! Calculate the new board
         do row=2, nrow-1
             do col=2, ncol-1
                 sum = 0
@@ -157,21 +142,43 @@ contains
             enddo
         enddo
 
-        ! Check for steady state
-        steady_state = .true.
+        return
+    end subroutine evolve_board
+
+    !> Check if we have reached steady state, i.e. current and new board match
+    subroutine check_for_steady_state()
+        integer :: row, col
+
         do row=1, nrow
             do col=1, ncol
                 if (.not. current_board(row, col) == new_board(row, col)) then
                     steady_state = .false.
-                    exit
+                    return
                 end if
             end do
-            if (.not. steady_state) exit
         end do
+        steady_state = .true.
+    end subroutine check_for_steady_state
 
-        current_board = new_board
+    !> Output the current board to the terminal
+    subroutine draw_board()
+        integer :: row, col
+        character(nrow) :: output
 
-        return
-    end subroutine run_next_iteration
+        ! Clear the terminal screen
+        call system("clear")
+
+        do row=1, nrow
+            output = ""
+            do col=1, ncol
+                if (current_board(row,col) == 1) then
+                    output = trim(output)//"#"
+                else
+                    output = trim(output)//"."
+                endif
+            enddo
+            print *, output
+        enddo
+    end subroutine draw_board
 
 end program game_of_life
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

### 5. Replace repeated code with a procedure

**Smell:** Fragments of repeated code appear.

:::::::::::::::::::::::::::::::::::::::::::: spoiler

#### Before

```f90
subroutine read_matrices_from_file(filename)
    character(len=*), intent(in) :: filename
    integer :: n, iostat, i, j
    integer :: unit

    open(newunit=unit, file=filename, status='old', action='read', iostat=iostat)
    if (iostat /= 0) then
        print *, 'Error opening file: ', trim(filename)
        stop
    end if

    read(unit, *, iostat=iostat) n
    if (iostat /= 0) stop 'Error reading matrix size.'

    allocate(A(n,n), B(n,n))

    print *, 'Reading matrix A (', n, 'x', n, ')'
    do i = 1, n
        read(unit, *, iostat=iostat) (A(i,j), j=1,n)
        if (iostat /= 0) stop 'Error reading matrix A.'
    end do

    print *, 'Reading matrix B (', n, 'x', n, ')'
    do i = 1, n
        read(unit, *, iostat=iostat) (B(i,j), j=1,n)
        if (iostat /= 0) stop 'Error reading matrix B.'
    end do

    close(unit)
end subroutine read_matrices_from_file
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler

#### After

```f90
subroutine read_matrices_from_file(filename)
    character(len=*), intent(in) :: filename
    integer :: n, iostat, i, j
    integer :: unit

    open(newunit=unit, file=filename, status='old', action='read', iostat=iostat)
    if (iostat /= 0) then
        print *, 'Error opening file: ', trim(filename)
        stop
    end if

    read(unit, *, iostat=iostat) n
    if (iostat /= 0) stop 'Error reading matrix size.'

    allocate(A(n,n), B(n,n))

    print *, 'Reading matrix A (', n, 'x', n, ')'
    call read_next_matrix_from_file(A, unit)

    print *, 'Reading matrix B (', n, 'x', n, ')'
    call read_next_matrix_from_file(B, unit)

    close(unit)
end subroutine read_matrices_from_file

subroutine read_next_matrix_from_file(matrix, unit)
    real, allocatable, intent(inout) :: matrix(:,:)
    integer, intent(in) :: unit

    integer :: i, j, iostat, n

    n = size(matrix, 1)

    do i = 1, n
        read(unit, *, iostat=iostat) (matrix(i,j), j=1,n)
        if (iostat /= 0) stop 'Error reading matrix.'
    end do
end subroutine read_next_matrix_from_file
```

::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: callout

John's code appears to not have any repeated code, so there's nothing to do
for this refactoring principle. If you've spotted some, well Done!

:::::::::::::::::::::::::::::::::::::::::::::


### 6. Replace global variables with procedure arguments

**Smell:** A global variable is assigned and then used inside a called function.

**Smell:** A variable is edited within a procedure in which it is not declared.

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### Before:

```f90
subroutine multiply_matrices()
    integer :: i, j, k, n
    n = size(A, 1)

    allocate(C(n,n))

    C = 0.0
    do i = 1, n
        do j = 1, n
            do k = 1, n
                C(i,j) = C(i,j) + A(i,k) * B(k,j)
            end do
        end do
    end do
end subroutine multiply_matrices
```

::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### After:

```f90
subroutine multiply_matrices(A, B, C)
    real, allocatable, intent(int) :: A(:,:), B(:,:)
    real, allocatable, intent(out) :: C(:,:)

    integer :: i, j, k, n
    n = size(A, 1)

    allocate(C(n,n))
    
    C = 0.0
    do i = 1, n
        do j = 1, n
            do k = 1, n
                C(i,j) = C(i,j) + A(i,k) * B(k,j)
            end do
        end do
    end do
end subroutine multiply_matrices
```

::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge

Update John's code to replace any global variables accessed within procedures
with dummy arguments.

:::::::::::::::::::::::: solution 

This can be achieved with the following diff

```diff
--- a/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
+++ b/episodes/7-refactoring-fortran/solution/src/game_of_life.f90
@@ -8,11 +8,15 @@ program game_of_life
 
     implicit none
 
-    logical, parameter :: animate = .true.
-    integer, dimension(:,:), allocatable :: starting_board
-    integer :: generation_number
+    !! Board args
+    integer, parameter :: max_generations = 100, max_nrows = 100, max_ncols = 100
+    integer :: nrow, ncol, generation_number
+    integer, dimension(:,:), allocatable :: current_board, new_board
     logical :: steady_state = .false.
 
+    !> Whether to animate the board
+    logical, parameter :: animate = .true.
+
     !! CLI args
     character(len=:), allocatable :: executable_name, input_filename
 
@@ -26,9 +30,9 @@ program game_of_life
         stop
     end if
 
-    call read_model_from_file(input_filename, starting_board)
+    call read_model_from_file()
 
-    call find_steady_state(steady_state, generation_number, starting_board, animate)
+    call find_steady_state()
 
     if (steady_state) then
         write(*,'(a,i6,a)') "Reached steady after ", generation_number, " generations"
@@ -39,7 +43,7 @@ program game_of_life
 contains
 
     !> Read a cli arg at a given index and return it as a string (character array)
-    subroutine read_cli_arg(arg_index, arg)
+    recursive subroutine read_cli_arg(arg_index, arg)
         !> The index of the cli arg to try and read
         integer, intent(in) :: arg_index
         !> The string into which to store the cli arg
@@ -55,16 +59,12 @@ contains
     end subroutine read_cli_arg
 
     !> Populate the board from a provided file
-    subroutine read_model_from_file(input_filename, board)
-        character(len=:), allocatable, intent(in) :: input_filename
-        integer, dimension(:,:), allocatable, intent(out) :: board
-
+    subroutine read_model_from_file()
         !> A flag to indicate if reading the file was successful
         character(len=:), allocatable :: io_error_message
 
         ! Board definition args
-        integer :: row, nrow, ncol
-        integer, parameter :: max_nrows = 100, max_ncols = 100
+        integer :: row
 
         ! File IO args
         integer :: input_file_io, iostat
@@ -98,12 +98,12 @@ contains
 
         if (.not. allocated(io_error_message)) then
 
-            allocate(board(nrow, ncol))
+            allocate(current_board(nrow, ncol))
 
             read(input_file_io,'(a)') text_to_discard ! Skip next line
             ! Populate the boards starting state
             do row = 1, nrow
-                read(input_file_io,*) board(row, :)
+                read(input_file_io,*) current_board(row, :)
             end do
 
         end if
@@ -118,27 +118,14 @@ contains
     end subroutine read_model_from_file
 
     !> Find the steady state of the Game of Life board
-    subroutine find_steady_state(steady_state, generation_number, input_board, animate)
-        !> Whether the board has reached a steady state
-        logical, intent(out) :: steady_state
-        !> The number of generations that have been processed
-        integer, intent(out) :: generation_number
-        !> The starting state of the board
-        integer, dimension(:,:), allocatable, intent(in) :: input_board
-        !> Whether to animate the board
-        logical, intent(in) :: animate
-
-        integer, dimension(:,:), allocatable :: current_board, new_board
-        integer, parameter :: max_generations = 100
+    subroutine find_steady_state()
 
         !! Animation args
         integer, dimension(8) :: date_time_values
         integer :: mod_ms_step
         integer, parameter :: ms_per_step = 250
 
-        allocate(current_board(size(input_board,1), size(input_board, 2)))
-        allocate(new_board(size(input_board,1), size(input_board, 2)))
-        current_board = input_board
+        allocate(new_board(size(current_board,1), size(current_board, 2)))
         new_board = 0
 
         ! Clear the terminal screen
@@ -156,10 +143,10 @@ contains
             end if
 
             if (mod_ms_step == 0) then
-                call evolve_board(current_board, new_board)
-                call check_for_steady_state(steady_state, current_board, new_board)
+                call evolve_board()
+                call check_for_steady_state()
                 current_board = new_board
-                if (animate) call draw_board(current_board)
+                if (animate) call draw_board()
 
                 generation_number = generation_number + 1
             end if
@@ -168,16 +155,8 @@ contains
     end subroutine find_steady_state
 
     !> Evolve the board into the state of the next iteration
-    subroutine evolve_board(current_board, new_board)
-        !> The current state of the board
-        integer, dimension(:,:), allocatable, intent(in) :: current_board
-        !> The new state of the board
-        integer, dimension(:,:), allocatable, intent(inout) :: new_board
-
-        integer :: row, col, sum, nrow, ncol
-
-        nrow = size(current_board, 1)
-        ncol = size(current_board, 2)
+    subroutine evolve_board()
+        integer :: row, col, sum
 
         do row=2, nrow-1
             do col=2, ncol-1
@@ -206,18 +185,8 @@ contains
     end subroutine evolve_board
 
     !> Check if we have reached steady state, i.e. current and new board match
-    subroutine check_for_steady_state(steady_state, current_board, new_board)
-        !> Whether the board has reached a steady state
-        logical, intent(out) :: steady_state
-        !> The current state of the board
-        integer, dimension(:,:), allocatable, intent(in) :: current_board
-        !> The new state of the board
-        integer, dimension(:,:), allocatable, intent(inout) :: new_board
-
-        integer :: row, col, nrow, ncol
-
-        nrow = size(current_board, 1)
-        ncol = size(current_board, 2)
+    subroutine check_for_steady_state()
+        integer :: row, col
 
         do row=1, nrow
             do col=1, ncol
@@ -231,17 +200,9 @@ contains
     end subroutine check_for_steady_state
 
     !> Output the current board to the terminal
-    subroutine draw_board(board)
-        !> The current state of the board
-        integer, dimension(:,:), allocatable, intent(in) :: board
-
-        integer :: row, col, nrow, ncol
-        character(:), allocatable :: output
-
-        nrow = size(board, 1)
-        ncol = size(board, 2)
-
-        allocate(character(nrow) :: output)
+    subroutine draw_board()
+        integer :: row, col
+        character(nrow) :: output
 
         ! Clear the terminal screen
         call system("clear")
@@ -249,7 +210,7 @@ contains
         do row=1, nrow
             output = ""
             do col=1, ncol
-                if (board(row,col) == 1) then
+                if (current_board(row,col) == 1) then
                     output = trim(output)//"#"
                 else
                     output = trim(output)//"."
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

### 7. Separate code concepts into files or modules

**Smell:** You find it hard to locate a piece of code.

**Smell:** You get a lot of version control conflicts.

::::::::::::::::::::::::::::::::::::: spoiler 

#### Before

Using the example we have seen so far, we start with two files
`my_matrix_prog.f90` and `process_marices_mod.f90`.

```
|-- project/directory/
    |-- my_matrix_prog.f90
    |   |-- subroutine read_filename
    |-- process_marices_mod.f90
        |-- subroutine read_matrices_from_file
        |-- subroutine read_next_matrix_from_file
        |-- subroutine multiply_matrices
        |-- subroutine display_trace
```

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: spoiler 

#### After

If we split the procedures in these files across multiple modules which focus
on different tasks, we could end up with something like this.

```
|-- project/directory/
    |-- my_matrix_prog.f90
    |-- io.f90
    |   |-- subroutine read_filename
    |   |-- subroutine read_matrices_from_file
    |   |-- subroutine read_next_matrix_from_file
    |-- matrix_operations.f90
        |-- subroutine multiply_matrices
        |-- subroutine display_trace
```

> Note: there isn't one correct way to group these subroutines. For example, we
> could place `display_trace` in `io.f90`.

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge

Update John's code to separate code concepts into modules.

:::::::::::::::::::::::: solution

You should end up with a module structure. For example, like this...
```
|-- src/
    |-- main.f90
    |-- animation.f90
    |   |-- subroutine draw_board
    |-- cli.f90
    |   |-- subroutine read_cli_arg
    |-- game_of_life.f90
    |   |-- subroutine find_steady_state
    |   |-- subroutine evolve_board
    |   |-- subroutine check_for_steady_state
    |-- io.f90
        |-- subroutine read_model_from_file
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

## References

- Martin Gardner, 1970. [The fantastic combinations of John Conway’s new solitaire game “life” by Martin Gardner](https://web.stanford.edu/class/sts145/Library/life.pdf). Scientific American, 223, pp.120–123.
