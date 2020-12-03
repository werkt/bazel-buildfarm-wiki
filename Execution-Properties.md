Buildfarm supports the following [execution properties](https://docs.bazel.build/versions/master/be/common-definitions.html#common.exec_properties).  
Custom properties can also be added to buildfarm's configuration in order to facilitate queue matching (see [Platform Queues](https://github.com/bazelbuild/bazel-buildfarm/wiki/Shard-Platform-Operation-Queue)).
Please note that not all execution properties may be relevant to you or the best option depending on your client.  
For example, some execution_properties were created to facilitate behavior before bazel had a better solution in place.

### `min-cores`
**description:** the minimum number of cores needed by an action.  Should be set to >= 1

### `max-cores`
**description:** the maximum number of cores needed by an action. Buildfarm will enforce a max.

### `choose-queue`
**description:** place the action directly on the chosen queue (queue name must be known based on buildfarm configuration).  

**use case:** Other remote execution solutions have slightly different paradigms on deciding where actions go. They leverage `execution_properties` for selecting a "pool" of machines to send the action. We sort of have a pool of workers waiting on particular queues. For parity with this concept, we support this execution property which will take precedence in deciding queue eligibility.

### `env-vars`
**description:** ensure the action is executed with additional environment variables.  These variables are applied last in the order given.

**use case:**
Users may need to set additional environment variables through `exec_properties`.  
Changing code or using `--action_env` may be less feasible than specifying them through these exec_properties.  
Additionally, the values of their environment variables may need to be influenced by buildfarm decisions.  

For example, pytorch tests can still see the underlying hardware through `/proc/cpuinfo`.  
Despite being given 1 core, they see all of the cpus and decide to spawn that many threads. This essentially starves them and gives poor test performance (we may spoof cpuinfo in the future).  Another solution is to use env vars `OMP_NUM_THREADS` and `MKL_NUM_THREADS`.  This could be done in code, but we can't trust that developers will do it consistently or keep it in sync with `min-cores` / `max-cores`.  Allowing these environment variables to be passed the same way as the core settings would be ideal.  

**Standard Example:**  
This test will succeed when env var TESTVAR is foobar, and fail otherwise.
```
#!/bin/bash
[ "$TESTVAR" = "foobar" ]
```
```
./bazel test  \
--remote_executor=grpc://127.0.0.1:8980 --noremote_accept_cached  --nocache_test_results \
//env_test:main
FAIL
```

```
./bazel test --remote_default_exec_properties='env-vars={"TESTVAR": "foobar"}' \
 --remote_executor=grpc://127.0.0.1:8980 --noremote_accept_cached  --nocache_test_results \
//env_test:main
PASS
```
**Template Example:**
If you give a range of cores, buildfarm has the authority to decide how many your operation actually claims.  You can let buildfarm resolve this value for you (via [mustache](https://mustache.github.io/)).  
```
#!/bin/bash
[ "$MKL_NUM_THREADS" = "1" ]
```
```
./bazel test  \
--remote_executor=grpc://127.0.0.1:8980 --noremote_accept_cached  --nocache_test_results \
//env_test:main
FAIL
```
```
./bazel test  \
--remote_default_exec_properties='env-vars="MKL_NUM_THREADS": "{{limits.cpu.claimed}}"' \
--remote_executor=grpc://127.0.0.1:8980 --noremote_accept_cached  --nocache_test_results \
//env_test:main
PASS
```

**Available Templates:**  
`{{limits.cpu.min}}`: what buildfarm has decided is a valid min core count for the action.  
`{{limits.cpu.max}}`: what buildfarm has decided is a valid max core count for the action.  
`{{limits.cpu.claimed}}`: buildfarm's decision on how many cores your action should claim.  

### `debug-before-execution` (not implemented)
**description:** Fails the execution with important debug information on how the execution will be performed.

### `debug-after-execution` (not implemented)
**description:** Runs the execution, but fails it afterward with important debug information on how the execution was performed.

