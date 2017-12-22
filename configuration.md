# Configuration

| Name | Default Value | Comment |
| :--- | :--- | :--- |
| crail.version | 3002 |  |
| crail.directorydepth | 16 |  |
| crail.tokenexpiration | 10 |  |
| crail.blocksize | 67108864 |  |
| crail.cachelimit | 1073741824 |  |
| crail.cachepath | /home/stu/craildata/cache |  |
| crail.user | stu |  |
| crail.shadowreplication | 1 |  |
| crail.debug | false |  |
| crail.statistics | true |  |
| crail.rpctimeout | 1000 |  |
| crail.datatimeout | 1000 |  |
| crail.buffersize | 1048576 | Crail buffer size in bytes |
| crail.slicesize | 524288 | Crail slice size in bytes |
| crail.singleton | false |  |
| crail.regionsize | 1073741824 | Crail region size in bytes |
| crail.directoryrecord | 512 |  |
| crail.directoryrandomize | true |  |
| crail.cacheimpl | com.ibm.crail.memory.MappedBufferCache | The class name of Crail cache implementation. Currently available values are "com.ibm.crail.memory.MappedBufferCache" or "com.ibm.crail.storage.nvmf.NvmfBufferCache". |
| crail.locationmap | _Empty string_ |  |
| crail.namenode.address | _Empty string_ | The IP address which a NameNode binds. |
| crail.namenode.fileblocks | 16 |  |
| crail.namenode.blockselection | roundrobin | The distribution strategy for writing data blocks. Currently available values are "roundrobin" or "random". |
| crail.namenode.rpctype | com.ibm.crail.namenode.rpc.darpc.DaRPCNameNode | The currently available value is only "com.ibm.crail.namenode.rpc.darpc.DaRPCNameNode". |
| crail.namenode.rpcservice | com.ibm.crail.namenode.NameNodeService | Currently available values are "com.ibm.crail.namenode.NameNodeService" or "com.ibm.crail.namenode.LogDispatcher". |
| crail.namenode.log | _Empty string_ |  |
| crail.storage.types | com.ibm.crail.storage.rdma.RdmaStorageTier | Currently available values are "com.ibm.crail.storage.rdma.RdmaStorageTier" and "com.ibm.crail.storage.nvmf.NvmfStorageTier" |
| crail.storage.classes | 1 |  |
| crail.storage.rootclass | 0 |  |
| crail.storage.keepalive | 2 |  |



