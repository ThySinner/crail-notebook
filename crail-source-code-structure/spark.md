# Spark Module

The Crail Spark module includes a Crail based shuffle engine as well as a broadcast service. The shuffle engine maps key ranges to directories in Crail. Each map task, while partitioning the data, appends key/value pairs to individual files in the corresponding directories. Tasks running on the same core within the cluster append to the same files, which reduces storage fragmentation.

  
![](http://www.crail.io/overview/shuffle.png)  
  


As is the case with the Crail HDFS adapter, the shuffle engine benefits from the performance and tiering advantages of the Crail data store. For instance, individual shuffle files are served using horizontal tiering. In most cases that means the files are filling up the memory tier as long as there is some DRAM available in the cluster, after which they extend to the flash tier. The shuffle engine further uses the Crail location affinity API to make sure local DRAM and flash is preferred over remote DRAM and flash respectively. Note that the shuffle engine is also completely zero-copy, as it transfers data directly from the I/O memory of the mappers to the I/O memory of the reducers.

The Crail-based broadcast plugin for Spark stores broadcast variables in Crail files. In contrast to the shuffle engine, broadcast is implemented without location affinity, which makes sure the underlying blocks of the Crail files are distributed across the cluster, leading to a better load balancing when reading broadcast variables. Crail shuffle and broadcast components can be enabled in Spark by setting the following system properties in spark-defaults.conf:

```
spark.shuffle.manager		org.apache.spark.shuffle.crail.CrailShuffleManager
spark.broadcast.factory		org.apache.spark.broadcast.CrailBroadcastFactory

```

Both broadcast and shuffle require Spark data objects to be serialized into byte streams \(as is also the case for the default Spark broadcast and shuffle components\). Even though both Crail components work fine with any of the Spark built-in serializers \(e.g. Kryo\), to achieve the best possible performance applications running on Crail are encouraged to provide serialization and deserialization methods for their data types explicitly. One reason for this is that the built-in Spark serializers assume byte streams of type java.io.\(InputStream/OutputStream\). These stream types are less powerful than Crail streams. For instance, streams of type InputStream/OutputStream expose a synchronous API and are restricted to on-heap memory. Crail streams, on the other hand, expose an asynchronous API and integrate well with off-heap memory to reduce data copies. By defining custom serialization/deserialization methods, applications can take full advantage of Crail streams during broadcast and shuffle operations. Moreover, serializers dedicated to one particular application type may further exploit information about the specific application data types to achieve a better performance. As we show in the blog, a custom serializer for a sorting application running on key/value objects of a fixed length byte array will not need to store serialization meta data, which reduces the final data size and simplifies the serialization process.

  
![](http://www.crail.io/overview/serializer.png)  
  


Serialization is one important aspect for broadcast and shuffle operations, sorting another, even though specific to shuffling. Sorting directly follows the network fetch phase in a shuffle operation if a key ordering is requested by the application. Again, the Crail shuffle engine works fine with the Spark built-in sorter, but often the shuffle performance can be improved by an application specific sorter. For instance, an application may use the Crail GPU tier to store data. In that case, sorting can be pushed to the GPU, rather than fetching the data into main memory and sorting it on the CPU. In other cases, the application may know the data types in advance and use the information to simplify sorting \(e.g. use Radix sort instead TimSort\).

Application specific serializers and sorters can be defined by setting the following system properties in spark-defaults.conf:

```
spark.crail.shuffle.serializer  
spark.crail.shuffle.sorter
```



