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

## Test output

Each of the three frameworks print the test output to the terminal in different formats.
We have a lot of control over the contents of this output and what it looks like, whether
that's for a failing test or a passing test.

Several aspects of the output can be defined by us for all frameworks,

- The name and description of each test.
- The message printed in the event of a failed assertion.

For test-drive in particular, we have even more control over the output, as we are
required to write the test runner ourselves.

## Filtering tests

Each of the three frameworks offer the ability to filter the tests that run. This can
be useful when debugging a failing test in order to reduce the noise on the terminal
screen, especially if there are many tests failing and you want to tackle them one at
a time.

::::::::::::::::::::::::::::: spoiler
#### Veggies

Veggies comes with a built-in mechanism for filtering tests via the CLI flag `-f`.

```
-f string, --filter string    Only run cases or collections whose
                              description matches the given regular
                              expression. This option may be provided
                              multiple times to filter repeatedly before
                              executing the suite.
```

:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

### Challenge 1: Where do we define the name and description of a test

Use the previous exercises we've worked through to identify where, in each of the
three frameworks, we define the name and description of a test.

:::::::::::::::::::::::::::::::: solution

#### Veggies

The terms on which we can filter Veggies tests are defined within the testsuite function.

```F90
function my_test_suite() result(tests)
    type(test_item_t) :: tests
    type(example_t) :: my_test_data(1)
    
    ! Given input is 1, output is 2
    my_test_data(1) = example_t(my_test_params(1, 2))

    tests = describe( &
        "my_src_procedure", &
        [ it( &
            "with specific inputs something else specific should happen", &
            my_test_data, &
            check_my_src_procedure &
        )] &
    )
end function my_test_suite
```

The first argument given to `describe` defines an overarching descriptor which is
applied to all the tests within it. Each `it` then defines its own descriptor which
allows further filtering on a scenario by scenario basis. With the above example,
we could filter for this specific test with the following command

```sh
fpm test -- -f "my_src_procedure" -f "specific inputs"
```

#### Filtering with ctest

To maintian the ability to run our tests individually, we can add named tests to
ctest. To do this, we can add the following to the CMakeLists.txt.

```cmake
# Create a list of tests
set(
  tests
  "my_src_procedure"
)
#...
# Define test executable and Link library 
#...
# Define tests using the veggies test executable
foreach(t IN LISTS tests)
  add_test(NAME "veggies_${t}" COMMAND "test_${PROJECT_NAME}-veggies" "-f" "${t}")
endforeach()

# Or define tests using the test-drive executable
foreach(t IN LISTS tests)
  add_test(NAME "testdrive_${t}" COMMAND "test_${PROJECT_NAME}-test-drive" "${t}" WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}")
endforeach()
```

:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

### Challenge 2: Debug and fix a failing test.

Take a look at the [4-debugging-a-broken-test/challenge-1 README.md](https://github.com/UCL-ARC/fortran-unit-testing-exercises/episodes/4-debugging-a-broken-test/challenge-1/README.md) in the exercises repository.

:::::::::::::::::::::::::::::::: solution

A solution is provided in [README-solution.md](https://github.com/UCL-ARC/fortran-unit-testing-exercises/episodes/4-debugging-a-broken-test/challenge-1/README-solution.md).

:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
