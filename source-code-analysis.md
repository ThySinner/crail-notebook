# FAQ

* **How does Crail balance the network transmission pressure while Spark doing shuffle?** Data accesses in Crail usually result in metadata RPCs and then data operations. \(i\) For RPCs flow-controls, we use the Datacenter RPC design on RDMA networks which is describe in detail at https://dl.acm.org/citation.cfm?id=2670994. \(ii\) And for data operations which use RDMA one-sided reads and writes, we rely on NICs scheduling for flow control. So, on the iWARP-based RDMA networks it would use TCP flow control, on Infiniband networks it will use their credit-flow design, and on RoCE it will use the lossless Ethernet to manage network flows. We have played with some explicit QoS mechanism at the Crail level but we don't have conclusive results yet. So far, this design have helped us to successfully test up to 128 nodes. http://www.crail.io/blog/2017/01/sorting.html

* **How does Crail solve the data skew problem while Spark doing shuffle?**
  Internally, Crail uses a configurable block size. The default size is 1MB. All the Crail data files and directory \(and other files types\) are chopped in the block sizes and distributed over the participating data servers. This helps to balance data storage over all nodes. Typically we use Round-Robin but this is configurable. However, if compute processing is skewed in Spark \(for example, with 2 executor - one processing 10 GB and other processing 1GB\), then there is nothing in Crail to balance this. In the most of the workloads that we have tested with Spark \(RDDs, SQL, ML\), shuffle data is often balanced \(or can be balanced by increasing the number of tasks\). Because in order to get a good performance with Spark one needs to balance both the computer and storage.  

* **What is the strategy for caching SSD data blocks?**
  Currently there is no explicit data caching in Crail. There is a client-side, leased-based metadata caching \(and some pre-fetching in the case of sequential accesses\) but the basic hypothesis of the crail design is that network and storage devices are fast enough to deliver data at a very high speed. We don't have to complicate our lives with caching and invalidation processes. For example in a crail blog we have demonstrated that it is possible to deliver ~90+ Gbps from NMVe devices over the network without caching. http://www.crail.io/blog/2017/08/crail-nvme-fabrics-v1.html





