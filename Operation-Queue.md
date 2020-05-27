# Operation Queue
Elements are removed from the prequeue and added to the operation queue.  
The operation queue is used to hold operations for workers to perform.  

## Working with different platform requirements 
Some operations may require different platform requirements in order to execute.
Likewise, specific workers may only want to take on work that they deem eligible.
To solve this, the operation queue can be customized to divide work into separate provisions queues so that specific workers can choose which queue to read from.  

Provision queues are intended to represent particular operations that should only be processed by particular workers. An example use case for this would be to have two dedicated provision queues for CPU and GPU operations. CPU/GPU requirements would be determined through the remote api's command platform properties. We designate provision queues to have a set of "required provisions" (which match the platform properties). This allows the scheduler to distribute operations by their properties and allows workers to dequeue from particular queues.

### Server Example

### Worker Example