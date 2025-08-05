---
title: "Setup"
---

## Exercises repository

Throughout this walkthrough, we will use the repository [UCL-ARC/fortran-unit-testing-exercises](https://github.com/UCL-ARC/fortran-unit-testing-exercises) which contains example exercises written in Fortran.

::::::::::::::::::::::::::::::::::::: challenge

### Challenge 1: Can you clone the exercises repository

Try to clone the exercises we will use throughout this lesson.

:::::::::::::::::::::::::::: solution

```bash
$ git clone https://github.com/UCL-ARC/fortran-unit-testing-exercises.git
Cloning into 'fortran-unit-testing-exercises'...
remote: Enumerating objects: 256, done.
remote: Counting objects: 100% (256/256), done.
remote: Compressing objects: 100% (140/140), done.
remote: Total 256 (delta 98), reused 229 (delta 71), pack-reused 0 (from 0)
Receiving objects: 100% (256/256), 45.96 KiB | 4.18 MiB/s, done.
Resolving deltas: 100% (98/98), done.
```

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::

## Software Setup

To following along with this lesson's exercises you will require the following

- Fortran Package Manager (FPM)
- CMake
- pFUnit

::::::::::::::::::::::::::::::::::::: challenge

### Challenge 2: Can you install the above dependencies?

Try to install the dependencies listed above. 

- FPM [Install instructions](https://fpm.fortran-lang.org/install/index.html)
- CMake can be installed via [homebrew](https://formulae.brew.sh/formula/cmake) on mac or your package manager (apt, etc) on Linux.
- pFUnit can be install via the bash script provided in the exercises repo [build-pfunit.sh](https://github.com/UCL-ARC/fortran-unit-testing-exercises/build-pfunit.sh).

:::::::::::::::::::::::::::: solution

```bash
$ fpm --version
Version:     0.12.0, alpha

$ cmake --version
cmake version 3.27.0

$ ./build-pfunit.sh -t
[  0%] Built target posix_predefined.x
[  0%] Built target generate_posix_parameters
[ 19%] Built target funit-core
[ 33%] Built target fhamcrest
[ 55%] Built target asserts
[ 56%] Built target funit-main
[ 56%] Built target funit
[ 57%] Built target pfunit-core
[ 58%] Built target pfunit
[ 59%] Built target new_ptests
[ 60%] Built target new_ptests.x
[ 62%] Built target other_shared
[ 67%] Built target funit_tests
[ 68%] Built target funit_tests.x
[ 69%] Built target robust
[ 70%] Built target remote.x
[ 72%] Built target robust_tests.x
[ 88%] Built target new_tests.x
[ 97%] Built target fhamcrest_tests.x
[ 99%] Built target pfunittests
[100%] Built target parallel_tests.x
[100%] Built target build-tests
      Start  1: unit_test_processor
 1/14 Test  #1: unit_test_processor ........................   Passed    0.09 sec
      Start  2: processor_test_MpiParameterizedTestCaseC
 2/14 Test  #2: processor_test_MpiParameterizedTestCaseC ...   Passed    0.32 sec
      Start  3: processor_test_MpiTestCaseB
 3/14 Test  #3: processor_test_MpiTestCaseB ................   Passed    0.08 sec
      Start  4: processor_test_ParameterizedTestCaseB
 4/14 Test  #4: processor_test_ParameterizedTestCaseB ......   Passed    0.06 sec
      Start  5: processor_test_TestA
 5/14 Test  #5: processor_test_TestA .......................   Passed    0.07 sec
      Start  6: processor_test_TestCaseA
 6/14 Test  #6: processor_test_TestCaseA ...................   Passed    0.06 sec
      Start  7: processor_test_beforeAfter
 7/14 Test  #7: processor_test_beforeAfter .................   Passed    0.06 sec
      Start  8: processor_test_simple
 8/14 Test  #8: processor_test_simple ......................   Passed    0.06 sec
      Start  9: old_tests
 9/14 Test  #9: old_tests ..................................   Passed    0.05 sec
      Start 10: robust_tests.x
10/14 Test #10: robust_tests.x .............................   Passed    0.02 sec
      Start 11: new_tests.x
11/14 Test #11: new_tests.x ................................   Passed    0.03 sec
      Start 12: fhamcrest_tests.x
12/14 Test #12: fhamcrest_tests.x ..........................   Passed    0.02 sec
      Start 13: mpi-tests
13/14 Test #13: mpi-tests ..................................   Passed    0.19 sec
      Start 14: new_ptests
14/14 Test #14: new_ptests .................................   Passed    0.12 sec

100% tests passed, 0 tests failed out of 14

Total Test time (real) =   1.26 sec
```

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
