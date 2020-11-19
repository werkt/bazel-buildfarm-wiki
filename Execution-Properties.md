Buildfarm supports the following `execution_properties`.

`min-cores`: the minimum number of cores needed by the action. Adjusted to >= 1

`max-cores`: the maximum number of cores needed by the action. Buildfarm can enforce a max.

`choose-queue`: put an action directly on the specified queue (queue names must be known based on buildfarm configuration) 

`env-vars`: TODO