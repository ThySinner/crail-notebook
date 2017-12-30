# RDMA StorageServer

## How RDMA StorageServer works

Note that the initialization process are called before starting an implementation of StorageServer.

1. It creates an instance of RdmaPassiveEndpointGroup with the given configuration.

2. Get an RdmaServerEndpoint from the RdmaPassiveEndpointGroup.

3. The RdmaPassiveEndpointGroup initializes with a new RdmaStorageEndpointFactor

4. The RdmaServerEndpoint bind to the URI specified in the configuration.

```java
public void init(CrailConfiguration conf, String[] args) throws Exception {
	RdmaConstants.init(conf, args);

	this.serverAddr = getDataNodeAddress();
	if (serverAddr == null){
		LOG.info("Configured network interface " + RdmaConstants.STORAGE_RDMA_INTERFACE + " cannot be found..exiting!!!");
		return;
	}
	URI uri = URI.create("rdma://" + serverAddr.getAddress().getHostAddress() + ":" + serverAddr.getPort());
	this.datanodeGroup = new RdmaPassiveEndpointGroup<RdmaStorageServerEndpoint>(-1, RdmaConstants.STORAGE_RDMA_QUEUESIZE, 4, RdmaConstants.STORAGE_RDMA_QUEUESIZE*100);
	this.datanodeServerEndpoint = datanodeGroup.createServerEndpoint();		
	datanodeGroup.init(new RdmaStorageEndpointFactory(datanodeGroup, this));
	datanodeServerEndpoint.bind(uri);
	
	this.dataDirPath = getDatanodeDirectory(serverAddr);
	LOG.info("dataPath " + dataDirPath);
	this.allocatedSize = 0;
	this.fileCount = 0;
	if (!RdmaConstants.STORAGE_RDMA_PERSISTENT){
		clean();		
	} 
}
```



