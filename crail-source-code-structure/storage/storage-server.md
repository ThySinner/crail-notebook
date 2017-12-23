# StorageServer

Once a StorageServer starts, it will complete the following steps:

1. Reads and verifies the configuration from file _crail-site.conf._
2. Creates an instance of StorageTier.
3. The StorageTier intializes a StorageServer with the given configuration and arguments.
4. The StorageServer starts as a thread.
5. Creates a Namenode RPC client.
6. Builds a ConcurrentLinkedQueue which contains RPC connection to each NameNode.
7. Creates an instance of StorageRpcClient



