---
title: "Fortran Unit Test Syntax"
teaching:
exercises:
---

:::::::::::::::::::::::::::::::::::::: questions 

- What is the syntax of writing a unit test in Fortran?
- How do I build my tests with my existing build system?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Able to write a unit test for a Fortran procedure with test-drive, veggies and/or pFUnit.
    - Assessment: Given a procedure, ask learner to write a unit test. Provide all the boilerplate needed

- Understand the similarities between each framework and where they differ.
    - Assessment: Assign the labels for shared unit test anatomy to sections of tests written with each framework.
    - Assessment: Provide a test in one framework and ask learners to reproduce in another.

::::::::::::::::::::::::::::::::::::::::::::::::


## What frameworks will we look at?

- [Veggies](https://gitlab.com/everythingfunctional/veggies)
    - Integrated with FPM and CMake.
- [pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit)
    - Most feature rich framework.
    - Requires writing tests in a non-standard file format which is then converted to F90 before compilation.
    - Integrated with CMake.
- [test-drive](https://github.com/fortran-lang/test-drive)
    - This is the least featured of the frameworks.
    - Requires more boilerplate than the other frameworks.
    - Integrated with FPM and CMake.

## What are shared between the frameworks?

- Defining types for test parameters
- Creating `test suites` to separate tests for different units or modules into isolated blocks.

### The basic structure of a test module

All three frameworks share a basic structure for their test modules.

```f90 
module test_something
    ! use veggies or testdrive or funit
    ! use the src to be tested
    implicit none

    ! Define types to act as test parameters
contains

    ! Define a test suite (collection of tests) to be returned from a procedure

    ! Define the actual test execution code which will call the src and execute assertions

    ! Define constructors for the test parameter types
end module test_something
```

## Let's dive into the syntax

We will use the game of life example from challenge 1 of the last episode to highlight the difference in syntax between the three frameworks.

### Define derived types to act as test parameters

This step is similar for all three frameworks and uses standard Fortran syntax to define a [derived type](https://fortran-lang.org/learn/quickstart/derived_types).

The key differences are...

- Whether the derived type inherits from another type or not.
- The required [type-bound procedures](https://fortran-lang.org/learn/quickstart/derived_types/#type-bound-procedures).

Take a look below at how the definition of `check_for_steady_state_in_out_t` changes between the three frameworks.

#### veggies

```f90
type, extends(input_t) :: check_for_steady_state_in_out_t
    integer, dimension(:,:), allocatable :: current_board, new_board
    logical :: expected_steady_state
end type check_for_steady_state_in_out_t
interface check_for_steady_state_in_out_t
    module procedure check_for_steady_state_in_out_constructor
end interface check_for_steady_state_in_out_t
```

Unique aspects:

- Our derived type extends `input_t`, a derived type from veggies.
- We define an interface for the constructor of our type.

#### pFUnit

```f90
@testParameter
type, extends(AbstractTestParameter) :: check_for_steady_state_in_out_t
    integer, dimension(:,:), allocatable :: current_board, new_board
    logical :: expected_steady_state
contains
      procedure :: toString => check_for_steady_state_in_out_toString
end type check_for_steady_state_in_out_t
```

Unique aspects:

- Our derived type has a decorated, `@testParameter`, which pFUnit will use when converting to F90.
- Our derived type extends `AbstractTestParameter`, a derived type from pFUnit.
- We define a type-bound procedure for how to convert our derived type to a string. This used to output our parameters when printing test information during execution.

#### test-drive

```f90
type :: check_for_steady_state_in_out_t
    integer, dimension(:,:), allocatable :: current_board, new_board
    logical :: expected_steady_state
end type check_for_steady_state_in_out_t
```
