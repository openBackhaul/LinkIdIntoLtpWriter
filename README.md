Delete this link at the end of the specification process:  
- [Roadmap to Specification](../../issues/8)

# LinkIdIntoLtpWriter

### Location
The LinkIdIntoLtpWriter is part of the HighPerformanceNetworkInterface.

### Description
Planning data (includes content of legacy databases like x:akta or APT) is identified by IDs generated by Telefonica (e.g. network element ID or link ID). Actual configuration data, which has been retrieved from the network, is identified by IDs (e.g. LTP UUID) that have been generated by devices. Joining planning and configuration data is required for use cases like data reconciliation, configuration verification, live network view and many more. The LinkidIntoLtpWriter is calculating (might involve probability calculations) pairs of Telefonica Link ID and ONF LTP UUID and writes the Telefonica Link ID into the LTP inside the device.

### Relevance
If the LinkIdIntoLtpWriter would write wrong LinkIDs into devices, other applications will write correct data into wrong devices.

### Resources
- [Specification](./spec/)
- [TestSuite](./testing/)
- [Implementation](./server/)

### Comments
This application will be specified during [training for ApplicationOwners](https://gist.github.com/openBackhaul/5aabdbc90257b83b9fe7fc4da059d3cd).
