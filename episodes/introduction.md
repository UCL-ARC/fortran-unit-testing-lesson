---
title: "Introduction"
teaching: 3
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- How can you follow along with this walkthrough?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Can clone the exercises repo for the walkthrough.
- Understand the objectives of this walkthrough.

::::::::::::::::::::::::::::::::::::::::::::::::

## Introduction

This walkthrough aims to...

- Demonstrate the importance of testing Fortran codes with unit tests.
- Show how to write unit tests for Fortran Code using three different frameworks: test-drive, veggies and pFUnit. 
- Show how to integrate these tests with both CMake and FPM build systems.
- Highlight the differences between writing unit tests for parallel and serial Fortran code.

## Putting it into practice

Throughout this walkthrough, we will use a repository containing example exercises written in Fortran - [UCL-ARC/fortran-unit-testing-exercises](https://github.com/UCL-ARC/fortran-unit-testing-exercises).

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

::::::::::::::::::::::::::::::::::::: callout

## pFUnit

To run any of the pFUnit tests, you will need to have pFUnit built and installed locally. This can be done using the bash script provided in the exercises repo [build-pfunit.sh](https://github.com/UCL-ARC/fortran-unit-testing-exercises/build-pfunit.sh).

```bash
./build-pfunit.sh -b
```

Test your installation via 

```bash
./build-pfunit.sh -t
```

:::::::::::::::::::::::::::::::::::::::::::::::
