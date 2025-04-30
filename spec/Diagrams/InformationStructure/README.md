# Information Structure  

The internal data stores shall be structured according to the NMDA concepts ([IETF RFC 8342](https://datatracker.ietf.org/doc/html/rfc8342)).

The specified data stores are assigned the following semantic meanings:  
- The publicly offered services shall allow transferring the planning data into the CandidateDataStore.  
- After some validation tests, the planning data shall be copied into the RunningDataStore.  
- The information provided by the devices in the live network shall be consolidated into the OperationalDataStore.  

<img src="./CategoriesOfFunctions.png" alt="CategoriesOfFunctions" width="700" style="display: block; margin: 0 auto"/>  

This means for the Link objects:  
- A Link object inside the RunningDS is describing a planned microwave link that _is_ actually identified with a LinkID.  
- The Link objects inside the OperationalDS are describing relationships between AirInterface objects that _might_ correspond to a planned microwave link with a specific LinkID.  

As the devices cannot provide information about the connections between the devices, in the OperationalDS ... 
- an AirInterface is referenced by several Link objects connecting it with several other AirInterfaces.  
- the individual Link object might correspond to diverse planned microwave links.  

<img src="./MeaningOfLinks.png" alt="MeaningOfLinks" width="350" style="display: block; margin: 0 auto"/>  

Each of the Link objects in the OperationalDS has a probability to correspond to one of the planned microwave links with its specific LinkID.

So, the overall problem of writing the most likely LinkID into the externalLabel attribute at some AirInterface can be sub-structured into the following activities on the data stores:  
- The respective probabilities for a Link object in the OperationalDS to correspond to one of the planned microwave links (including its LinkID) in the RunningDS are to be estimated. This calculation shall compare the information about the planned microwave links in RunningDS with the information retrieved from the actual devices in the OperationalDS. The _resulting estimates are documented at the Link object in the OperationalDS_.  
- Based on the estimated probability values, the most likely distribution of the LinkIDs across the Link objects in the OperationalDS has to be calculated. Rules (such as: each LinkID may only be assigned once) must be observed. The _resulting assignments of LinkIDs are documented at the AirInterface objects in the RunningDS_.  
- If there would be differences between the values of LinkIDs at the AirInterface objects in the RunningDS and the OperationalDS, the values from the RunningDS would have to be configured into the device.  

The information within the three data stores shall have the following identical structure:  

<img src="./InformationStructure.png" alt="InformationStructure" width="700" style="display: block; margin: 0 auto"/>  

Its top level element is a [DomainController](./schemas/00_DomainController.yaml) that holds the parameter settings of the [Functions](./schemas/01_Function.yaml) and the [CurrentAlarms](./schemas/02_CurrentAlarm.yaml) within the LinkIdintoLtpWriter.  

Apart from that it holds four different documentations of the same [Network](./schemas/03_NetworkControlDomain.yaml) (running, operational, startup and candidate), which is exclusively composed from [Devices](./schemas/31_Device.yaml) that share a couple of different [DeviceTemplates](./schemas/07_DeviceTemplate.yaml).  
