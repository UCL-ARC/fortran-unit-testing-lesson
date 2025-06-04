---
title: "Introduction"
teaching: 3
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- How can you follow along with this walkthrough?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Can clone the provided repo for the walkthrough.
- Understand the objectives of this walkthrough.

::::::::::::::::::::::::::::::::::::::::::::::::

## Introduction

This walkthrough aims to...
- Demonstrate the importance of testing Fortran codes with unit tests.
- Show how to write unit tests for Fortran Code using three different frameworks: test-drive, veggies and pFUnit. 
- Show how to integrate these tests with both CMake and FPM build systems.
- Highlight the differences between writing unit tests for parallel and serial Fortran code.

To achieve these aims we will use a repository containing some Fortran code with an existing test setup so we can try some of what we learn.

::::::::::::::::::::::::::::::::::::: callout

To run any of the pFUnit tests, you will need to have pFUnit built locally. Please follow the [instructions provided by pFUnit](https://github.com/Goddard-Fortran-Ecosystem/pFUnit?tab=readme-ov-file#building-and-installing-pfunit) to set this up.

:::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

Switch to terminal and clone the repo [UCL-ARC/fortran-tooling](https://github.com/UCL-ARC/fortran-tooling.git). 

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge 

## Challenge 1: Can you access the exercises repo?

Are you able to clone the exercises repository?

```sh
git clone https://github.com/UCL-ARC/fortran-tooling.git
```

:::::::::::::::::::::::: solution 

## Output
 
```output
Cloning into 'fortran-tooling'...
remote: Enumerating objects: 955, done.
remote: Counting objects: 100% (419/419), done.
remote: Compressing objects: 100% (219/219), done.
remote: Total 955 (delta 242), reused 344 (delta 199), pack-reused 536 (from 1)
Receiving objects: 100% (955/955), 192.00 KiB | 3.62 MiB/s, done.
Resolving deltas: 100% (498/498), done.
```

:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::
