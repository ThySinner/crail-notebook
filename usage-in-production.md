# Usage In Production

Crail has a plug-able module for the transport and storage tiers. Currently, there are RDMA, NVMeF, and TPC-netty-based. The last one \([https://github.com/zrlio/crail-netty](https://github.com/zrlio/crail-netty)\) does not require RDMA network and should sufficient to run on 10 Gbps networks. We also recently developed a native TPC-nio based implementation, which is in testing and will be released soon. It has better performance than the netty one. So any TCP-\* tier can be used to test/develop a feature on non-RDMA networks \(or even the loopback network\). 

There is SoftiWARP \([https://github.com/zrlio/softiwarp](https://github.com/zrlio/softiwarp)\) that is a software emulation of RDMA functionality in kernel for non-RDMA networks. With this, you can run RDMA code \(including Crail\) anywhere on any TCP/IP-supporting network. SoftiWARP is under discussion at the Linux kernel mailing list for inclusion in the upstream kernel.

Unfortunately Azure Cloud is the only cloud provider we know who is providing RDMA and NVM devices in their servers.

If you are after prototyping and testing then any of these options can be used. However, for the top performance we naturally recommend using RDMA networks. However, I suspect that a good socket-based implementation would do a fine job, if not the best job.





