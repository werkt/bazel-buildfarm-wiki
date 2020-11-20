Buildfarm supports the following [execution properties](https://docs.bazel.build/versions/master/be/common-definitions.html#common.exec_properties).  
Custom properties can also be added to buildfarm's configuration in order to facilitate queue matching (see [Platform Queues](https://github.com/bazelbuild/bazel-buildfarm/wiki/Shard-Platform-Operation-Queue)).

### `min-cores`
**description:** the minimum number of cores needed by an action.  Should be set to >= 1

### `max-cores`
**description:** the maximum number of cores needed by an action. Buildfarm will enforce a max.

### `choose-queue`
**description:** place the action directly on the chosen queue (queue names must be known based on buildfarm configuration).  

**use case:** Other remote execution solutions have slightly different paradigms on deciding where actions go. They leverage `execution_properties` for selecting a "pool" of machines to send the action. We sort of have a pool of workers waiting on particular queues. For parity with this concept, we support this execution property which will take precedence in deciding queue eligibility.

### `env-vars`
TODO