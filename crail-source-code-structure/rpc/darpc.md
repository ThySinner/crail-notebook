# DaRPC

DaRPC is an RPC framework and API for Java which uses RDMA to implement a tight integration between RPC message processing and network processing in a user space.

## Why DaRPC?

Remote Procedure Call \(RPC\) has been the cornerstone of distributed systems since the early 80s. Recently, new classes of large-scale distributed systems running in data centers are posing extra challenges for RPC systems in terms of scaling and latency.

DaRPC is an RPC framework and API for Java which uses RDMA to implement a tight integration between RPC message processing and network processing in a user space. DaRPC efficiently distributes computation, network resources, and RPC resources across CPU cores and DRAM to achieve a high aggregate throughput \(2-3M ops/sec\) at a very low per-request latency \(less than 10 μs\).

![](https://developer.ibm.com/open/wp-content/uploads/sites/50/2016/08/darpc-image.png)

The DaRPC programming API gives full control over network and compute resources during a remote procedure call. As such, applications decide how many CPU cores they want to use for server-side RPC processing and whether the application should interact with the network interface via interrupts or polling. RPC calls in DaRPC as issued by applications are non-blocking. In contrast to other non-blocking RPC libraries, DaRPC does not engage any background threads and, thus, achieves lower latencies. DaRPC relies on jVerbs, a framework and API for RDMA communication in Java.

## DaRPC In the Crail Project





