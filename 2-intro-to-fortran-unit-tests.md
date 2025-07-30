---
title: "Introduction to Unit Testing in Fortran"
teaching:
exercises:
---

:::::::::::::::::::::::::::::::::::::: questions 

- Why is there a lack of unit tests in existing Fortran codes?
- Why do we need to write unit tests for Fortran code?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand the need for unit testing Fortran code.
- Identify Fortran code which is problematic for unit testing.
- Understand steps to make Fortran more unit testable.

::::::::::::::::::::::::::::::::::::::::::::::::

## Why your Fortran may not already be unit tested

Many Fortran codes don't yet have unit tests. This can be for a number of reasons...

- Lack of resources (time and/or money).
    - Funding is earmarked for furthering science.
- The code is not able be unit tested in its current state.
    - Code is written in Fortran 77 or uses outdated paradigms which make unit testing near impossible.
- Lack of skills.
    - Developers don't have the skills to implement unit tests.

## What's the matter with Fortran?

Assuming we have the time, money and the skills to implement some unit tests in a Fortran project, what about the code makes it a challenge to do so?

Various bad practices can make it difficult to unit test Fortran including...

- Global variables
- Large, multipurpose procedures

::::::::::::::::::::::::::::::::::::: challenge 

#### Challenge 1: Identify bad practice for unit testing Fortran

Take a look at [2-intro-to-fortran-unit-tests/challenge-1/challenge](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/2-intro-to-fortran-unit-tests/challenge-1/challenge) in the exercises repository.

:::::::::::::::::::::::::::::::: solution

A solution is provided in [2-intro-to-fortran-unit-tests/challenge-1/solution](https://github.com/UCL-ARC/fortran-unit-testing-exercises/tree/main/episodes/2-intro-to-fortran-unit-tests/challenge-1/solution).

:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::

## How to start improving

Unit tests do not need to be added to a Fortran project in one huge implementation. The best approach is to incrementally add unit tests 

1. Ensure an end-to-end test exists
    - This can be written in a different language if needed. Pytest is often a good choice.
2. Identify a procedure with minimal dependencies 
    - If one can't be found extract one into a procedure
3. Ensure end-to-end test covers this procedure
    - Try to break the procedure and check your test fails as expected.
4. Remove all non-local state in the procedure
5. Check end-to-end test passes
6. Write unit test(s) for procedure
7. Repeat steps 1-6

This incremental approach will, over time, become smoother. As more unit tests exist the code will become easier to unit test as its infrastructure will become more mature and feature rich.

::::::::::::::::::::::: callout

### Working effectively with legacy code

Untested codebase is effectively legacy code. Therefore, a great resource for us is *[Working Effectively with Legacy Code](https://search.worldcat.org/title/660166658)* (Feathers, 2004)

If you don't have time to read the entire book, there is a good summary of the key point in this blog post [The key points of Working Effectively with Legacy Code](https://understandlegacycode.com/blog/key-points-of-working-effectively-with-legacy-code/)

:::::::::::::::::::::::::::::::

## References 

- Michael Feathers (2004). [Working Effectively with Legacy Code](https://search.worldcat.org/title/660166658). Pearson.
