<!-- markdownlint-disable MD029 -->
# Fortran Unit Test Syntax - Solution: Write tests using pFUnit

## Building

The solution provided here is an entirely self-contained project which can be built using CMake.

```bash
cmake -B build -DCMAKE_PREFIX_PATH=/path/to/pfunit/install
cmake --build build
./build/game-of-life ../models/model-1.dat # Or another data file
```

## Tasks

> 1. Update the [test/CMakeLists.txt](./test/CMakeLists.txt) or [test/Makefile](./test/Makefile) to build the fully implemented test,
>    **test_evolve_board.pf**.

The test has been added to [test/CMakeLists.txt](./test/CMakeLists.txt) as a new `add_pfunit_ctest`. The test has also been added
to the list **tests_TESTS** in [test/Makefile](./test/Makefile).

> 2. Finish the partially implemented tests in [test](./test/) and ensure they are built via CMake or Make.

Completed pFUnit tests can be found in [test](./test/) and each has been added to the
[test/CMakeLists.txt](./test/CMakeLists.txt) as a new `add_pfunit_ctest` and to the list **tests_TESTS** in [test/Makefile](./test/Makefile).

> 3. Write a completely new pFUnit test for the subroutine `find_steady_state` in
>    [game_of_life_mod.f90](./src/game_of_life_mod.f90) and ensure it is built via CMake or Make.

The new test can be found in
[test/test_find_steady_state.pf](./test/test_find_steady_state.pf) and
has been added to the end of [test/CMakeLists.txt](./test/CMakeLists.txt) as a new `add_pfunit_ctest` and to the list
**tests_TESTS** in [test/Makefile](./test/Makefile).
