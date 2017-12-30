# RDMA StorageServer

## How RDMA StorageServer works

Note that the initialization process are called before starting an implementation of StorageServer.

1. It creates an instance of RdmaPassiveEndpointGroup with the given configuration.

2. Get an RdmaServerEndpoint from the RdmaPassiveEndpointGroup.

3. The RdmaPassiveEndpointGroup initializes with a new RdmaStorageEndpointFactor

4. The RdmaServerEndpoint bind to the URI specified in the configuration.



