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
program wrap_logic_in_procedures
    use data_processing_mod, only : process_data
    implicit none

    integer :: n, i
    real, allocatable :: data(:)


    print *, 'Enter number of data points:'
    read *, n
    allocate(data(n))

    print *, 'Enter the data values:'
    do i = 1, n
        read *, data(i)
    end do

    call process_data(data)

end program wrap_logic_in_procedures
```

::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler

#### After

```f90 
program wrap_logic_in_procedures
    use data_processing_mod, only : process_data
    implicit none

    real, allocatable :: data(:)


    call read_data(data)

    call process_data(data)

contains
    subroutine read_data(data)
        implicit none
        real, allocatable, intent(out) :: data(:)
        integer :: i, n

        print *, 'Enter number of data points:'
        read *, n
        allocate(data(n))

        print *, 'Enter the data values:'
        do i = 1, n
            read *, data(i)
        end do
    end subroutine read_data
end program wrap_logic_in_procedures
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
module data_processing_mod
    implicit none

contains
    subroutine process_data(data)
        real, allocatable, intent(in) :: user_data(:)

        integer :: i, j, n
        real :: sum, avg, maxval, minval, temp

        n = size(user_data, 1)

        ! --- Compute statistics (mean, max, min, sum) ---
        sum = 0.0
        maxval = user_data(1)
        minval = user_data(1)
        do i = 1, n
            sum = sum + user_data(i)
            if (user_data(i) > maxval) maxval = user_data(i)
            if (user_data(i) < minval) minval = user_data(i)
        end do
        avg = sum / n

        ! --- Display statistics ---
        print *, 'Sum = ', sum
        print *, 'Average = ', avg
        print *, 'Maximum = ', maxval
        print *, 'Minimum = ', minval

        ! --- Sort the user_data ---
        do i = 1, n-1
            do j = 1, n-i
                if (user_data(j) > user_data(j+1)) then
                    temp = user_data(j)
                    user_data(j) = user_data(j+1)
                    user_data(j+1) = temp
                end if
            end do
        end do

        ! --- Compute and display differences between consecutive values ---
        print *, 'Differences between consecutive sorted values:'
        do i = 2, n
            print *, user_data(i) - user_data(i-1)
        end do

        ! --- Clean up ---
        deallocate(user_data)
    end subroutine process_data
end module data_processing_mod
```
::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::: spoiler
#### After:

```f90
module data_processing_mod
    implicit none

contains

    subroutine compute_statistics(data, sum, avg, maxval, minval)
        real, allocatable, intent(in) :: data(:)
        real, intent(out) :: sum, avg, maxval, minval
        integer :: i, n

        n = size(data, 1)
        sum = 0.0
        maxval = data(1)
        minval = data(1)

        do i = 1, n
            sum = sum + data(i)
            if (data(i) > maxval) maxval = data(i)
            if (data(i) < minval) minval = data(i)
        end do

        avg = sum / n
    end subroutine compute_statistics

    subroutine display_statistics(sum, avg, maxval, minval)
        real, intent(in) :: sum, avg, maxval, minval

        print *, '--- Statistics ---'
        print *, 'Sum      = ', sum
        print *, 'Average  = ', avg
        print *, 'Maximum  = ', maxval
        print *, 'Minimum  = ', minval
    end subroutine display_statistics

    subroutine sort_data(arr)
        implicit none

        real, allocatable, intent(inout) :: arr(:)
        integer :: i, j, n
        real :: temp

        n = size(arr, 1)

        do i = 1, n - 1
            do j = 1, n - i
                if (arr(j) > arr(j+1)) then
                    temp = arr(j)
                    arr(j) = arr(j+1)
                    arr(j+1) = temp
                end if
            end do
        end do
    end subroutine sort_data

    subroutine display_differences(data, n)
        real, allocatable, intent(in) :: data(:)
        integer :: i, n

        n = size(arr, 1)

        print *, '--- Differences between consecutive sorted values ---'
        do i = 2, n
            print *, data(i) - data(i-1)
        end do
    end subroutine display_differences
end module data_processing_mod
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

## References

- Martin Gardner, 1970. [The fantastic combinations of John Conway’s new solitaire game “life” by Martin Gardner](https://web.stanford.edu/class/sts145/Library/life.pdf). Scientific American, 223, pp.120–123.
