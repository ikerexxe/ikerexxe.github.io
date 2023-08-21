---
layout:     post
title:      "Unit test framework selection for shadow"
date:       2023-08-21 12:00:00 +0100
categories: idm
---

# Introduction

Currently, [shadow](https://github.com/shadow-maint/shadow) doesn't use any unit testing framework. The objective of this post is to compare several frameworks and select the best one to increase the overall quality of the project.

## Considered characteristics

* Open source license

* Well maintained and active project

* Good documentation

* Packaged in distributions included in shadow's CI: Alpine, Debian and Fedora

* Easy to mock functions

* Used in projects from the same domain (IdM)

## Frameworks under consideration

* [CppUTest](http://cpputest.github.io/manual.html) and [CppUMock](http://cpputest.github.io/mocking_manual.html): CppUTest is a C/C++ based unit xUnit test framework for unit testing and for test-driving your code. It is written in C++ but is used in C and C++ projects.

* [Unity](https://www.throwtheswitch.org/unity) and [CMock](https://www.throwtheswitch.org/cmock): Unity is a lightweight, portable, and flexible unit testing framework for C and C++. It's designed to make the process of creating and running tests – both simple and complex – as easy as possible.

* [Check](https://libcheck.github.io/check/): Check is a unit testing framework for C. It features a simple interface for defining unit tests, putting little in the way of the developer. Tests are run in a separate address space, so Check can catch both assertion failures and code errors that cause segmentation faults or other signals.

* [CMocka](https://cmocka.org/): CMocka is a test framework for C with support for mock objects. It only requires the standard C library, works on a range of computing platforms and with different compilers.

* [Criterion](https://criterion.readthedocs.io/en/master/): Criterion is a non-intrusive cross-platform C unit testing framework that is simple to use and setup.

# Comparison

Let's start with CppUTest. Its latest release was three years ago, and the last commit two months ago <sup>[3](#references)</sup>, so the project is active. The documentation looks very complete, with lots of examples and explaining how to set up the environment. The package is included in the most common distributions. The mocking seems easy to get done. Indeed, it looks very similar to [googlemock](https://github.com/google/googletest). Several projects in the embedded world use it: SparkPost, the QP/C Real-Time Embedded Framework, bugfree_robot... But I have been unable to find any in the IdM domain.

Unity's latest release was two years ago, and the last commit was done a few days ago <sup>[4](#references)</sup>. Thus, the projects is under active development. The documentation is fine, not as complete as CppUTest but it contains enough information to set everything up. The package isn't available in any of the distributions. and you need to install it from source. Some projects in the embedded world, like PlatformIO, use this framework. The framework is especially designed for embedded software, which may not be perfect for our use cases.

Check's latest release and last commit is from two years ago <sup>[5](#references)</sup>. Although the documentation contains some examples and enough information to set everything up, it does not explain how to mock functions. The package is included in Alpine, Debian and Fedora. A quick search doesn't render any result for projects from the IdM domain.

CMocka's latest version was released six months ago, and the last commit was just a few days ago <sup>[6](#references)</sup>. The documentation looks very complete, with some examples, explaining how to set up the environment and with a well documented API. The package is included in the most common distributions. The mocking is easy to get done. It is used by projects like Samba, SSSD, libssh, etc. All of them belong to the IdM domain. Moreover, CMocka allows to handle exception signals (i.e. SIGSEGV, SIGILL) and memory errors detection (i.e leaks, overflows, underflows).

Criterion's latest release and the last commit were three months ago <sup>[7](#references)</sup>. The documentation doesn't explain how to mock or how to set up the environment with Makefile. The package isn't available in Alpine and Fedora repositories. A quick search doesn't render any result for projects from the IdM domain.

## Summary table

| Framework | License            | Well-maintained | Documentation | Included in distros | Easy to mock | Used in IdM projects |
| --------- | ------------------ | --------------- | ------------- | ------------------- | ------------ | -------------------- |
| CppUTest  | BSD 3-Clause       | Yes             | Yes           | Yes                 | Yes          | No                   |
| Unity     | MIT                | Yes             | No            | No                  | Yes          | No                   |
| Check     | LGPL-2.1           | No              | No            | Yes                 | ?            | No                   |
| CMocka    | Apache License 2.0 | Yes             | Yes           | Yes                 | Yes          | Yes                  |
| Criterion | MIT                | Yes             | No            | No                  | ?            | No                   |

# Conclusion

Unity, Check and Criterion should be excluded since they are either not maintained, or the documentation is not complete, or the packages are not included in the distributions. That leaves us with CppUTest and **CMocka**, and this last one **seems like the best suited for shadow**. The feature that has tipped the balance has been the fact that it is used to test other IdM projects. In addition, exception handling and memory error detection are also very useful features.

# References

1. [Unit Testing Frameworks for C: Comparison](https://stackoverflow.com/questions/1468110/unit-testing-frameworks-for-c-comparison)
2. [List of unit testing frameworks](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks)
3. [CppUTest development repository](https://github.com/cpputest/cpputest)
4. [Unity development repository](https://github.com/ThrowTheSwitch/Unity.git)
5. [Check development repository](https://github.com/libcheck/check)
6. [CMocka development repository](https://git.cryptomilk.org/projects/cmocka.git/)
7. [Criterion development repository](https://github.com/Snaipe/Criterion/)
