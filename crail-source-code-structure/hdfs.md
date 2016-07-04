# HDFS Adapter

The Crail HDFS adaptor enables users to access Crail using the standard HDFS API. For instance, administrators can interact with Crail using the standard HDFS shell:

```
./bin/crail fs -mkdir /test
./bin/crail fs -ls /
./bin/crail fs -copyFromLocal 
<
path-to-local-file
>
 
./bin/crail fs -cat /test/
<
file-name
>
```

Moreover, regular HDFS-based applications will transparently work with Crail when using fully qualified path names \(or when specifying Crail as the default Hadoop file system\):

```
Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(conf);
fs.create("crail://test/hello.txt");

```



