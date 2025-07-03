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

## Dependencies

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
TODO: Add output from tetsing pfunit and implement testing pFUnit.
```

:::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
