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

# Pipelines

A pipeline handles operations as they arrive and are processed on a worker. Each stage of the pipeline performs its task on an operation, and holds that task until the subsequent stage can take over. This creates backpressure to mitigate risk and limit resource consumption. Since a worker only takes on enough work to exhaust the most limited resource at a time (CPU/IO/Bandwidth), losing that worker due to unforeseen failures is not disruptive to the rest of the cluster. Keeping work in each stage holds resources for an operation as well, so preventing operations from piling up in an earlier stage due to a longer running later stage reduces the overall resource footprint required for the worker.

## Stages

[[/images/WorkerStages.png|Stages]]

Stages have access to a WorkerContext provided to them by their Worker implementation (OperationQueue or Shard) which is used for all activity common across Worker types. Each stage must claim the subsequent stage before submitting it for processing. This allows measurement of processing time and latency for each operation per stage without any interleaving.

Superscalar stages have a configurable number of slots for their activity. Claims on these stages block until they are full, and have a number of slots to claim exclusively for activity.

### Match

The Match stage is responsible for dequeuing an operation from the Ready-To-Run queue. This operation is dequeued as a QueueEntry which contains the ExecuteEntry and a Digest for the transformed QueuedOperation. The ExecuteEntry contains a Platform definition which must match the worker's provided platform manifest in order to proceed. A rejected QueueEntry for this reason will be reinserted into the Ready-To-Run queue.

The Match stage is unique in that it claims a slot in the Input Fetch stage prior to its iteration. This removes a polling requirement for the active operation present in other stages while waiting to feed the interstage, reducing the stage's complexity.

### Input Fetch

Input Fetch is a superscalar stage responsible for downloading the QueuedOperation from the CAS, and creating the execution directory for the Operation. This is the worker ingress bandwidth, and likely the disk IO write, consuming stage. Its configured concurrency is available in the worker config as `input_fetch_stage_width`.

### Execution

Execution is a superscalar stage which initiates operation executions, applying any ExecutionPolicies. The operation transitions to the EXECUTING state when it reaches this stage. After spawning the process, it intercepts writes to stdout and stderr, and will terminate the process if it runs longer than its Action specified timeout.

### Report Result

The Report Result stage injects any outputs from the operation into the CAS, and populates the ActionResult from from the results of the execution. It can inject into the ActionCache for cacheable actions, and can record an action in the blacklist if it violates output policy. The operation transitions to COMPLETED state after the outputs are recorded. After this stage is complete, the execution directory is destroyed, and the Operation exits the worker.