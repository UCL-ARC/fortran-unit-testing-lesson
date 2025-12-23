---
title: "Integrating with build systems"
teaching:
exercises:
---

:::::::::::::::::::::::::::::::::::::: questions 

- How do we go from `.pf` files to an executable test?
- How do we identify which test is failing and where?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Be able to add a new test to our existing Make and CMake build system.
- Understand where we name our tests within the build system.
- Diagnose a failing test and find the cause.

::::::::::::::::::::::::::::::::::::::::::::::::

## Integrating pFUnit with Make

- Include pfunit
- Define variables for pFunit to pick up prefixed with the name of your tests
    - Filter out main src
    - Link with pre-built src file objects
    - Link with the libs required b the src
- Call make_pfunit_tests with the name of your tests

### Test Makefile

Lets break down the steps required to add pFUnit tests to a project built using Make.
Firstly, assume we have a `WORK_DIR` variable defined as the root of the following file structure.

```
-- WORK_DIR
    | Makefile
    |-- src
    |   |-- Makefile
    |   |-- main.f90
    |   |__ ... Some module files containing src code
    |      
    |-- tests
        |-- Makefile
        |-- test_something.pf
        |-- test_something_else.pf
```

The top level `Makefile` may look something like this

```make
# Define relevant paths
WORK_DIR = $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))/
SRC_DIR = $(WORK_DIR)/src
TEST_DIR = $(WORK_DIR)/tests
BUILD_DIR = $(WORK_DIR)/build

# Define compiler flags
FC_FLAGS = ... Some flags required for compilation
LIBS = ... Some libs to link to

# Define target for cleaning the build dir
clean:
	rm -rf build

# Phony targets
.PHONY: clean

# Export variables for the other Makefile to use
export BUILD_DIR
export SRC_DIR
export WORK_DIR
export SRC_FILES
export FC
export FC_FLAGS
export LIBS

# Include make command from src Makefile
exe:
	@$(MAKE) -C $(WORK_DIR)/src exe

# Include make command from tests Makefile
tests: $(OBJS)
	@echo "Building pFUnit test suite..."
	@$(MAKE) -C $(WORK_DIR)/tests tests
```

and the `src/Makefile` might look like this

```make
# List src files
SRC_FILES = \
    main.f90 \
    ... Some module files containing src code

# Map src files to .o files
OBJS = $(patsubst %.f90, $(BUILD_DIR)/%.o, $(SRC_FILES))

# Build .o files
$(OBJS) : $(BUILD_DIR)/%.o : %.f90
	$(FC) -c $(FC_FLAGS) -module $(BUILD_DIR) -o $@ $<

# Build src executable
$(BUILD_DIR)/a.exe: $(OBJS)
	$(FC) -o $@ $(FC_FLAGS) $^ $(LIBS)

# Map exe target to building executable
exe: $(BUILD_DIR)/a.exe

export OBJS
``` 

The `tests/Makefile` might then look like this...

```make
PFUNIT_SOURCE_DIR = $(WORK_DIR)/external/pFUnit
PFUNIT_INCLUDE_DIR = $(PFUNIT_SOURCE_DIR)/build/installed/PFUNIT-4.15/include
include $(PFUNIT_INCLUDE_DIR)/PFUNIT.mk

TEST_DEPS = $(filter-out $(BUILD_DIR)/main.o, $(SRC_OBJS))
TEST_FLAGS = -I$(BUILD_DIR) $(FC_FLAGS) $(LIBS) $(PFUNIT_EXTRA_FFLAGS)

tests_TESTS = \
	test_something.pf \
	test_something_else.pf 

tests_OTHER_SOURCES = $(addprefix $(SOURCE_DIR)/, $(TEST_DEPS))
tests_OTHER_LIBRARIES = $(TEST_FLAGS)

# Triggers pre-processing and defines rule for building test executable
$(eval $(call make_pfunit_test,tests))

# Converts pre-processed test files into objects ready for building of the executable 
%.o: %.F90
	$(FC) -c $(TEST_FLAGS) $<
```

## Integrating pFUnit with CMake



## Test output

pFUnit will output some helpful information to the terminal. We have a lot
of control over the contents of this output, whether that's for a failing
test or a passing test.

Several aspects of the output can be defined by us,

- The name and description of each test.
- The message printed in the event of a failed assertion.

## Defining the name and description of a test

pFUnit allows us to give a name to a test and to add a description which gives
a more detailed explanation of what exactly is being tested. We provide the
test name within the CMakeLists.txt. In the example below we define
a test with the name `test_my_src_procedure`.

```cmake
find_package(PFUNIT REQUIRED)
enable_testing()

# Filter out the main.f90 files. We can only have one main() function in our tests
set(PROJ_SRC_FILES_EXEC_MAIN ${PROJ_SRC_FILES})
list(FILTER PROJ_SRC_FILES_EXEC_MAIN EXCLUDE REGEX ".*main.f90")

# Create library for src code
add_library (sut STATIC ${PROJ_SRC_FILES_EXEC_MAIN})

# List all  test files
set(test_srcs
  "${PROJECT_SOURCE_DIR}/test/pfunit/test_my_src_procedure.pf"
)

add_pfunit_ctest (test_my_src_procedure
  TEST_SOURCES ${test_my_src_procedure}
  LINK_LIBRARIES sut # your application library
  )
```

The other aspect of a pFUnit test in which we can add a descriptor is the string
printed to describe an individual test case. This is defined within the toString
function. In the example below, we directly print a description contained within
the parameter set itself.

```F90
@testParameter
type, extends(AbstractTestParameter) :: test_my_src_procedure_params
    integer :: input, output
    character(len=100) :: description
contains
    procedure :: toString => test_my_src_procedure_params_toString
end type test_my_src_procedure_params
!..
function test_my_src_procedure_testsuite() result(params)
    !...
    ! Define a set of input params within the testsuite function
    params(1) = test_my_src_procedure_params(input, output, "Some description")
    !...
end function test_my_src_procedure_testsuite
!..
function test_my_src_procedure_params_toString(this) result(string)
    class(test_my_src_procedure_params), intent(in) :: this
    character(:), allocatable :: string

    string = trim(this%description)
end function test_my_src_procedure_params_toString
!..
```

This results in the output

```bash
$ ctest 
Test project /Users/connoraird/work/fortran-unit-testing-exercises/episodes/5-debugging-a-broken-test/challenge-1/build-cmake
    Start 2: pfunit_transpose_tests
1/1 Test #2: pfunit_transpose_tests ...........   Passed    0.24 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   0.55 sec
```

Notice that this is the output from ctest and if there are no test failures, only
a short summary is outputted. In the event of a failure we can get more detail
via the `--output-on-failure` flag

```bash
$ ctest --output-on-failure
    Start 1: pfunit_my_src_procedure_tests
1/1 Test #1: pfunit_my_src_procedure_tests ...........***Failed  Error regular expression found in output. Regex=[Encountered 1 or more failures/errors during testing]  0.01 sec
 

 Start: <test_my_src_procedure.TestMySrcProcedure[Some description][Some description]>
. Failure in <test_my_src_procedure.TestMySrcProcedure[Some description][Some description]>
F   end: <test_my_src_procedure.TestMySrcProcedure[Some description][Some description]>

Time:         0.000 seconds
  
Failure
 in: 
test_my_src_procedure.TestMySrcProcedure[Some description][Some description]
  Location: 
[test_my_src_procedure.pf:59]
ArrayAssertEqual failure:
      Expected: <2.00000000>
        Actual: <1.00000000>
    Difference: <1.00000000> (greater than tolerance of 0.999999975E-5)
  
 FAILURES!!!
Tests run: 1, Failures: 1, Errors: 0
, Disabled: 0
STOP *** Encountered 1 or more failures/errors during testing. ***


0% tests passed, 1 tests failed out of 1

Total Test time (real) =   0.02 sec

The following tests FAILED:
          1 - pfunit_my_src_procedure_tests (Failed)
Errors while running CTest
```

:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

### Challenge 1: Rename a test and improve its output.

Using one of the previous exercises we've looked at, try to rename a test in each of
the three frameworks. Can you improve the information outputted in the event of a
test failure?

::::::::::::::::::::::::::::::::::::::::::::::::

## Filtering tests

pFUnit provides its own mechanism for filtering tests as well as leverage CTest.
This can be useful when debugging a failing test in order to reduce the noise
printed to the terminal screen, especially if there are many tests failing and
you wish to tackle them one at a time.

::::::::::::::::::::::::::::: spoiler

### pFUnit filtering

It is possible to filter tests to only those that match a desired pattern. To do
so we can use the `--filter` option...

```
-f or --filter <pattern>        Only run tests that match the specified substring
```

For example, if we had both serial and mpi tests and we only wished to run the serial
ones, so long as we had included the word `serial` in our test name(s) we could filter
to run only them with the following command.

```sh
$ ./test.exe -f serial
```

:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::: spoiler

### CTest filtering

Tests can be filtered by name using CTests inbuilt regex filtering options...

```
-L <regex>, --label-regex <regex>    = Run tests with labels matching regular
                                       expression.  With multiple -L, run tests
                                       where each regular expression matches at
                                       least one label.
-R <regex>, --tests-regex <regex>    = Run tests matching regular expression.
-E <regex>, --exclude-regex <regex>  = Exclude tests matching regular expression.
-LE <regex>, --label-exclude <regex> = Exclude tests with labels matching regular
                                       expression.  With multiple -LE, exclude
                                       tests where each regular expression matches
                                       at least one label.
```

:::::::::::::::::::::::::::::::::::::

## Debugging a failing test

As with the output of a passing test, the output of a failing test differs
depending on the framework used to write them. As shown above, the
information we get from a test output is highly configurable. The more effort we
put in when writing tests the easier it will be to debug should it fail. For
example, it's clear which of the following options will be easier to debug should
the assertion fail.

```F90
! This will not be very clear
do row = 1, nrow
    do col = 1, ncol
        call check(error, params%expected_output_matrix(row, col), actual_output(row, col), &
            "Actual and expected output matrices did not match")
        if (allocated(error)) return
    end do
end do

! This is much better
do row = 1, nrow
    do col = 1, ncol
        write(failure_message,'(a,i1,a,i1,a,F3.1,a,F3.1)') "Unexpected value for output(", row, ",", col, "), got ", &
            actual_output(row, col), " expected ", params%expected_output_matrix(row, col)
        call check(error, params%expected_output_matrix(row, col), actual_output(row, col), failure_message)
        if (allocated(error)) return
    end do
end do
```

::::::::::::::::::::::::::::::::::::: challenge

### Challenge 2: Debug and fix a failing test.

Take a look at the [5-debugging-a-broken-test/challenge](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/5-debugging-a-broken-test/challenge) in the exercises repository.

:::::::::::::::::::::::::::::::: solution

A solution is provided in [README-solution.md](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/5-debugging-a-broken-test/solution).

:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
