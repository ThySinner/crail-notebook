# NVMf StorageServer

## Initialization of NVMf StorageServer

Note that the initialization process are called before starting an implementation of StorageServer.

1. It creates an instance of NvmeEndpointGroup with the disni library.
2. The NvmeEndpointGroup create an endpoint to connect to the NVM devices via the NVMf protocol.
3. Mark the NVMf StorageServer as alive.

```java
public void init(CrailConfiguration crailConfiguration, String[] args) throws Exception {
	if (initialized) {
		throw new IOException("NvmfStorageTier already initialized");
	}
	initialized = true;
	NvmfStorageConstants.parseCmdLine(crailConfiguration, args);

	NvmeEndpointGroup group = new NvmeEndpointGroup(new NvmeTransportType[]{NvmeTransportType.RDMA}, NvmfStorageConstants.SERVER_MEMPOOL);
	endpoint = group.createEndpoint();

	URI uri = new URI("nvmef://" + NvmfStorageConstants.IP_ADDR.getHostAddress() + ":" + NvmfStorageConstants.PORT +
				"/0/" + NvmfStorageConstants.NAMESPACE + "?subsystem=" + NvmfStorageConstants.NQN);
	endpoint.connect(uri);

	long namespaceSize = endpoint.getNamespaceSize();
	alignedSize = namespaceSize - (namespaceSize % NvmfStorageConstants.ALLOCATION_SIZE);
	offset = 0;

	isAlive = true;
}
```



