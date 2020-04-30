# Quick Start

This section will quickly describe how to use buildfarm for remote execution with a minimal configuration - a single memory instance, with a host-colocated worker that can execute a single process at a time - via a bazel invocation on a small workspace.

Let's start with a bazel WORKSPACE with a single file to compile into an executable, in a directory named `my_main`:

`main.cc`:
```
#include <iostream>

int
main( int argc, char *argv[] )
{
  std::cout << "Hello, World!" << std::endl;
}
```

`BUILD`:
```
cc_binary(
    name = "main",
    srcs = ["main.cc"],
)
```

And an empty WORKSPACE file.

As a test, verify that `bazel run :main` builds your main program and runs it, and prints `Hello, World!`. This will ensure that you have properly installed bazel and a C++ compiler, and have a working target before moving on to remote execution.

This tutorial assumes that you have a bazel binary in your path and you are in the root of your buildfarm clone/release, and has been tested to work with bash on linux.

From a single terminal, run `bazel run src/main/java/build/buildfarm:buildfarm-server $PWD/examples/server.config.example`

From another terminal, run `bazel run src/main/java/build/buildfarm:buildfarm-operationqueue-worker $PWD/examples/worker.config.example`

From a third terminal, in your `my_main` directory, run `bazel run --remote_executor=localhost:8980 :main`

If your build once again printed build statistics and `Hello, World!`, congratulations, you just build something through remote execution!