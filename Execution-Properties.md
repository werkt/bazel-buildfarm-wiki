Buildfarm supports the following [execution properties](https://docs.bazel.build/versions/master/be/common-definitions.html#common.exec_properties).  
Custom properties can also be added to buildfarm's configuration in order to facilitate queue matching (see [Platform Queues](https://github.com/bazelbuild/bazel-buildfarm/wiki/Shard-Platform-Operation-Queue)).

### min-cores
**description:** the minimum number of cores needed by the action.  Should be set to >= 1

### max-cores
**description:**the maximum number of cores needed by the action. Buildfarm will enforce a max.

### choose-queue
**description:** put an action directly on the specified queue (queue names must be known based on buildfarm configuration).  

**explanation:** Other remote execution solutions have slightly different paradigms on deciding where actions go. They leverage execution_properties for selecting a "pool" of machines to send the action. We sort of have a pool of workers waiting on particular queues. For parity with this concept, we support a new execution property called choose-queue which will take precedence in deciding eligibility.

### env-vars
TODO