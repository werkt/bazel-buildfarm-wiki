# User Guide

Bazel Buildfarm is a service software stack which presents an implementation of the [Remote Execution API](https://github.com/bazelbuild/remote-apis). This means it can be used by any client of that API to retain content ([[ContentAddressableStorage]]), cache ActionResults by a key ([[ActionCache]]), and execute actions asynchronously ([[Execution]]).

This documentation is intended as a comprehensive and ongoing description of its architecture, features, functionality, and operation, as well as a guide to a smoothly running install of the software. Familiarity with the Remote Execution API is expected, and references will be provided to it as needed.

_This wiki is a work in progress, and parts of it may still be under construction._

## [[Architecture]]

## [[General Features]]

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