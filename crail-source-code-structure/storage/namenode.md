# NameNode

Once a NameNode starts, it will complete the following steps:

1. Read and verify the configuration from file _crail-site.conf_.
2. Get listening IP address and port of the primary NameNode.
3. Create an RpcNameNodeService according to the configuration item **crail.namenode.rpcservice**, which has a default value of "com.ibm.crail.namenode.NameNodeService".
4. Create an RpcBinding according to the configuration item **crail.namenode.rpctype,** which has a default value of "com.ibm.crail.namenode.rpc.darpc.DaRPCNameNode".

## Distribution Strategy For Writing Data

There are two distribution strategies for writing data:

* RoundRobin
* Random

When an RpcNameNodeService is created, it will use the configuration item **crail.namenode.blockselection**\(default value is "roundrobin"\) to choose the distribution strategy.

