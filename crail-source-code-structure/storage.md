# Storage

Crail Store implements a hierarchical namespace across a cluster of RDMA interconnected storage resources such as DRAM or flash. Storage resources may be co-located with the compute nodes of the cluster, or disagreggated inside the data center, or a mix of both. Nodes in the Crail namespace consist of arrays of blocks distributed across storage resources in the cluster. Crail groups storage resources into different tiers \(e.g, DRAM, flash, disk\) and permits node segments \(blocks\) to be allocated in specific tiers but also across tiers. For instance, by default Crail uses horizontal tiering where higher performing storage resources are filled up across the cluster prior to using lower performing tiers -- resulting in a more effective usage of storage hardware.

Crail currently supports five types of nodes to be stored in its namespace: regular data files, directories, multifiles, tables and keyvalue nodes. Regular data files are append-and-overwrite with only a single-writer permitted per file at a given time. Append-andoverwrite means that – aside from appending data to the file – overwriting existing content of a file is also permitted. Directories in Crail are just regular files containing fixed length directory records. The advantage is that directory enumeration becomes just a standard file read operation which makes enumeration fast and scalable with regard to the number of directory entries. Multifiles are files that can be written concurrently. Internally, a multifile very much resembles a flat directory. Multiple concurrent substreams on a multifile are backed with separate files inside the directory. Keyvalue nodes are similar to data files, except that keyvalue nodes can be updated with completely new values. Updating keyvalue nodes can happen concurrently by multiple clients in which case the last update prevails. Keyvalue nodes can only be attached to tables, which are similar to directory with the exception that tables cannot be nested.

![](http://crail.io/overview/filesystem2.png)

  


Access to storage resources over the network -- as happening during file read/write operations -- are implemented using a variety of network and stoage APIs and protocols. Which API and protocol is uses to read/write a particular block depends to the type of storage that is accessed. For instance, accesses to blocks residing in the DRAM tier are implemented using one-sided read/write RDMA operations. Similarly, access to blocks residing in the NVMe tier of Crail are implemented using NVMe of fabrics \(NVMf\). In most of the cases, the network and storage devices are access directly from user-space via RDMA or other user-level APIs such as DPDK or SPDK. Crail is built in a way that new storage tiers can be added easily: storage tiers are actual plugins. With this, Crail can support new protocols and APIs and leverage upcomding storage and network technologies efficiently.

  
![](http://www.crail.io/overview/tiering.png)  
  


Crail's top level storage API offers asynchronous non-blocking functions for reading and writing data. Typically, the user-level APIs used by the storage tiers offer an asynchronous interface to the hardware which Crail directly leverages, thus, Crail is naturally asynchronous and does not need to engage any extra threads to provide asynchronism. The asynchronous API is particular important in the context of data processing, as it facilitates interleaving of computation and I/O in data processing workloads. Aside from the standard read/write operations, Crail provides extra semantics geared towards its use case. For instance, Crail exports functions to allocate dedicated I/O buffers from a reuseable pool -- memory that is registered with the hardware if needed to support zero-copy I/O. Moreover, Crail provides detailed control as to which storage tier and location preference should be used when allocating file system resources.

Crail not only exports a Java API but also is written entirely in Java, which makes it easy to use and allows for a better integration with data processing frameworks like Spark, Flink, Hadoop, etc. A simple example of a Crail write operation is shown below:

```
CrailConfiguration conf = new CrailConfiguration();
CrailFS fs = CrailFS.newInstance(conf);
CrailFile file = fs.createFile(filename, 0, 0).get().syncDir();
CrailOutputStream outstream = file.getDirectOutputStream();
ByteBuffer dataBuf = fs.allocateBuffer();
Future
<
DataResult
>
 future = outputStream.write(dataBuf);
...
future.get();

```

Crail uses [DiSNI](https://github.com/zrlio/disni), a user-level network and storage stack for the Java virtual machine. DiSNI allows data to be exchanged in a zero-copy fashion between Java I/O memory and remote storage resources over RDMA.

