---
title: Use LLVM Toolchain With Bazel Part 2
description: follow up on the previous post on using LLVM toolchain with Bazel.
categories:
  - Blog
tags:
  - techtips
---
In the previous [post]({% post_url 2019-08-11-clang-libcxx-with-bazel %}), I
explained how to build c++ targets using Clang instead of the default compiler
gcc on Ubuntu, but failed to figure how to compile and link against libc++,
libabi++ and libm. I asked the question on StackOverflow, but got no attention
so I guess there isn't a wide use of it. That post now has been removed. Now
that I have figured out a solution, I might just record and share it here.

Last time I was taken down the wrong route and tried to build a [custom Bazel
toolchain](https://docs.bazel.build/versions/master/tutorial/cc-toolchain-config.html),
but that tutorial is rather confusting and prbably outdated. The right approach
is to let
[`cc_configure.bzl`](https://github.com/bazelbuild/bazel/blob/master/tools/cpp/cc_configure.bzl)
figure out how to set up the toolchain. To begin with, we need to add your LLVM
binaries to the system `PATH` environment variable. In this way, Clang will be
able to build with the LLVM headers when you compile with `-stdlib=libc++`. So
in total, at this point you should have the following env variables (assuming
your LLVM binaries are at `/usr/lib/llvm-8/bin`)
```sh
export CC=clang
export CXX=clang++
export PATH=/usr/lib/llvm-8/bin:$PATH
```
Note that in the previous post I set `CC=clang-8` and `CXX=clang++-8` while here
I removed the version suffix. I have seen reports saying that if you use
`clang-8` there is a chance that the build won't be able to find the headers so
I would recommend create symlinks and just use `clang` and `clang++`.

Then we need to pass these environment variables onto Bazel, so in your
`.bazelrc` file, add
```sh
build --action_env=CC
build --action_env=CXX
build --action_env=PATH
build --action_env=CXXFLAGS=-stdlib=libc++
```

With all these, the build should be able to use the LLVM headers and tried to
link against libc++, libabi++ and libm. But the current toolchain does not know
where to find them yet. We need to add the following three build settings to
make it work.
```sh
build --action_env=BAZEL_CXXOPTS=-stdlib=libc++
build --action_env=BAZEL_LINKLIBS=-lc++:-lc++abi:-lm
build --host_linkopt=-fuse-ld=lld
```
The first two lines are used by `cc_configure.bzl` to locate the libraries and
the last sets the linker to `lld` so that linkage with LLVM libraries can be
done properly.

That should be it.

