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
- Understand the similarities between each framework and where they differ.

::::::::::::::::::::::::::::::::::::::::::::::::


## What framework will we look at?

There are multiple frameworks available for writing unit tests in Fortran, as detailed on the
[Fortran Lang website](https://fortran-lang.org/packages/programming/). However, we recommend
the use of [pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit) as it is...

- the most feature rich framework.
- the most widely used framework.
- being maintained.
- able to integrate with CMake and make.

**Key features of pFUnit:**

- **Supports MPI**: Supports testing MPI parallelized code, including parametrizing tests by
  number of MPI ranks.
- **Simple interface**: Tests are written in `.pf` format which is then pre-processed by a tool
  provided by pFUnit into `.f90` before compilation. This removes the need to write a lot of
  boilerplate code.

## The structure of a test module

All test modules share a basic structure...

```f90 
module test_something
    ! use funit
    ! use the src to be tested
    implicit none

    ! Derived types: Define types to act as test parameters and test cases.
contains

    ! Test Suite: Define a test suite (collection of tests) to be returned from a procedure.

    ! Test Logic: Define the actual test execution code which will call the src and execute assertions.

    ! Type Constructors: Define constructors for your derived types (test parameters/cases).
end module test_something
```

## Let's dive into the syntax

We will continue to use the temperature conversion example from the previous episode to cover
the syntax of pFUnit.

:::::::::::::::::::::::::::::::::::::::::::::::::::: spoiler

### Derived types:

This uses standard Fortran syntax to define some
[derived types](https://fortran-lang.org/learn/quickstart/derived_types). 

#### Test parameters

The test parameter type should contain the inputs and expected outputs of the code we are testing.

:::::::::::::::::::::::::::::: callout

#### Treat the src to be tested like a black box

When writing a unit test,

- The **inputs and outputs** are the important aspects to understand about our src code to be tested.
- **The implementation should not influence how we write our test**. Not every test needs to be
  parametrized, but you will always need to consider the inputs and outputs of the src code you
  are testing.

::::::::::::::::::::::::::::::::::::::

Firstly, the test parameter derived-type is written as...

```F90
@testParameter
type, extends(AbstractTestParameter) :: my_test_params
    integer :: input, expected_output
contains
    procedure :: toString => my_test_params_toString
end type my_test_params
```

**Key points:**

- Our parameter type must be decorated with **@testParameter** so that the pFUnit pre-processor
  understands that this derived type defines a test parameter.
- We must extend one of the base types provided by pFUnit, in this case **AbstractTestParameter**
  which is the most generic.
- We have declared a type-bound procedure **toString** which maps to the procedure
  **my_test_params_toString**. This allows pFUnit to log a helpful description of our parameter set
  which should be returned from **my_test_params_toString** (we'll see more on this later).

#### Test case

Then we can write our test case derived-type as...

```F90
@TestCase(constructor=my_test_params_to_my_test_case, testParameters={my_test_suite()})
type, extends(ParameterizedTestCase) :: my_test_case
    type(my_test_params) :: params
end type my_test_case
```

**Key points:**

- Our parameter type must be decorated with **@TestCase** so that the pFUnit pre-processor
  understands that this derived type defines a test case.
- The **@TestCase** decorator includes some extra information to tell the pre-processor how
  the test case should be constructed. What we have defined is...
    - To convert from an instance of **my_test_params** to an instance of **my_test_case**, one
      must call **my_test_params_to_my_test_case**.
    - The list of parameter sets which define each individual parametrized test will be
      returned from the function **my_test_suite**
- Just like with the test parameter type, we must extend one of the base types provided by
  pFUnit, in this case **ParameterizedTestCase** which indicates that this test should be
  parametrized.
- We then define a single type-bound value which is of the test parameter type we have just
  defined.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::: spoiler

### Test Suite:

In this section we define our parameter sets (or test suite). We define a function which
returns our test parameters like so...

```F90
function my_test_suite() result(params)
    type(my_test_params), allocatable :: params(:)

    params = [ &
        my_test_params(1, 2), & ! Given input is 1, output is 2
        my_test_params(3, 4) & ! Given input is 3, output is 4
    ]
end function my_test_suite
```

**Key points:**

- The function returns an array of **my_test_params**.
- We are using a constructor function to define each parameter set which we do not need to
  define ourselves.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::: spoiler

### Test Logic:

This is where we actually call our src procedure and carry out assertions...

```F90
@Test
subroutine TestMySrcProcedure(this)
    class (my_test_case), intent(inout) :: this

    integer :: actual_output

    call my_src_procedure(this%params%input, actual_output)

    @assertEqual(this%params%input, actual_output, "Unexpected output from my_src_procedure")
end subroutine TestMySrcProcedure
```

**Key points:**

- We must decorate the test subroutine with the pFUnit annotation **@Test** so the pre-processor
  knows this is a test.
- We are utilising a pre-processor directive provided by pFUnit **@assertEqual** which allows the
  exact comparison of two values (also works for comparing arrays). For a full list of the
  available assertion directives see
  [pFUnit documentation page for their preprocessor directives](https://pfunit.sourceforge.net/page_Assert.html)
    - As is done here, it is recommended to provide a helpful message in case of an assertion
      failing to help diagnose the issue.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::: spoiler

### Type Constructors:

We are required to define two functions.

#### A conversion from test parameters to a test case

```F90
function my_test_params_to_my_test_case(testParameter) result(tst)
    type (my_test_case) :: tst
    type (my_test_params), intent(in) :: testParameter

    tst%params = testParameter
end function my_test_params_to_my_test_case
```

:::::::::::::::::::::::::::::: callout

It may be necessary to individually map each type-bound value within the
**testParameter** to that in the **tst**, depending on their complexity.

::::::::::::::::::::::::::::::::::::::


#### A conversion from test parameters to a string

This function helps to provide a clearer description of each test case as the result
of this function will be displayed alongside the name of the test for each parameter
set.

```F90
function my_test_params_toString(testParameter) result(string)
    class (my_test_params), intent(in) :: this
    character(:), allocatable :: string

    character(len=80) :: buffer

    write(buffer,'("Given ",i4," we expect to get ",i4)') this%input, this%expected_output
    string = trim(buffer)
end function my_test_params_toString
```

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Test temperature conversions using pFUnit

Continuing with part two of [3-writing-your-first-unit-test/challenge][exercises-challenge] from the
exercises repo. Re-write your standard Fortran test using pFUnit.

:::::::::::::::::::::::::::::::: solution

A solution is provided in [3-writing-your-first-unit-test/solution](exercises-solution).

:::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::

[pfunit-directives]: https://pfunit.sourceforge.net/page_Assert.html3-writing-your-first-unit-test/challenge/src/temp_conversions.f90
[exercises-challenge]: https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/3-writing-your-first-unit-test/challenge
[exercises-solution]: https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/3-writing-your-first-unit-test/solution
