# Workers

Workers of all types throughout buildfarm are responsible for presenting execution roots to operations that they are matched with, fetching content from a CAS, executing those processes, and reporting the outputs and results of executions. Additionally, buildfarm supports some common behaviors across worker types:

* ExecutionPolicies, which allow for explicit and implicit behaviors to control execution.
* A CAS FileCache, which is capable of reading through content for Digests of files or directories, and efficiently presenting those contents based on usage and reference counting, as well as support for cascading into delegate CASs.
* Concurrent pipelined execution of operations, with support for superscalar stages at input fetch and execution.
* Operation exclusivity, preventing the same operation from running through the worker pipeline concurrently.

# Worker Types

## Operation Queue

Operation Queue workers are responsible for taking operations from the Memory OperationQueue service and reporting their contents via external CAS and AC services. Executions are the only driving force for their CAS FileCache.

## Shard

Sharded workers interact with the shard backplane for both execution and CAS presentation. Their CAS FileCache serves a CAS gRPC interface as well as the execution root factory.