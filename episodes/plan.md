# Plan for this walkthrough

This file contains a rough plan for the walkthrough and will be removed from the final product.

## Objectives for the entire lesson/walkthrough

- [ ] Explain the benefit of unit tests on top of integration/e2e tests.
- [ ] Understand the key anatomy of a Fortran unit test (test-suite boiler-plate, input types, calls to functions under test, assertions, etc).
- [ ] Able to identify Fortran code which is problematic for unit testing.
- [ ] Able to write a Fortran unit test for a procedure with...
   - [ ] test-drive
   - [ ] veggies
   - [ ] pFUnit
- [ ] Able to add unit tests into the build system of a Fortran project using...
   - [ ] CMake with...
      - [ ] test-drive
      - [ ] veggies
      - [ ] pFUnit
   - [ ] FPM with...
      - [ ] test-drive
      - [ ] veggies
      - [ ] pFUnit
- Able to understand the failure output of a Fortran unit test written in...
   - [ ] test-drive
   - [ ] veggies
   - [ ] pFUnit

## Episodes

### 1. Getting the walkthrough src code

Objectives
- [ ] Can clone the provided repo for the walkthrough.

### 2. Intro to unit tests

Objectives
- [ ] Able to define the key aspects of a unit test (isolated, testing minimal functionality, fast, etc).
- [ ] Understand the key anatomy of a unit test in any language (given-when-then, starting state, assertions, fixtures, expected outputs, etc).
- [ ] Can explain the benefit of unit tests on top of integration/e2e tests.
    - Could use a narrative to highlight how an e2e failure does not show where in the code the bug exists.
        - A new member of the team is adding a feature and some existing e2e tests start failing.
        - They don't realize but this due to their edit of an intent in struct/type (`intent(in)` is not respected for these).
        - A unit test of this procedure would have highlighted that it was this function that had been broken.
- [ ] Understand when to run unit tests.

### 3. Intro to unit testing in Fortran

Objectives
- [ ] Understand the need for unit testing Fortran code.
- [ ] Able to identify Fortran code which is problematic for unit testing.

### 4. Define the anatomy of a unit test

Objectives
- [ ] Understand the anatomy of a unit test in any language (given-when-then, starting state, assertions, fixtures, expected outputs, etc).

### 5. Fortran unit test syntax

Objectives
- [ ] Able to write a unit test for a Fortran procedure with test-drive, veggies and/or pFUnit.
- [ ] Understand the similarities between each framework and where they differ.

### 6. Debugging a broken test

Objectives
- [ ] Can build and edit the provided repo for the walkthrough.
- [ ] Capable of determining where in the tests or src code the failure is occurring.
- [ ] Can fix a failing test.
- [ ] Understand where a test is defined in the build system (CMake and FPM).
- [ ] 

### 7. Adding a test for an untested feature

Objectives
- [ ] Able to add a test for a completely untested feature/procedure.

### 8. Testing parallel code

Objectives
- [ ] Understand what is different when testing parallel vs serial code.
