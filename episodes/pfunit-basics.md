---
title: "pFUnit basics"
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
the use of [pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit) as it is…

- the most feature rich framework.
- the most widely used framework.
- being maintained.
- able to integrate with CMake and make.

**Key features of pFUnit:**

- **Supports MPI**: Supports testing MPI parallelized code, including parametrizing tests by
  number of MPI ranks.
- **Simple interface**: Tests are written in **.pf** format which is then pre-processed by a tool
  provided by pFUnit into **.f90** before compilation. This removes the need to write a lot of
  boilerplate code.

## The most basic pFUnit test

As we've seen in the [previous episode](../writing-your-first-unit-test.html), if we were to write our own unit tests using a
custom testing setup we would need to define a test runner that could track success and failure states for each test and report
the reason for each failure back to us.

Alternatively, if we were to use pFUnit, there is no longer a need to define this test runner because pFUnit handles that for us.
Therefore, the most basic test we can define using pFunit becomes simple. For example, if we wanted to test the Fortran intrinsic
function **dot_product**, we could write the following test.

```fortran
module test_dot_product_intrinsic
    use funit
    implicit none
contains
    @Test
    subroutine test_dot_product()
        integer :: a(10), b(10), c

        ! Define inputs and expected outputs for the scenario we want to test
        a = [1,2,3,4,5,6,7,8,9,10]
        b = [11,12,13,14,15,16,17,18,19,20]
        c = 935

        ! Check that the call to dot_product returned what we expect
        @assertEqual(c, dot_product(a, b), message="Unexpected value returned for the dot_product")

    end subroutine test_dot_product
end module test_dot_product_intrinsic
```

Here we have introduced some new syntax in the form of **@Test** and **@AssertEqual**. These are pFUnit pre-processor directives
which simplify how we write tests:

- **@Test** designates the subroutine **test_dot_product** as a test that should be ran on execution of your pFUnit test suite.
- **@AssertEqual** is one of many assert directives provided by pFUnit. More specifically, **@AssertEqual** allows the
  exact comparison of values (also works for comparing arrays). For a full list of the available assertion directives see
  [pFUnit documentation page for their preprocessor directives](https://pfunit.sourceforge.net/page_Assert.html)
  - As is done here, it is recommended to provide a helpful message, in case of an assertion
      failing, to help diagnose the issue.

::: callout

### @AssertEqual for floating point values

For floating point values, @AssertEqual no longer carries out an exact comparison but become a comparison up to a tolerance.

:::

If we then wish to add a new test case we can add another subroutine, again decorated with **@Test**:

```fortran
module test_dot_product_intrinsic
    use funit
    implicit none
contains
    @Test
    subroutine test_dot_product()
        integer :: a(10), b(10), c

        ! Define inputs and expected outputs for the scenario we want to test
        a = [1,2,3,4,5,6,7,8,9,10]
        b = [11,12,13,14,15,16,17,18,19,20]
        c = 935

        ! Check that the call to dot_product returned what we expect
        @AssertEqual(c, dot_product(a, b), message="Unexpected value returned for the dot_product")

    end subroutine test_dot_product

    @Test
    subroutine test_dot_product_all_zeros()
        integer :: a(10), b(10), c

        ! Define inputs and expected outputs for the scenario we want to test
        a = 0
        b = 0
        c = 0

        ! Check that the call to dot_product returned what we expect
        @AssertEqual(c, dot_product(a, b), message="Unexpected value returned for the dot_product")

    end subroutine test_dot_product_all_zeros
end module test_dot_product_intrinsic
```

::: challenge

### Challenge: Test temperature conversions using pFUnit

Continuing with part two of
[3-writing-your-first-unit-test/challenge](https://github.com/carpentries-incubator/fortran-unit-testing/tree/main/exercises/3-writing-your-first-unit-test/challenge)
from the exercises. Write a single test for the temperature conversion using pFUnit.

::: solution

A solution is provided in [3-writing-your-first-unit-test/solution](https://github.com/carpentries-incubator/fortran-unit-testing/tree/main/exercises/3-writing-your-first-unit-test/solution).

:::

:::
