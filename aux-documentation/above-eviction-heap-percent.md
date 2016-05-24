## Eviction Heap Percentage is exceeded:

Eviction is configured for a region and implemented on a per member basis.
The Least Recently Used data on the member is chosen for eviction from 
the region.
While eviction can be based on the number of entries in a region 
or on the total number of bytes used for the region,
this situation concentrates on eviction based on the percentage 
of tenured heap space used.

LRU eviction is meant to effectively deal with occasional, 
bursty use of heap space.
When the eviction-heap-percentage is exceeded,
the resource manager evicts entries,
such that they can be garbage collected and thus, free heap space.
If heap memory usage exceeds the eviction-heap-percentage most of the time,
then the resource manager and the garbage collector impact system performance.
It means that the jvm/member is improperly sized.

If the eviction-heap-percentage is exceeded,
immediate and urgent action is *not* needed.
Instead, monitor to determine if and when further changes are indicated.

## How the situation manifests (presents):
A message is logged at the info level when monitoring detects 
that the threshold is crossed.
Evictions occur.

## Action to take:
Track heap space usage over time.
If usage is consistently above the heap-eviction-percentage,
such that evictions occur,
implement one or more of the following:

1. Increase the configured value for eviction-heap-percentage.
Be aware that the standard recommendation is for the
eviction-heap-percentage to be 10% lower than the 
critical-heap-percentage and 10% higher than 
the GC initiating occupancy fraction.
The default values are set this way.
However, for larger heap sizes,
the 10% difference may be too much,
causing evictions when there is plenty of heap space available.
Raising the eviction-heap-percentage tunes the system.

2. Lower the JVM garbage collector's initiating occupancy fraction.
The goal is for garbage collection to be initiated
when less memory has been used,
such that more heap space is freed,
and the eviction heap percentage will no longer be exceeded.
This only works if there is memory that can be garbage collected,
such that the system stays below the eviction heap percentage most of the time.
Track garbage collection to make sure that the collector is 
not constantly running, as it potentially competes for system resources.

3. Increase tenured heap size.

4. For GemFire 9.0, use a larger-capacity, off-heap space.

5. For partitioned regions,
increase capacity by adding more members (servers).
Rebalance across the system.

