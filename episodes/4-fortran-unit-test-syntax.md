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


## What frameworks will we look at?

- [Veggies](https://gitlab.com/everythingfunctional/veggies)
    - Integrated with FPM and CMake.
- [test-drive](https://github.com/fortran-lang/test-drive)
    - This is the least featured of the frameworks.
    - Requires more boilerplate than the other frameworks.
    - Integrated with FPM and CMake.
- [pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit)
    - Most feature rich framework.
    - Requires writing tests in a non-standard file format which is then converted to F90 before compilation.
    - Integrated with CMake.

## The shared structure of a test module

All three frameworks share a basic structure for their test modules.

```f90 
module test_something
    ! use veggies|testdrive|funit
    ! use the src to be tested
    implicit none

    ! Define types to act as test parameters (and test case for pfunit)
contains

    ! Define a test suite (collection of tests) to be returned from a procedure

    ! Define the actual test execution code which will call the src and execute assertions

    ! Define constructors for your derived types (test parameters/cases)
end module test_something
```

## Let's dive into the syntax

We will use the game of life example from challenge 1 of the last episode to highlight the difference in
syntax between the three frameworks.


### Define types to act as test parameters (and test case for pfunit)

This step is similar for all three frameworks and uses standard Fortran syntax to define a [derived type](https://fortran-lang.org/learn/quickstart/derived_types). The key differences are:

- Whether the derived type extends another type or not.
- The required [type-bound procedures](https://fortran-lang.org/learn/quickstart/derived_types/#type-bound-procedures).
- Whether a test case derived type is needed.

::::::::::::::::::::::::::::: spoiler

#### Veggies

```F90
type, extends(input_t) :: my_test_params
    integer :: input, expected_output
end type my_test_params
```

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### test-drive

```F90
type :: my_test_params
    integer :: input, expected_output
end my_test_params
```

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### pFUnit

```F90
@testParameter
type, extends(AbstractTestParameter) :: my_test_params
    integer :: input, expected_output
contains
        procedure :: toString => my_test_params_toString
end type my_test_params

@TestCase(testParameters={my_test_suite()}, constructor=my_test_params_to_my_test_case)
type, extends(ParameterizedTestCase) :: my_test_case
    type(my_test_params) :: params
end type my_test_case
```

:::::::::::::::::::::::::::::::::::::

### Define a test suite (collection of tests) to be returned from a procedure

In this section we define our suite of tests to test the unit in question. This can return a single test but
it's likely that there are multiple scenarios and edge cases we would like to test. Therefore, we return an
array of tests rather than a single test.

::::::::::::::::::::::::::::: spoiler

#### Veggies

For Veggies, we define a function which returns a Veggies derived-type that takes an array of test parameters
representing different test scenarios and a generic test function, in this case `check_my_src_procedure`. This
test function is where we actually call our src procedure and carry out assertions (see [the next section](#define-the-actual-test-execution-code-which-will-call-the-src-and-execute-assertions)). 

```F90
function my_test_suite() result(tests)
    type(test_item_t) :: tests

    type(example_t) :: my_test_data(1)

    ! Given input is 1, output is 2
    my_test_data(1) = example_t(my_test_params(1, 2))

    tests = describe( &
        "my_src_procedure", &
        [ it( &
            "given some inputs, when I call my_src_procedure, Then we get the expected output", &
            my_test_data, &
            check_my_src_procedure &
        )] &
    )
end function my_test_suite
```

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### test-drive

For test-drive, we define a subroutine which populates an array of test parameters called the testsuite.
To build this testsuite we provide additional subroutines which actually set up the test parameters and
then call the test function.

```F90
subroutine my_test_suite(testsuite)
    type(unittest_type), allocatable, intent(out) :: testsuite(:)

    testsuite = [ &
        new_unittest("my_src_procedure: Given input is 1, output is 2", test_my_procedure_with_input_1) &
    ]
end subroutine my_test_suite

!> Given input is 1, output is 2
subroutine test_my_procedure_with_input_1(error)
    type(error_type), allocatable, intent(out) :: error

    type(my_test_params) :: params

    params%input = 1
    params%expected_output = 2

    call check_my_src_procedure(error, params)
end subroutine test_my_procedure_with_input_1
```

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### pFUnit

For pFUnit, we define a function which returns an array of our test parameter derived-type.

```F90
function my_test_suite() result(params)
    type(my_test_params), allocatable :: params(:)

    params = [ &
        my_test_params(1, 2) & ! Given input is 1, output is 2
    ]
end function my_test_suite
```

:::::::::::::::::::::::::::::::::::::

### Define the actual test execution code which will call the src and execute assertions

This is where we actually call our src procedure and carry out assertions.

::::::::::::::::::::::::::::: spoiler

#### Veggies

For Veggies, we define a function which takes a veggies `input_t` type and returns a veggies `result_t`
type. As this input_t type is generic compared to out parameter type, we do some additional verification
to ensure we are passing the expected test parameter type.

```F90
function check_my_src_procedure(params) result(result_)
    class(input_t), intent(in) :: params
    type(result_t) :: result_

    integer :: actual_output

    select type (params)
    type is (my_test_params)
        call my_src_procedure(params%input, actual_output)

        reult_ = assert_equal(params%expected_output, actual_output, "Unexpected output from my_src_procedure")
    class default
        result_ = fail("Didn't get my_test_params")

    end select

end function check_my_src_procedure
```

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### test-drive

For test-drive, we define a subroutine which takes an error and an instance of our test parameters derived-type.

```F90
subroutine check_my_src_procedure(error, params)
    type(error_type), allocatable, intent(out) :: error
    class(my_test_params), intent(in) :: params

    integer :: actual_output
    
    call my_src_procedure(params%input, actual_output)

    call check(error, params%expected_output, actual_output, "Unexpected output from my_src_procedure")
    if (allocated(error)) return
end subroutine check_my_src_procedure
```

::::::::::::::::::::::::::::::::::::: callout

We must check if error has been allocated after every `check`. i.e.

```F90
call check(...)
if (allocated(error)) return

call check(...)
if (allocated(error)) return
```

:::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### pFUnit

For pFUnit, we define a subroutine which takes an instance of our test case derived-type and is annotated
with the pFUnit annotation `@Test`.

```F90
@Test
subroutine TestMySrcProcedure(this)
    class (my_test_case), intent(inout) :: this

    integer :: actual_output

    call my_src_procedure(this%params%input, actual_output)

    @assertEqual(this%params%input, actual_output, "Unexpected output from my_src_procedure")
end subroutine TestMySrcProcedure
```

:::::::::::::::::::::::::::::::::::::

### Define constructors for your derived types (test parameters/cases)

For **Veggies** and **test-drive**, this step is not always required but can be useful to simplify
populating multiple different test cases. For example, if we wished to test a subroutine which
performs some operations on a large matrix we could create a constructor to populate this matrix
with random values. We would then need to call this constructor with different
inputs to generate multiple test cases.

If we want to add a constructor for these types, it must be declared, at this point as an interface to
the derived-type

::::::::::::::::::::::::::::: spoiler

### Veggies and test-drive

Shown here is how to create an arbitrarily simple constructor. This would not actually be necessary as
compilers can handle this for us. However, we use the same syntax for more complex derived types. First,
declare your constructor,

```f90
interface my_test_params
    module procedure my_test_params_constructor 
end interface my_test_params
```

Then, implement your constructor,

```f90
contains
    function my_test_params_constructor(input, expected_output) result(params)
        integer, intent(in) :: input, expected_output

        type(my_test_params) :: params

        my_test_params%input = input
        my_test_params%expected_output = expected_output
    end function check_for_steady_state_in_out_constructor
```

:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::: spoiler

#### pFUnit

For pFUnit, we are required to define two functions

- A conversion from test parameters to a test case
- A conversion from test parameters to a string

```F90
function my_test_params_to_my_test_case(testParameter) result(tst)
    type (my_test_case) :: tst
    type (my_test_params), intent(in) :: testParameter

    tst%params%input = testParameter%input
    tst%params%expected_output = testParameter%expected_output
end function my_test_params_to_my_test_case

function my_test_params_toString(testParameter) result(string)
    class (my_test_params), intent(in) :: this
    character(:), allocatable :: string

    character(len=80) :: buffer

    write(buffer,'("Given ",i4," we expect to get ",i4)') this%input, this%expected_output
    string = trim(buffer)
end function my_test_params_toString
```
:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge: Write Fortran unit tests in multiple frameworks.

Take a look at [4-fortran-unit-test-syntax/challenge](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/4-fortran-unit-test-syntax/challenge).

:::::::::::::::::::::::::::::::: solution

A solution is provided in [4-fortran-unit-test-syntax/solution](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/4-fortran-unit-test-syntax/solution).

:::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
