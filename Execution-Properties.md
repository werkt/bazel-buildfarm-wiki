Buildfarm supports the following `execution_properties`.  
New execution_properties may be added in buildfarm config in order support matching to different worker types.

### min-cores
the minimum number of cores needed by the action.  Should be set to >= 1

### max-cores
the maximum number of cores needed by the action. Buildfarm can enforce a max.

### choose-queue
put an action directly on the specified queue (queue names must be known based on buildfarm configuration).  

Other remote execution solutions have slightly different paradigms on deciding where actions go. They leverage execution_properties for selecting a "pool" of machines to send the action. We sort of have a pool of workers waiting on particular queues. For parity with this concept, we support a new execution property called choose-queue which will take precedence in deciding eligibility.

### env-vars
TODO