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

    ! Define types to act as test parameters (and test case for pfunit)
contains

    ! Define a test suite (collection of tests) to be returned from a procedure

    ! Define the actual test execution code which will call the src and execute assertions

    ! Define any constructors for your derived types (test parameters/cases)
end module test_something
```

## Let's dive into the syntax

We will use the game of life example from challenge 1 of the last episode to highlight the difference in syntax between the three frameworks.

### Define types to act as test parameters (and test case for pfunit)

This step is similar for all three frameworks and uses standard Fortran syntax to define a [derived type](https://fortran-lang.org/learn/quickstart/derived_types).

The key differences are...

- Whether the derived type extends another type or not.
- The required [type-bound procedures](https://fortran-lang.org/learn/quickstart/derived_types/#type-bound-procedures).
- Whether a test case derived type is needed.

>NOTE: If we want to add a constructor for these types, it must be declared as
> an interface to the derived-type
> ```F90
> interface my_test_params
>   module procedure mu_test_params_constructor
> end interface my_test_params
> ```

#### Veggies

```F90
type, extends(input_t) :: my_test_params
    ! Declare some test parameters
    integer :: input, expected_output
end type my_test_params
```

#### test-drive

```F90
type :: my_test_params
    ! Declare some test parameters
    integer :: input, expected_output
end my_test_params
```

#### pFUnit

```F90
@testParameter
type, extends(AbstractTestParameter) :: my_test_params
    ! Declare some test parameters
    integer :: input, expected_output
contains
        procedure :: toString => my_test_params_toString
end type my_test_params

@TestCase(testParameters={my_test_suite()}, constructor=my_test_params_to_my_test_case)
type, extends(ParameterizedTestCase) :: my_test_case
    type(my_test_params) :: params
end type my_test_case
```

### Define a test suite (collection of tests) to be returned from a procedure

The principle behind this step is the same for all three frameworks but the implementation differs somewhat.

For Veggies and pFUnit, we define a function which returns an array of derived-types. For veggies 

#### Veggies

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

#### test-drive

```F90
subroutine my_test_suite(testsuite)
    type(unittest_type), allocatable, intent(out) :: testsuite(:)

    testsuite = [ &
        new_unittest("call my_src_procedure with input of 1", test_my_procedure_with_input_1) &
    ]
end subroutine my_test_suite

subroutine test_my_procedure_with_input_1(error)
    type(error_type), allocatable, intent(out) :: error

    type(my_test_params) :: params

    params%input = 1
    params%expected_output = 1

    call check_my_src_procedure(error, params)
end subroutine test_my_procedure_with_input_1
```

#### pFUnit

```F90
function my_test_suite() result(params)
    type(my_test_params) :: params(1)

    ! Given input is 1
    params(1) = my_test_params(1)
end function my_test_suite
```

### Define the actual test execution code which will call the src and execute assertions

This is where the src code to be tested and assertion procedures provided by the test frameworks will be called.

#### Veggies

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

#### test-drive

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

#### pFUnit

```F90
@Test
subroutine TestMySrcProcedure(this)
    class (my_test_case), intent(inout) :: this

    integer :: actual_output

    call my_src_procedure(this%params%input, actual_output)

    @assertEqual(this%params%input, actual_output, "Unexpected output from my_src_procedure")

    deallocate(actual_new_board)
end subroutine TestMySrcProcedure
```

## Define any constructors for your derived types (test parameters/cases)

For **Veggies** and **test-drive**, this step is not always required but can be useful to simplify
populating multiple different test cases. For example, if we wished to test a subroutine which
performs some operations on a large matrix we could create a constructor to populate this matrix
with random values, for example. We would then just need to call this constructor with different
inputs to generate multiple test cases.

For **pFUnit** we are also required to define functions to convert from test parameters to both a test
case and a string.

#### pFUnit

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
