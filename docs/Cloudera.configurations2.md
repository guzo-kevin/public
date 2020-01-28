# Map reduce memory

On each node, yarn.nodemanager.resource.memory-mb, determine memory yarn can allocate to map reduce jobs.

When there is a job request, the client uses mapreduce.map.memory.mb and mapreduce.reduce.memory.mb to request memory.  Yarn use yarn.scheduler.minimum-allocation-mb, or n* yarn.scheduler.minimum-allocation-mb  to assign memory to client task, up to yarn.scheduler.maximum-allocation-mb. 

before map reduce tasks can be scheduled and run, the application master need to be allocated with a container, its memory request is by yarn_app_mapreduce_am_resource_mb


useful info: http://ercoppa.github.io/HadoopInternals/
