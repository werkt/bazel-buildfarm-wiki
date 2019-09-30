# User Guide

Bazel Buildfarm is a service software stack which presents an implementation of the Remote Execution API. This means it can be used by any client of that API to retain content (ContentAddressableStorage), cache ActionResults by a key (ActionCache), and execute actions asynchronously (Execution).

Buildfarm uses modular instances, where an instance is associated with one concrete type that governs its behavior and functionality. The Remote Execution API uses instance names to identify every request made, which allows Buildfarm instances to represent partitions of resources. One server may support many different instances, each with their own name, and each instance will have its own type.

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

## General Features

Buildfarm has endeavored to support a wide variety of features implied or mandated by the Remote Execution API, including those currently not in use or worked around by bazel or other clients.

Most notably, buildfarm has universal support for:

* configurable instances with specific instance types
* progressive and flow controlled CAS reads and writes
* pluggable external CAS endpoints
* RequestMetadata behavior attribution

## Instance Types

### Memory

This is a reference implementation of the Remote Execution API which provides an in-memory CAS, AC, and OperationQueue. It has no persistent retention of its state internally, but can be configured to use an external gRPC endpoint for CAS operations, allowing it to act as a proxy. Instances of this type cannot share operation information across multiple servers/hosts. It presents a matching interface for workers via an OperationQueue service definition, which provides blocking queue `take` functionality, as well as a `put` for results, and it maintains watchdogs for all outstanding operations, with expiration resulting in reentrance to the queue, assuming that all input preconditions are still met at that time.

### Shard

The shard instance type is a frontend for a set of common backplane operations, allowing for wide distribution of retention and execution. The backplane serves as a registry of shard workers, the storage for the AC, the various queues and event monitors used in Operation processing, and the CAS index.

The only current backplane implementation uses redis, but the backplane interface is strongly decoupled from its usage, and any single or composite communication layer may be used to satisfy its requirements.

Sharded instances select arbitrary members of the Workers set for CAS writes, and reference the backplane CAS index for CAS reads, selecting any of a set of shards that advertise content to retrieve it. These shards can be any gRPC CAS compatible service endpoint.

Executions on shard are transformed into a worker queue through a processor built into the instance. A server which runs the instance (configurably) participates in this pool, reducing load on a directly connected service to the client. This allows an entire cluster of servers to participate evenly in populating the worker queue with heavyweight operation definitions, even if a client is only communicating with a single host.

## Common Worker Concepts

Workers of all types throughout buildfarm are responsible for presenting execution roots to operations that they are matched with, fetching content from a CAS, executing those processes, and reporting the outputs and results of executions. Additionally, buildfarm supports some common behaviors across worker types:

* ExecutionPolicies, which allow for explicit and implicit behaviors to control execution.
* A CAS FileCache, which is capable of reading through content for Digests of files or directories, and efficiently presenting those contents based on usage and reference counting, as well as support for cascading into delegate CASs.
* Concurrent pipelined execution of operations, with support for superscalar stages at input fetch and execution.
* Operation exclusivity, preventing the same operation from running through the worker pipeline concurrently.

## Worker Types

### Operation Queue

Operation Queue workers are responsible for taking operations from the Memory OperationQueue service and reporting their contents via external CAS and AC services. Executions are the only driving force for their CAS FileCache.

### Shard

Sharded workers interact with the shard backplane for both execution and CAS presentation. Their CAS FileCache serves a CAS gRPC interface as well as the execution root factory.