# NVMf StorageServer

## Initialization of NVMf StorageServer

Note that the initialization process are called before starting an implementation of StorageServer.

1. It creates an instance of NvmeEndpointGroup with the disni library.
2. The NvmeEndpointGroup create an endpoint to connect to the NVM devices via the NVMf protocol.
3. Mark the NVMf StorageServer as alive.



