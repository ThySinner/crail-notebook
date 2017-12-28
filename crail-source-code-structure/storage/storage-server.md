# StorageServer/DataNode

A StorageServer/DataNode is just like HDFS's DataNode, which store and manage data blocks.

## How StorageServer works

Once a StorageServer starts, it will complete the following steps:

1. Reads and verifies the configuration from file _crail-site.conf._
2. Creates an instance of StorageTier.
3. The StorageTier intializes a StorageServer with the given configuration and arguments.
4. The StorageServer starts as a thread.
5. Creates a Namenode RPC client.
6. Builds a ConcurrentLinkedQueue which contains RPC connection to each NameNode.
7. Creates an instance of StorageRpcClient.
8. The StorageServer constantly allocates resource until there is no resource left. The allocated resource is used for setting blocks. 
9. Finally, as long as the StorageServer is still alive, it will statistics the DataNode information.

## StorageServer types

| Type | Comment |
| :--- | :--- |
| NvmfStorageServer | A StorageServer implementation based on Non-Volatile Memory Express over Fabrics standard. |
| RdmaStorageServer | A StorageServer implementation based on Remote Direct Memory Access. |

## UML Class Hierarchy Diagram

![](/assets/storage-uml.png)

![](/assets/storage-tier-uml.png)

