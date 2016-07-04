# Overview

The backbone of the Crail I/O architecture is Crail Store, a high-performance multi-tiered data store for temporary data in analytics workloads. If the context permits we often refer to Crail Store simply as Crail. Data processing frameworks and applications may directly interact with Crail for fast storage of in-flight data, but more commonly the interaction takes place through one of the Crail modules. As an example, the CrailHDFS adapter provides a standard HDFS interface allowing applications to use Crail Store the same way they use regular HDFS. Applications may want to use CrailHDFS for short-lived performance critical data, and regular HDFS for long-term data storage. CrailSparkIO is a Spark specific module which implements various I/O intensive Spark operations such as shuffle, broadcast, etc. Both CrailHDFS and CrailSparkIO can be used transparently with no need to recompile either the application or the data processing framework.

![](/assets/import.png)

Crail modules are thin layers on top of Crail Store. Implementing new modules for a particular data processing framework or a specific I/O operation requires only a moderate amount of work. At the same time, modules inherit the full benefits of Crail in terms of user-level I/O, performance and storage tiering. For instance, in the blog section we show that the Crail-based shuffle engine for Spark permits all-to-all data shuffling very close to the speed of the 100Gb/s network fabric.

