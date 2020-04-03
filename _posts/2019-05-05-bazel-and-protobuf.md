---
title: Use Bazel and Protobuf
description: use Bazel and Protobuf for your own projects
categories:
  - English 
tags:
  - techtips
---

This is a pretty long overdue post that I wanted to make a while back when I
first tried to use Bazel and Protobuf for my personal project outside Google. I
have been pretty happy with the Google-internal version of Bazel and Protobuf at
work so I decided to use them for my person projects too. Turned out that you
need to spend some effort in making them work outside Google. Here is how you
can do it.

**Note** that since it has been a while since I set them up for my personal
projects, I might have forgotten some important details and some might have even
changed. So any comments are appreciated.

**Note** also that this article does not explain why you should use Bazel or
Protobuf, even though I would recommend the two tools over others. For Bazel I
have used plain "Makefile" and "CMake" for C/C++ projects before and Bazel is
just much better. For Jave projects I have used Maven before, which for Java is
probably a very good option, but for projects that involve other languages,
Bazel is the better one. I have also used Thrift extensively before. Compared to
Protobuf it is as good, mostly because of reasons like easy to use and quality
of documentation.

## Set up Bazel

Installing Bazel is pretty straightforward. Just follow the instructions
[here](https://docs.bazel.build/versions/master/install-ubuntu.html). In order
to use Bazel for your project, you need to have file name `WORKSPACE` in the
root directory of your project. At the minimum, it defines the name of your
project, but it does a lot more than that, as we will see later when we tried to
make it work with Protobuf. 

At each "package" of the project (directories that contain your build targets),
there need to a `BUILD` file that defines the targets. They can be libraries,
binaries or unit tests in various languages. That's it. See [this
documentation](https://docs.bazel.build/versions/master/command-line-reference.html)
for how to run Bazel.

## Set up Protobuf

You can probably easily find tutorials about how to use Protobuf in a standalone
fashion. For example the [official tutorial for
c++](https://developers.google.com/protocol-buffers/docs/cpptutorial). But what
I (most developers) would want is better incorporation with projects, that is, I
don't want to type `protoc` every time I build some target that depends on the
proto messages. When I type `bazel test/run/build`, `protoc` should be invoked
automatically as part of the build process. So here is how to do it with Bazel.
In your `WORKSPACE` file, add the following:

```python

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive") 

http_archive(
  name = "bazel_skylib",
  sha256 = "bbccf674aa441c266df9894182d80de104cabd19be98be002f6d478aaa31574d",
  strip_prefix = "bazel-skylib-2169ae1c374aab4a09aa90e65efe1a3aad4e279b",
  urls = ["https://github.com/bazelbuild/bazel-skylib/archive/2169ae1c374aab4a09aa90e65efe1a3aad4e279b.tar.gz"],
)

local_repository(
  name = "com_google_protobuf",
  path = "<PATH TO PROTOBUF FOLDER>",
)

load("@com_google_protobuf//:protobuf_deps.bzl", "protobuf_deps")

protobuf_deps()

http_archive(
  name = "six_archive",
  build_file = "@//:six.BUILD",
  sha256 = "105f8d68616f8248e24bf0e9372ef04d3cc10104f1980f54d57b2ce73a5ad56a",
  urls = ["https://pypi.python.org/packages/source/s/six/six-1.10.0.tar.gz#md5=34eed507548117b2ab523ab14b2f8b55"],
)

bind(
  name = "six",
  actual = "@six_archive//:six",
)
```

Here are some explanations:
  - You need to load `http_archive` because it is no longer in the standard
    Bazel function definitions. Instead it has been moved to `bazel_tools`.

  - In order to build `proto` targets as defined in `BUILD` files, Bazel looks
    for the binary `@com_google_protobuf//:protoc`, so the external project
    `@come_google_protobuf` needs to be defined in the `WORKSPACE` file.  You
    may define that project using `http_archive`, but I cloned the Protobuf
    project from GitHub and defined it using `local_repository`. Perhaps it is
    easier in this way to keep the binary up to date and you don't have to worry
    about all the settings like `sha256` and `strip_prefix` etc.

  - To make use of Protobuf in your project, you also need to add the other
    definitions since they are required for build `protoc`. It seems they need
    to be defined in the project where Bazel commands are invoked, even though
    they have already been defined the Protobuf project. So by trial and error,
    I found the minimal set of definitions you need to add.

  - You also need to copy over the `six.BUILD` file over from the Protobuf
    project and put it in your project root directory as it is needed for the
    `six_archive` dependency. The content of the file is

```python
genrule(
  name = "copy_six",
  srcs = ["six-1.10.0/six.py"],
  outs = ["six.py"],
  cmd = "cp $< $(@)",
)

py_library(
  name = "six",
  srcs = ["six.py"],
  srcs_version = "PY2AND3",
  visibility = ["//visibility:public"],
)
```

## Python Issues

I also need to use a special version of Python (a virtual env) for my project.
How to do it with Bazel? The solution I use is to define a custom Python
runtime. Add the following to the `BUILD` file in the project root directory

```python
py_runtime(
  name = "<RUNTIME NAME>",
  files = [],
  interpreter_path = "<PATH TO YOUR PYTHON INTERPRETOR>",
  visibility = ["//visibility:public"],
)
```

Then when you run Bazel command, you can use that runtime in the following way
```sh
bazel test --python_top=//:<RUNTIME NAME> ...
```

Another issue is to use proto objects in Python. To do so, you need to wrap the
`proto_library` target in a `py_proto_library` target, which needs to be loaded
from `@com_google_protobuf` using

```python
load("@com_google_protobuf//:protobuf.bzl", "py_proto_library")
```

Every time you use Protobuf Python API's, you need to include `@com_google_protobuf//:protobuf_python` in the build target. An example is more than a thousan words so here is a minimal example 

```python
load("@com_google_protobuf//:protobuf.bzl", "py_proto_library")

proto_library(
  name = "A_proto",
  srcs = ["A.proto"],
  deps = [
    "@com_google_protobuf//:timestamp_proto",
  ],
)

py_proto_library(
  name = "A_proto_py_pb2",
  srcs = ["A.proto"],
  srcs_version = "PY2AND3",
  deps = [
    "@com_google_protobuf//:protobuf_python",
  ],
)

py_test(
  name = "A_test",
  srcs = ["A_test.py"],
  deps = [
    ":B_lib",
    ":A_proto_py_pb2",
    ":C_lib",
    "@com_google_protobuf//:protobuf_python",
  ],
)
```

## CLion Integration
CLion has Bazel and Protobuf plugins, named "Bazel" and "Protobuf support",
respectively. They are pretty good, especially the Bazel one, even though there
is still a lot of things that can be improved. I initially tried them with
PyCharm. They actually don't quite work (I forgot the error message), so I had
to switch to CLion, which also supports Python.
