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

```java
public static void main(String[] args) throws Exception {
    Logger LOG = CrailUtils.getLogger();
    CrailConfiguration conf = new CrailConfiguration();
    CrailConstants.updateConstants(conf);
    CrailConstants.printConf();
    CrailConstants.verify();

    int splitIndex = 0;
    for (String param : args){
        if (param.equalsIgnoreCase("--")){
            break;
        } 
        splitIndex++;
    }

    //default values
    StringTokenizer tokenizer = new StringTokenizer(CrailConstants.STORAGE_TYPES, ",");
    if (!tokenizer.hasMoreTokens()){
        throw new Exception("No storage types defined!");
    }
    String storageName = tokenizer.nextToken();
    int storageType = 0;
    HashMap<String, Integer> storageTypes = new HashMap<String, Integer>();
    storageTypes.put(storageName, storageType);
    for (int type = 1; tokenizer.hasMoreElements(); type++){
        String name = tokenizer.nextToken();
        storageTypes.put(name, type);
    }
    int storageClass = -1;

    //custom values
    if (args != null) {
        Option typeOption = Option.builder("t").desc("storage type to start").hasArg().build();
        Option classOption = Option.builder("c").desc("storage class the server will attach to").hasArg().build();
        Options options = new Options();
        options.addOption(typeOption);
        options.addOption(classOption);
        CommandLineParser parser = new DefaultParser();

        try {
            CommandLine line = parser.parse(options, Arrays.copyOfRange(args, 0, splitIndex));
            if (line.hasOption(typeOption.getOpt())) {
                storageName = line.getOptionValue(typeOption.getOpt());
                storageType = storageTypes.get(storageName).intValue();
            }                
            if (line.hasOption(classOption.getOpt())) {
                storageClass = Integer.parseInt(line.getOptionValue(classOption.getOpt()));
            }                    
        } catch (ParseException e) {
            HelpFormatter formatter = new HelpFormatter();
            formatter.printHelp("Storage tier", options);
            System.exit(-1);
        }
    }
    if (storageClass < 0){
        storageClass = storageType;
    }

    StorageTier storageTier = StorageTier.createInstance(storageName);
    if (storageTier == null){
        throw new Exception("Cannot instantiate datanode of type " + storageName);
    }
    StorageServer server = storageTier.launchServer();

    String extraParams[] = null;
    splitIndex++;
    if (args.length > splitIndex){
        extraParams = new String[args.length - splitIndex];
        for (int i = splitIndex; i < args.length; i++){
            extraParams[i-splitIndex] = args[i];
        }
    }
    server.init(conf, extraParams);
    server.printConf(LOG);

    Thread thread = new Thread(server);
    thread.start();

    RpcClient rpcClient = RpcClient.createInstance(CrailConstants.NAMENODE_RPC_TYPE);
    rpcClient.init(conf, args);
    rpcClient.printConf(LOG);                    

    ConcurrentLinkedQueue<InetSocketAddress> namenodeList = CrailUtils.getNameNodeList();
    ConcurrentLinkedQueue<RpcConnection> connectionList = new ConcurrentLinkedQueue<RpcConnection>();
    while(!namenodeList.isEmpty()){
        InetSocketAddress address = namenodeList.poll();
        RpcConnection connection = rpcClient.connect(address);
        connectionList.add(connection);
    }
    RpcConnection rpcConnection = connectionList.peek();
    if (connectionList.size() > 1){
        rpcConnection = new RpcDispatcher(connectionList);
    }        
    LOG.info("connected to namenode(s) " + rpcConnection.toString());                


    StorageRpcClient storageRpc = new StorageRpcClient(storageType, CrailStorageClass.get(storageClass), server.getAddress(), rpcConnection);

    HashMap<Long, Long> blockCount = new HashMap<Long, Long>();
    long sumCount = 0;
    long lba = 0;
    while (server.isAlive()) {
        StorageResource resource = server.allocateResource();
        if (resource == null){
            break;
        } else {
            storageRpc.setBlock(lba, resource.getAddress(), resource.getLength(), resource.getKey());
            lba += (long) resource.getLength();

            DataNodeStatistics stats = storageRpc.getDataNode();
            long newCount = stats.getFreeBlockCount();
            long serviceId = stats.getServiceId();

            long oldCount = 0;
            if (blockCount.containsKey(serviceId)){
                oldCount = blockCount.get(serviceId);
            }
            long diffCount = newCount - oldCount;
            blockCount.put(serviceId, newCount);
            sumCount += diffCount;
            LOG.info("datanode statistics, freeBlocks " + sumCount);        
        }
    }

    while (server.isAlive()) {
        DataNodeStatistics stats = storageRpc.getDataNode();
        long newCount = stats.getFreeBlockCount();
        long serviceId = stats.getServiceId();

        long oldCount = 0;
        if (blockCount.containsKey(serviceId)){
            oldCount = blockCount.get(serviceId);
        }
        long diffCount = newCount - oldCount;
        blockCount.put(serviceId, newCount);
        sumCount += diffCount;            

        LOG.info("datanode statistics, freeBlocks " + sumCount);
        Thread.sleep(CrailConstants.STORAGE_KEEPALIVE*1000);
    }            
}
```

## StorageServer types

| Type | Comment |
| :--- | :--- |
| NvmfStorageServer | A StorageServer implementation based on Non-Volatile Memory Express over Fabrics standard. |
| RdmaStorageServer | A StorageServer implementation based on Remote Direct Memory Access. |

## UML Class Hierarchy Diagram

![](/assets/storage-uml.png)

![](/assets/storage-tier-uml.png)

