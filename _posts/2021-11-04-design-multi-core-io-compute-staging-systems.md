---
layout: post
title: Design of multi-core IO and compute staging systems
categories: low-level-systems
---

**rule 01**: All kind of data abstractions io_submit, channels, network streams, io_uring match with the sink/stream interface

**rule 02**: All sinks/streams are backed by real resources, hence bounded and hence block on empty/full for read/write respectively

**rule 03**: Turning blocking into non-blocking requires support of mechanism to register for notification when something is blocked and notification its unblocked

**rule 04**: X operations for N objects done independently is total X*N operations. If a bulk API exists then it reduces to X operations instead. We rarely work on 1 
		 stream/sink. Hence, bulk optimization for each individual step works for it. For streams/sink, X1 = read/write X2 = register for notification X3 = get 
	 	 notified

**rule 05**: IO async does not need lot of CPU hence a single thread can multiplex lot of tiny tasks which are doing lot of IO. We can spin up compute heavy tasks into
		 threads of their own

**rule 06**: Per rule 05, largest contiguous sequence of stages that are all light on cpu and heavy on io can be combined into a single stage. This single stage can 
		 then multiplex all of it

**rule 07**: After applying rule 06, an io stage will be surrounded by all compute stages

**rule 08**: Length of contiguous sequence of compute stages though, is determined by how much parallelism and we can get by running those stages in parallel in 
		 different threads and how cheap will it be to share/move data across these threads which will mostly be running on different cores.

**rule 09**: rule 06 is wrong if combining all io stages into single stage and 1 thread activiity makes other threads idle and wasted. We can make other threads/cores 
		 busy by splitting work horizontally (thread 1 does N1 streams/sinks, thread 2 does N2 streams/sinks etc.) or splitting vertically (thread 1 does X1 
		 operations, thread 2 does X2 operations and they communicate/share stage output with each other). horizontal split has more IO operation cost due to lack 
		 of sharing. vertical split has more data exchange cost across cores/cache lines. IO operation cost is higher than data exchange cost across cores/lines 
		 hence horizontal splitting is more cpu efficient and better choice if reducing cpu consumption is the goal. However that much X-depth may not be there to 
		 utilize all cores/threads and one may be forced to utilize the N-depth by horizontal splitting.

**rule 10**: compute stages by definition need lot of time to produce output. So, it should not be sitting idle if input and free core is available. Better to produce 
		 the output and wait for the subsequent compute or io stage to be ready than wait for readiness and waste time producing a result. In general compute stage 
		 must read its input, process it, produce output and keep stashing the outputs until the wait buffer holding results until the subsequent stage is ready 
		 gets full. At that point it should apply backpressure
