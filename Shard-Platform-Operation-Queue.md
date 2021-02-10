# Shard Platform Operation Queue
This section discusses the purpose of the Shard Platform Operation Queue, and how it can be customized depending on the type of operations you wish to support and efficiently distribute among workers.

Some time after an Action execute request occurs, the longrunning operation it corresponds to will enter the QUEUED state, and will receive an update to that effect on the operation response stream. An operation in the QUEUED state is present in an Operation Queue, which holds the operations in sequence until a worker is available to execute it.

## Working with different platform requirements 
Some operations' Actions may have specific platform requirements in order to execute.
Likewise, specific workers may only want to take on work that they deem eligible.
To solve this, the operation queue can be customized to divide work into separate provisioned queues so that specific workers can choose which queue to read from.  

Provision queues are intended to represent particular operations that should only be processed by particular workers. An example use case for this would be to have two dedicated provision queues for CPU and GPU operations. CPU/GPU requirements would be determined through the [remote api's command platform properties](https://github.com/bazelbuild/remote-apis/blob/86c040d03101654a949539151d32e22dfea30d62/build/bazel/remote/execution/v2/remote_execution.proto#L595). We designate provision queues to have a set of "required provisions" (which match the platform properties). This allows the scheduler to distribute operations by their properties and allows workers to dequeue from particular queues.

If your configuration file does not specify any provisioned queues, buildfarm will automatically provide a default queue with full eligibility on all operations.
This will ensure the expected behavior for the paradigm in which all work is put on the same queue.

### Server Example

In this example the scheduler declares a GPU queue and CPU queue:
```
redis_shard_backplane_config: {
  provisioned_queues: {
    queues: [
      {
        name: "gpu_queue"
        platform: {
          properties: [{name: "gpu" value: "1"}]
        }
      },
      {
          name: "cpu_queue"
      }
    ]
  }
}
```

### Worker Example

Queues are defined similarly on Workers:

```
redis_shard_backplane_config: {
  provisioned_queues: {
    queues: [
      {
        name: "gpu_queue"
        platform: {
          properties: [{name: "gpu" value: "1"}]
        }
      },
      {
          name: "cpu_queue"
      }
    ]
  }
}
```

In addition to the queue declaration, the worker must specify from which queue tasks are obtained (otherwise the worker process will refuse to start):

```
omit_from_cas: true

dequeue_match_settings: {
  platform: {
    properties: [{name: "gpu" value: "1"}]
  }
}
```

Note: in both of these examples we specify the CPU queue last.
Since operation queues consist of multiple provisioned queues in which the order dictates the eligibility and placement of operations, it is recommended to have a final provision queue with no actual platform requirements. This ensures that all operations are eligible for the final queue.

Note: make sure that all workers can communicate with each other before trying these examples

Note: the parameter `omit_from_cas` is optional, and is specified here to ensure our gpu workers do not act as a CAS shard.

### Bazel Perspective

Bazel targets can pass these platform properties to buildfarm via [exec_properties](https://docs.bazel.build/versions/master/be/common-definitions.html#common.exec_properties).
Here is for example how to run a remote build for the GPU queue example above:

```shell
bazel build --remote_executor=grpc://server:port --remote_default_exec_properties=gpu=1 //...
```
