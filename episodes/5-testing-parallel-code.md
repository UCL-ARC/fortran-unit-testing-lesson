---
title: "Testing parallel code"
teaching:
exercises:
---

:::::::::::::::::::::::::::::::::::::: questions 

- How do I unit test a procedure which makes MPI calls?
- How do I easily test different numbers of MPI ranks?
- How do I test a procedure which uses OMP directives?
- How do I easily test different numbers of OMP threads?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand what is different when testing parallel vs serial code.

::::::::::::::::::::::::::::::::::::::::::::::::

## What's the difference?

Depending on the parallelisation tool and strategy employed, the implementation of parallel code can be very different
to that of serial code. This is especially true for code which utilises the message passing interface (MPI). These codes
almost always contain some functionality in which processes, or ranks, communicate by exchanging messages. This message
passing is often complex and will always benefit from testing.

There is added complexity when testing MPI code compared to serial as the logical path through the code is changed
depending on the number of ranks with which the code is executed. Therefore, it is important that we test for a range
of numbers of ranks. This will require controlling the number of ranks running the src and is not something we want
to implement ourselves. This limits the tools available to us. pFUnit is currently the only tool which supports
testing MPI code. Therefore, we will be focusing on pFUnit for this section.

## Tips for writing testable MPI code

### Where possible, separate calls to the MPI library into units (subroutines or functions).

If a procedure does not contain any calls to the MPI library, then it can be tested with a serial unit test. Therefore,
separating MPI calls into their own units makes for a simpler test suite for most of your logic. Only, procedures with
MPI library calls will require the more complex parallel pFUnit tests.

### Pass the MPI communicator information into each procedure to be tested.

If we pass the MPI communicator into a procedure, we can define this to be whatever we wish in our tests. This allows us
to use the communicator provided by pFUnit or some other communicator specific to our problem.

Creating types to wrap this information along with any other MPI specific information (neighbour ranks, etc) can be a
convenient approach. 

## Syntax of writing MPI enabled pFUnit tests

Firstly, we must change how we define our test parameters:

- We now use `MPITestParameter` instead of `AbstractTestParameter`.
    - `MPITestParameter` inherits from `AbstractTestParameter` and provides an additional parameter in its constructor which
    corresponds to the number of processors for which a particular test should be ran.
- We can't know for certain the rank of each process for the pFUnit communicator until the test case runs. Therefore, we
  now need to build arrays of input parameters with the rank of a process matching the index of the parameter array. For
  example, rank 0 would access index 1 of the input array during testing, rank 1 would access index 2 and so on. See below
  for an example.

```F90
@testParameter(constructor=new_exchange_boundaries_test_params)
type, extends(MPITestParameter) :: my_test_params
    integer, allocatable :: input(:), expected_output(:)
contains
    procedure :: toString => my_test_params_toString
end type my_test_params
```

We therefore need to update how we populate our test parameters to take into account the rank indexing:

```F90
function my_test_suite() result(params)
    type(my_test_params), allocatable :: params(:)
    integer, allocatable :: input(:), expected_output(:)
    integer, max_number_of_ranks

    max_number_of_ranks = 2
    allocate(params(max_number_of_ranks))
    allocate(input(max_number_of_ranks))
    allocate(expected_output(max_number_of_ranks))

    ! Tests with one rank
    input(1) = 1
    expected_output(1) = 2
    params(1) = my_test_params(1, input, expected_output)

    ! Tests with two ranks
    !     rank 0
    input(1) = 1
    expected_output(1) = 1
    !     rank 1
    input(2) = 1
    expected_output(2) = 1
    params(2) = my_test_params(2, input, expected_output)
end function my_test_suite
```

We also need to change how we define our test case:

- We now use `MPITestCase` instead of `ParameterizedTestCase`
    - `MPITestCase` provides several helpful methods for us to use whilst testing
        - `getProcessRank()` returns the rank of the current process allowing per rank selection of inputs and expected outputs.
        - `getMpiCommunicator()` returns the MPI communicator created by pFUnit to control the number of ranks per test.
        - `getNumProcesses()` returns the number of MPI ranks for the current test.

```F90
@TestCase(testParameters={my_test_suite()}, constructor=my_test_params_to_my_test_case)
type, extends(MPITestCase) :: my_test_case
    type(my_test_params) :: params
end type my_test_case
```

Finally, we ensure each process accesses the correct rank index parameters during the test

```F90
@Test
subroutine TestMySrcProcedure(this)
    class (my_test_case), intent(inout) :: this

    integer :: actual_output, rank_index

    rank_index = this%getProcessRank() + 1

    call my_src_procedure(this%params%input(rank_index), actual_output)

    @assertEqual(this%params%expected_output(rank_index), actual_output, "Unexpected output from my_src_procedure")
end subroutine TestMySrcProcedure
```

::::::::::::::::::::::::::::::::::::: challenge 

### Challenge 1: Testing MPI parallel code

Take a look at [5-testing-parallel-code/challenge-1](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/5-testing-parallel-code/) in the exercises repository.

:::::::::::::::::::::::::::::::: solution

A solution is provided in [episodes/5-testing-parallel-code/challenge-1/solution](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/5-testing-parallel-code/solution).

:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
