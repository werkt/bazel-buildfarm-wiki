# Instances

## Definition

An instance is a namespace which represents a pool of resources available to a remote execution client. All requests to the Remote Execution Services are identified with an instance name, and a client is expected to communicate with the same instance between requests in order to accomplish aggregate activities. For instance, a typical client usage of FindMissingBlobs, then one or more Write/BatchUploadBlobs, Execute, and zero or more Read/BatchReadBlobs would all have the same instance name associated with the requests.

## Implementation

Buildfarm uses modular instances, where an instance is associated with one concrete type that governs its behavior and functionality. The Remote Execution API uses instance names to identify every request made, which allows Buildfarm instances to represent partitions of resources. One endpoint may support many different instances, each with their own name, and each instance will have its own type.

# Typical Deployment

Modules and their Function
