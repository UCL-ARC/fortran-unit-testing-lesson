---
title: "Introduction to Unit Testing"
teaching: 
exercises: 
---

:::::::::::::::::::::::::::::::::::::: questions 

- What is unit testing?
- Why do we need unit tests?
- What makes src code difficult to unit test?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- [ ] Able to define the key aspects of a unit test (isolated, testing minimal functionality, fast, etc).
    - *Assessment:* Identify aspects of a test which does not follow all of these.
- [ ] Understand the key anatomy of a unit test in any language (given-when-then, starting state, assertions, fixtures, expected outputs, etc).
    - *Assessment:* Write a unit test in sudo code.
- [ ] Can explain the benefit of unit tests on top of integration/e2e tests.
    - Could use a narrative to highlight how an e2e failure does not show where in the code the bug exists.
        - A new member of the team is adding a feature and some existing e2e tests start failing.
        - They don't realize but this due to their edit of an intent in struct/type (`intent(in)` is not respected for these).
        - A unit test of this procedure would have highlighted that it was this function that had been broken.
- [ ] Understand when to run unit tests.

::::::::::::::::::::::::::::::::::::::::::::::::

## What is unit testing?

Unit testing is a way of verifying the validity of a code base by testing its smallest individual components, or **units**. Unit testing is essential as *"If the parts don't work by themselves, they probably won't work well together"* [Reference, The pragmatic programmer].

Several key aspects define a unit test. They should be...
- **Isolated** - Does not rely on any other unit of code within the repository.
- **Minimal** - Tests only one unit of code.
- **Fast** - Run on the scale of ms or s. 

There are other forms of testing, such as integration testing in which two or more units of a code base are tested to verify that they work together, or that they are **integrated** correctly. However, today we are focusing on unit tests as it is often the case that many of these larger tests are written using the same test tools and frameworks, hence we will make progress with both by starting with unit testing.

### What does a unit test look like?

All unit tests tend to follow the same pattern of Given-When-Then.
- **Given** we are in some specific starting state
    - Units of code almost always have some inputs. These inputs may be scalars to be passed into a function, but they may also be an external dependency such as a database, file or array which must be allocated.
    - This database, file or array memory must exist before the unit can be tested. Hence, we must set up this state in advance of calling our unit under test.
- **When** we carry out a specific action
    - This is the step in which we call the unit of code to be tested, such as a call to a function or subroutine.
    - We should limit the number of actions being performed here to ensure it is easy to determine which unit is failing in the event that a test fails.
- **Then** some specific event/outcome will have occurred.
    - Once we have called out unit of code to be tested we must check that what we expected to happen did indeed happen.
    - This could mean comparing a scalar or vector quantity returned from the called unit against some expected value. However, it could be something more complex such as validating the contents of a database or outputted file.

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge 1: Write a unit test in sudo code.

Assuming you have a function `reverse_array` which reverses the order of an allocated array. Write a unit test in sudo code for `reverse_array`.

:::::::::::::::::::::::: solution 
 
 ```
! Given
Allocate the input array input_array
Fill input_array with (1,2,3,4)
Allocate the expected output array expected_output_array
Fill expected_output_array with (4,3,2,1)

! When
Call reverse_array with input_array

! Then
for each element in input_array 
    Assert that the corrosponding element of expected_output_array matches that of input_array
```

:::::::::::::::::::::::::::::::::

### When should unit tests be run?

The benefit of unit tests are 

### Do we really need unit tests?

Yes! Let me explain.

You may be thinking that you don't require unit tests as you already have some well-defined end-to-end test cases which demonstrate that your code base works as expected. However, consider the case where this end-to-end test begins to fail. The message for this failure is likely to be something along the lines of
```
Expected my_special_number to be 1.234 but got 5.678
```
If you have a comprehensive understanding of your code, perhaps this is all you need. However, assuming the newest feature that caused this failure was not written by you, it's going to be difficult to identify what is going wrong without some lengthy debugging. Conversely, imagine a situation where the developer of this new change added unit tests for their new code. When running these unit tests, you may see something like
```
test_populate_arrays Failed: Expected 0 for index 1 but got 1
```
This is so much clearer. We immediately have an idea of what could be going wrong and the unit test itself will help us determine the problematic code to investigate.


::::::::::::::::::::::::::::::::::::: challenge 

### Challenge 2: Unit test bad practices

Take a look at this partial test written in Fortran where we are testing some code which performs transforms on matrices.
1. Can you identify the aspects of this test which make it a bad unit test?
2. Can you rewrite it to make it an exemplar unit test?

```f90
...
! Given
expected_matrix(1,1:num_particles) = [2.0, 4.0, 6.0, 8.0, 10.0]
expected_matrix(2,1:num_particles) = [12.0, 14.0, 16.0, 18.0, 20.0]
expected_matrix(3,1:num_particles) = [22.0, 24.0, 26.0, 28.0, 30.0]

! When
call apply_transform(transform_file_name, actual_matrix, num_particles)
call scale_matrix(2.0, actual_matrix, num_particles)

! Then
@assertEqual(expected_matrix(:, 1:num_particles), actual_matrix(:, 1:num_particles), "Unexpected values for actual matrix")
```

:::::::::::::::::::::::: solution 

There are several issues with this test.
1. For this test to work we need the transform file to exist. However, we are not checking if this file exists. There are multiple ways to improve this. For example, prior to calling the src code, we could...
    - add a simple check that the file exists before calling the src code.
        ```f90
        ...
        ! Open the file by name and verify it was successful
        open (unit=transform_file_unit, &
            file=transform_file_name, &
            status=status,            &
            IOSTAT=iostat)
        @assertEqual(status, 0, "Expected transform file not found")
        ...
        ```
    - create this file prior to calling the src code
        ```f90
        ...
        ! Open the file by name and verify it was successful
        open (unit=transform_file_unit, &
            file=transform_file_name, &
            status=status,            &
            IOSTAT=iostat)
        @assertEqual(status, 0, "Expected transform file not found")
        
        ! Write the tranform to the file
        transform(1,:) = [1, 0, 0]
        transform(2,:) = [0, 1, 0]
        transform(3,:) = [0, 0, 1]
        do i=1,3
            write(transform_file_unit,'(i2,i2,i2)') transform(1,i), transform(2,i), transform(3,i)
        end do

        close(transform_file_unit)
        ...
        ```
2. Within the test we are calling two functions in our `When` section. To make sure out test is **Isolated** and **Minimal**, we should be calling only one src procedure before checking assertions. We could improve this by...
    - Add an assertion after every subroutine call
        ```f90
        ...
        ! Given
        expected_matrix_after_transform(1,1:num_particles) = [1.0, 2.0, 3.0, 4.0, 5.0]
        expected_matrix_after_transform(2,1:num_particles) = [6.0, 7.0, 8.0, 9.0, 10.0]
        expected_matrix_after_transform(3,1:num_particles) = [11.0, 12.0, 13.0, 14.0, 15.0]

        expected_matrix_after_scaling(1,1:num_particles) = [2.0, 4.0, 6.0, 8.0, 10.0]
        expected_matrix_after_scaling(2,1:num_particles) = [12.0, 14.0, 16.0, 18.0, 20.0]
        expected_matrix_after_scaling(3,1:num_particles) = [22.0, 24.0, 26.0, 28.0, 30.0]

        ! When 1
        call apply_transform(transform_file_name, actual_matrix, num_particles)

        ! Then 1
        @assertEqual(expected_matrix_after_transform(:, 1:num_particles), actual_matrix(:, 1:num_particles), "Unexpected values for actual matrix after transform")

        ! When 2
        call scale_matrix(2.0, actual_matrix, num_particles)

        ! Then 2
        @assertEqual(expected_matrix_after_scaling(:, 1:num_particles), actual_matrix(:, 1:num_particles), "Unexpected values for actual matrix after scaling")
        ```
    - Split into two tests
        ```f90
        ...
        ! Test 1
        ! Given
        expected_matrix_after_transform(1,1:num_particles) = [1.0, 2.0, 3.0, 4.0, 5.0]
        expected_matrix_after_transform(2,1:num_particles) = [6.0, 7.0, 8.0, 9.0, 10.0]
        expected_matrix_after_transform(3,1:num_particles) = [11.0, 12.0, 13.0, 14.0, 15.0]

        ! When
        call apply_transform(transform_file_name, actual_matrix, num_particles)

        ! Then
        @assertEqual(expected_matrix_after_transform(:, 1:num_particles), actual_matrix(:, 1:num_particles), "Unexpected values for actual matrix after transform")
        ...
        ! Test 2
        ! Given
        actual_matrix(1,1:num_particles) = [1.0, 2.0, 3.0, 4.0, 5.0]
        actual_matrix(2,1:num_particles) = [6.0, 7.0, 8.0, 9.0, 10.0]
        actual_matrix(3,1:num_particles) = [11.0, 12.0, 13.0, 14.0, 15.0]

        expected_matrix_after_scaling(1,1:num_particles) = [2.0, 4.0, 6.0, 8.0, 10.0]
        expected_matrix_after_scaling(2,1:num_particles) = [12.0, 14.0, 16.0, 18.0, 20.0]
        expected_matrix_after_scaling(3,1:num_particles) = [22.0, 24.0, 26.0, 28.0, 30.0]
        ! When
        call scale_matrix(2.0, actual_matrix, num_particles)

        ! Then
        @assertEqual(expected_matrix_after_scaling(:, 1:num_particles), actual_matrix(:, 1:num_particles), "Unexpected values for actual matrix after scaling")
        ```
:::::::::::::::::::::::::::::::::