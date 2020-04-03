---
title: Use LLVM toolchain With Bazel
description: my (painful) experience of trying to use LLVM toolchain with Bazel.
categories:
  - Blog
tags:
  - techtips
---

Since I use Clang and other LLVM tools at work, I decided to migrating to using
the same set of tools for my side projects too. But it turned out that
customizing the cc toolchain (crosstool) of Bazel is very confusing and
time-confusing. There doesn't seem to be a good documentation in this regard.

## Easy part: install the LLVM toolchain
I am on Ubuntu 18.04, so it is very straightfoward to install the LLVM packages.
The stable versions should already on the main Ubuntu Apt sources, but if you
want the most recent/nightly packages, you can find the instructions at
[apt.llvm.org](http://apt.llvm.org/). Just add the deb source, add the Apt key
and install the packages you need.

## Make it work with Bazel
By default, Bazel uses gcc and libstdc++ to build C++ targets. To replace gcc
with clang, it is as straightforward as setting the following two environment
variables (I use Clang 8)
```sh
export CC=clang-8
export CXX=clang++-8
```
But trying to replace libstdc++ with libc++ proves to be much more difficult.
Simply ading `copt=["-stdlib=libc++"]` to the cc targets doesn't seem to work:
Bazel cannot find the libc++ headers. I have asked
[a question on SO](https://stackoverflow.com/questions/57455011/cannot-find-header-when-trying-to-use-libc-with-bazel).
If I ever get an aswer, I will share here too. But otherwise it seems to be a
waste of time to find an answer myself.

There are already some third party implementations of Bazel rules of using Clang,
and potentially libc++. For example, github/envoyproxy/envoy/bazel/toolchains
/configs/clang_libcxx/, github/grailbio/bazel-toolchain. But right now I don't
see much actual need of using libc++ and I would much prefer to not introduce
any further third-party dependency, especially those that I don't know much
about.
