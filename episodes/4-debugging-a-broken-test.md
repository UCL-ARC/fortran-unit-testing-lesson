---
title: "Debugging a broken test"
teaching:
exercises:
---

:::::::::::::::::::::::::::::::::::::: questions 

- How do I know if my tests are failing?
- How do I fix a failing tests?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- [ ] Can build and edit the provided repo for the walkthrough.
    - *Assessment:* Following the build instructions successfully builds the code.
    - *Assessment:* Following the instructions successfully runs the pre-existing tests.
- [ ] Capable of determining where in the tests or src code the failure is occurring.
    - *Assessment:* Starting from a failing test, find the failing test-suite/module/procedure/line
- [ ] Can fix a failing test.
    - *Assessment:* Starting from a failing test, update the code so the test passes
- [ ] Understand where a test is defined in the build system (CMake and FPM).
    - *Assessment:* Add a new test to the build system
    - *Assessment:* Update the name of a test in the build system
    - *Assessment:* Enabled and fix a disabled test which is broken.
- [ ] Understand the failure output of a Fortran unit test written in...
    - *Assessment:* Starting from a failing test, update the code so the test passes

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

#### Challenge 1: Run the tests.

Take a look at [4-debugging-a-broken-test/challenge-1](https://github.com/UCL-ARC/fortran-unit-testing-exercises/episodes/4-debugging-a-broken-test/challenge-1) in the exercises repository.

Try and run the tests? Are they all passing? He's a hint, they're not. Which one(s) are failing? 

:::::::::::::::::::::::::::::::: solution

From within `episodes/4-debugging-a-broken-test/challenge-1`


:::::::::::: spoiler

#### Veggies

```
Running Tests

Test that
    transpose
        a matrix is transposed as expected

A total of 1 test cases

Failed
Took 4.47e-4 seconds

Test that
    transpose
        a matrix is transposed as expected
            Expected
                    |[[1.0, 4.0, 0.0],
                      [6.0, 1.0, 0.0],
                      [0.0, 3.0, 1.0]]|
                to be within |±1.0e-5| of
                    |[[1.0, 6.0, 0.0],
                      [4.0, 1.0, 3.0],
                      [0.0, 0.0, 1.0]]|

1 of 1 cases failed
1 of 2 assertions failed
```

::::::::::::::::::::

:::::::::::: spoiler

#### test-drive

```
# Running testdrive tests suite
# Testing: transpose
  Starting 3x3 identity matrix ... (1/2)
       ... 3x3 identity matrix [PASSED]
  Starting 3x3 asymmetric matrix ... (2/2)
       ... 3x3 asymmetric matrix [FAILED]
  Message: Unexpected value for output(1,2), got 4.0 expected 6.0                          
1 test(s) failed!
ERROR STOP 1

Error termination. Backtrace:
#0  0x10079ad2f
#1  0x10079b8d7
#2  0x10079cb1f
#3  0x100153503
#4  0x100153973
#5  0x1001539b7
```

::::::::::::::::::::

:::::::::::: spoiler

#### pFUnit

```
    Start 2: pfunit_transpose_tests
1/1 Test #2: pfunit_transpose_tests ...........***Failed  Error regular expression found in output. Regex=[Encountered 1 or more failures/errors during testing]  0.01 sec
 

 Start: <test_transpose_suite.TestTranspose[3x3 Identity][3x3 Identity]>
.   end: <test_transpose_suite.TestTranspose[3x3 Identity][3x3 Identity]>
 

 Start: <test_transpose_suite.TestTranspose[3x3 Asymmetric][3x3 Asymmetric]>
. Failure in <test_transpose_suite.TestTranspose[3x3 Asymmetric][3x3 Asymmetric]>
F   end: <test_transpose_suite.TestTranspose[3x3 Asymmetric][3x3 Asymmetric]>

Time:         0.000 seconds
  
Failure
 in: 
test_transpose_suite.TestTranspose[3x3 Asymmetric][3x3 Asymmetric]
  Location: 
[test_transpose.pf:59]
ArrayAssertEqual failure:
      Expected: <4.00000000>
        Actual: <6.00000000>
    Difference: <2.00000000> (greater than tolerance of 0.999999975E-5)
      at index: [2,1]
  
 FAILURES!!!
Tests run: 2, Failures: 1, Errors: 0
, Disabled: 0
STOP *** Encountered 1 or more failures/errors during testing. ***


0% tests passed, 1 tests failed out of 1

Total Test time (real) =   0.02 sec

The following tests FAILED:
          2 - pfunit_transpose_tests (Failed)
Errors while running CTest
```

::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::