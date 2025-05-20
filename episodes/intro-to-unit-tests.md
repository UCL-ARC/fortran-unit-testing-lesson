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

## Challenge 1: Write a unit test in sudo code.

Assuming you have a functio `reverse_array` which reverses the order of an allocated array. Write a unit test in sudo code for `reverse_array`.

:::::::::::::::::::::::: solution 
 
```md
# Given 
Allocate the input array input_array
Allocate the expected output array expected_output_array
Fill input_array with (1,2,3,4)
Fill expected_output_array with (4,3,2,1)

# When
Call reverse_array with input_array

# Then
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
