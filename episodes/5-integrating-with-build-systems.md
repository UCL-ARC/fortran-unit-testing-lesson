---
title: "Integrating with build systems"
teaching:
exercises:
---

:::::::::::::::::::::::::::::::::::::: questions 

- How do we go from **.pf** files to an executable test?
- How do we identify which test is failing and where?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Be able to add a new test to our existing Make and CMake build system.
- Understand where we name our tests within the build system.
- Diagnose a failing test and find the cause.

::::::::::::::::::::::::::::::::::::::::::::::::

## Integrating pFUnit with Make

Let's break down the steps required to add pFUnit tests to a project built using Make.
Firstly, assume we have the following file structure.

```
|-- ROOT_DIR/
    | Makefile
    |-- src/
    |   |-- main.f90
    |   |__ ... Some module files containing src code
    |      
    |-- tests/
        |-- Makefile
        |-- test_something.pf
        |-- test_something_else.pf
```
The top level **Makefile** is responsible for compiling the src code but
should do very little regarding building the tests. However, it should...

- Export relevant variables for the **tests/Makefile** to pick up.
- Define targets which pass through to targets in the **tests/Makefile**.

```
# ... Rest of Makefile

# Export relevant variables for the tests/Makefile to pick up.
export SRC_BUILD_DIR
export ROOT_DIR
export SRC_OBJS
export FC
export FC_FLAGS
export LIBS

# Define targets which pass through to targets in the tests/Makefile.
tests: $(SRC_OBJS)
	@echo "Building pFUnit test suite..."
	@$(MAKE) -C $(ROOT_DIR)/tests tests
```

::::::::::::::::::::: spoiler

#### Full file

The full top level **Makefile** may look something like this...

```
# Define relevant paths
ROOT_DIR = $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
SRC_DIR = $(ROOT_DIR)/src
BUILD_DIR = $(ROOT_DIR)/build
SRC_BUILD_DIR = $(BUILD_DIR)/src

# Define compiler flags
FC = gfortran
FC_FLAGS = #... Some flags required for compilation
LIBS = #... Some libs to link to

# List src files
SRC_FILES = \
    src/something.f90 \
    src/main.f90

# Map src files to .o files
SRC_OBJS = $(patsubst %.f90, $(BUILD_DIR)/%.o, $(SRC_FILES))

# Build src executable
$(SRC_BUILD_DIR)/a.exe: $(SRC_OBJS)
	$(FC) -o $@ $(FC_FLAGS) $^ $(LIBS) 

# Map exe target to building executable
exe: $(SRC_BUILD_DIR)/a.exe

# Build src .o files
$(SRC_BUILD_DIR)/%.o: $(SRC_DIR)/%.f90 | $(SRC_BUILD_DIR)
	$(FC) -c $(FC_FLAGS) -J $(SRC_BUILD_DIR) -o $@ $<

# Define target for cleaning the build dir
clean:
	rm -rf $(BUILD_DIR)
	$(MAKE) -C $(ROOT_DIR)/tests clean

.PHONY: clean

# Ensure the build dirs exists
$(SRC_BUILD_DIR):
	mkdir -p $@

# Export relevant variables for the tests/Makefile to pick up.
export SRC_BUILD_DIR
export ROOT_DIR
export SRC_OBJS
export FC
export FC_FLAGS
export LIBS

# Define targets which pass through to targets in the tests/Makefile.
tests: $(SRC_OBJS)
	@echo "Building pFUnit test suite..."
	@$(MAKE) -C $(ROOT_DIR)/tests tests
```

:::::::::::::::::::::::::::::

The **tests/Makefile** would then look like this...

```
PFUNIT_SOURCE_DIR = $(ROOT_DIR)/../pfunit
PFUNIT_INCLUDE_DIR = $(PFUNIT_SOURCE_DIR)/build/installed/PFUNIT-4.12/include
include $(PFUNIT_INCLUDE_DIR)/PFUNIT.mk

TEST_FLAGS = -I$(SRC_BUILD_DIR) $(FC_FLAGS) $(LIBS) $(PFUNIT_EXTRA_FFLAGS)

# Define variables to be picked up by make_pfunit_test
tests_TESTS = \
  test_something.pf \
  test_something_else.pf
tests_OTHER_SOURCES = $(filter-out $(SRC_BUILD_DIR)/main.o, $(SRC_OBJS))
tests_OTHER_LIBRARIES = $(TEST_FLAGS)

# Triggers pre-processing and defines rule for building test executable
$(eval $(call make_pfunit_test,tests))

# Converts pre-processed test files into objects ready for building of the executable
%.o: %.F90
	$(FC) -c $(TEST_FLAGS) $<

clean:
	\rm -f *.o *.mod *.F90 *.inc tests
```

**Key points:**

- We must include the pre-installed pFUnit dependencies and Makefile options via the **PFUNIT.mk** file.
  - The version of pFUnit that has been built will affect the path to this file (i.e. **.../installed/PFUNIT-4.15/include/...**)
- We are utilising the function provided by pFUnit **make_pfunit_test**
  - This will create a target of the provided name (in this case **tests**)
  - We define the variables pFUnit requires to build the **tests** target as variables prefixed with **tests_**.
    - **tests_TESTS** - A list of the **.pf** test files to be pre-processed before compilation.
    - **tests_OTHER_SOURCES** - A list of src object files required for the tests defined within **tests** (excluding the src main/program file)
    - **tests_OTHER_LIBRARIES** - A list of library flags to pass to the compiler when compiling the test code
- We must create a target for compiling object files which uses the same flags as **tests_OTHER_LIBRARIES**
