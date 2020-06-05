ActionCahe is a service that can be used to query whether a defined action has already been executed and, if so, download its result. The service API is defined in the [Remote Execution API](https://github.com/bazelbuild/remote-apis).

An `Action` encapsulates all the information required to execute an action. Such information includes the command, input tree containing subdirectory/file tree, environment variables, platform information. All the information will contribute to the digest computation of an `Action` so that execution of an `Action` multiple times will produce the same output. With this, hash of an `Action` can be used as a key to cached `ActionResult`, which store result and output of an `Action` after an `Action` is run by the `Execution` service.

