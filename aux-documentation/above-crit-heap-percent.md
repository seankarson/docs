## Critical Heap Percentage is exceeded:

Once the critical-heap-percentage is exceeded, operations do not complete,
instead throwing a LowMemoryException for 
`Region.put()`, `Region.create()`, `Region.putAll()`, 
any index creation operation, and at the execution of any function. 
This continues until the tenured heap usage falls more than 2% below 
the configured critical-heap-percentage, 
at which point the system returns to normal operation without intervention.

Exceeding this threshold implies that the member is about to run out 
of tenured heap memory, 
and the Java garbage collector has not been able to reduce 
the quantity of allocated memory.
Since further heap allocations would likely cause an OOME,
the member does not complete those operations that require 
further memory allocations.  
Exceeding this threshold signals that a significantly larger 
than expected burst of allocations occurred or that something 
is critically wrong with the member. 
If not a burst, then a non-trivial fix or redesign will be needed. 
Something is not right in the design or in the sizing of the jvm 
for the current set of regions that this member hosts. 

Be aware that exceeding the threshold on one member 
may affect other members.  
With operations not completing, 
the membership in the distributed system of the member that is 
above the critical-heap-percentage may be revoked. 
This can cause the remaining members to increase their 
usage of memory to compensate for the missing member. 
If these remaining members are not sized to handle the increased load, 
the critical heap threshold issue may cascade through the system.

## How the situation manifests (presents):

Operations that would require memory allocation throw `LowMemoryException`,
and what happens after that depends on the code that catches the exception. 
With system monitoring, 
alerts identify a member that is above the specified critical-heap-percentage.

## Action to take:
Some possible temporary (bandaid) measures:

1. Force garbage collection.
From the command line, 
find the PID of the member, 
and use it in the command `jcmd <pid> GC.run` .

2. Proactively shut down the member that is above 
the critical-heap-percentage, 
and restart it with a larger tenured heap size. 
This action does an orderly shutdown, 
assuming that an orderly shutdown is preferred over a crash due to an OOME.
It also presumes that the member can be given more heap space
(-Xmx and -Xms).
This measure does not address the underlying question of why 
the member surpassed the configured critical-heap-percentage;
that still needs to be done.

3. If the member that is above critical-heap-percentage 
is hosting considerably more buckets of partitioned regions 
than other members hosting the same partitioned regions, 
a rebalance may help. 
It moves entire buckets from one member to another, 
thereby reducing the heap space load. 
Invoke the rebalance on all partitioned regions, 
once connected, 
with `gfsh rebalance`, or use the `--include-region` option 
to explicitly specify those regions that are to be included in the rebalance.

## Redesign actions come under 3 categories.
More than one action can be taken to mitigate the issue.

1. Increase the amount of available memory.
The increase may need to be applied to all the members.
  * Start the member with a larger tenured heap size.  Upon restart of the member, increase values for `-Xmx` and `-Xms`.

  * Increase the critical-heap-percentage.
This implies that there really is more heap space available,
and it is currently wasted.
  * For GemFire 9.0, use off-heap memory.
  * Increase the quantity of members (servers) hosting partitioned regions to horizontally scale the system.

2. Tune garbage collection.  
  * Lower the initiating occupancy fraction of the CMS algorithm, 
such that the concurrent collection cycle is initiated earlier 
and at a lower threshold of heap space usage. 
This will hopefully free tenured heap space such that the system will not reach the critical-heap-percentage.

3. Modify the system's regions so that less heap space is used.
  * Evict data.
Set an eviction threshold in the resource manager, 
and change the region types to those with an lru-heap-percentage 
eviction strategy. 
Decide what happens when an entry is evicted.
It can be destroyed (entire entry) or the value can be overflowed to a disk.
  * Expire data.

