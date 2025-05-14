# List of Functions  

<img src="./diagrams/CategoriesOfFunctions.png" alt="CategoriesOfFunctions" width="700" style="display: block; margin: 0 auto"/>  

### Interpretation  
_(potentially it would make sense to facilitate multiple microwave links being passed in a single request;_  
_would that comply with the status of the consuming application?)_
- v1-add-planned-microwave-link  
  - Copies content of RunningDS into CandidateDS  
  - Creates the specified CC objects and AirInterface LTPs in CandidateDS (may already be in place)  
  - Creates an FC object between the specified CCs in CandidateDS (may already be in place)  
  - Creates a new Link object between the specified AirInterface LTPs in CandidateDS  
  - Calls p1-validation-orchestrator  
  - IF ResponseCode==204  
    - Copies content of CandidateDS into RunningDS  
    - Responds 204 to requestor  
    ELSE  
    - Responds ResponseCode to requestor  
- v1-remove-planned-microwave-link  
  - Copies content of RunningDS into CandidateDS  
  - Deletes the Link object with the specified LinkID from CandidateDS  
  - Deletes all FC objects that do not reference any Link object from CandidateDS  
  - Deletes all CC objects that are not referenced by any FC object from CandidateDS  
  - Calls p1-validation-orchestrator  
  - IF ResponseCode==204  
    - Copies content of CandidateDS into RunningDS  
    - Responds 204 to requestor  
    ELSE  
    - Responds ResponseCode to requestor  

### Validation  
- p1-validation-orchestrator  
  - Calls a configurable set of the TestFunctions listed below  
  - IF all ResponseCodes==204  
    - Responds 204  
    ELSE  
    - Responds the first ResponseCode different from 204 and terminates  
- p1-ensure-unique-link-ids  
  Ensures that each LinkID is unique in the list of planned microwave links  

_(further examples to be potentially removed by ApplicationOwner:)_
- p1-prevent-redundant-fcs  
  Ensures that each pair of CCs is referenced by a maximum of one FC object  
- p1-prevent-redundant-links  
  Ensures that each pair of AirInterface LTPs is referenced by a maximum of one Link object  
- p1-ensure-every-fc-having-at-least-one-link
  Ensures that each FC object is referencing at least one Link object  

### Measurement  
- p1-calculate-ltp-external-label (cyclic operation)  
  - Picks next FC object from rolling list in RunningDS  
  - Updates OperationalDS by reading the necessary information about all AirInterface LTPs and the Equipment of the referenced devices from MWDI  
  - IF device cannot be found in MWDI
    - Creates an entry with ErrorCode [to be defined#1] that is referencing the FC object and the CC object (as in RunningDS) in the CurrentAlarms  
    - Deletes existing FC object, referenced Link objects, affected CC object and attached AirInterface LTPs from OperationalDS  
    - Terminates calculation of externalLabel for this FC  
  - Creates Link objects between these AirInterface LTPs in OperationalDS (may already be in place; some may even be deleted, if AirInterface LTPs couldn't be found)  
  - Updates FC object between the two CC objects in OperationalDS  
  - Reads the LinkIDs of all Link objects referenced by the FC object in RunningDS  
  - Adds these LinkIDs into the Score tables at all Link objects in OperationalDS (may already be in place)  
  - Calculates the Scores for all LinkIDs at all Link objects referenced by the picked FC object and write them into the Score tables in OperationalDS  
    - IF device data is incomplete and Scores cannot be calculated  
      - Creates an entry with ErrorCode [to be defined#2] that is referencing the FC object, the affected CC and AirInterface LTP (as in RunningDS) in the CurrentAlarms  
      - Deletes existing Scores for all LinkIDs at all Link objects referenced by the picked FC object in OperationalDS  
      - Does not delete existing entries in the calculatedLinkId attribute at the Link objects referenced by the picked FC in OperationalDS  
      - Terminates calculation of externalLabel for this FC  
  - Calculates the most likely distribution of the LinkIDs on the Link objects referenced by the picked FC object and write the results into the calculatedLinkId attribute at the Link objects in OperationalDS (some may stay empty)  
  - Compares calculatedLinkId attribute at the Link object with the externalLabel attributes at both referenced AirInterface (all in OperationalDS)  
    - IF externalLabel != calculatedLinkId  
      - creates an entry with ErrorCode [to be defined#3] that is referencing the Link object and the AirInterface LTP in the CurrentAlarms  
      ELSE  
      - Checks CurrentAlarms at Link object and AirInterface LTP for potentially existing entries with ErrorCode [to be defined#3] and deletes them  

### Monitoring  
- ./. (cyclic operation)  

_(further examples to be potentially removed by ApplicationOwner:)_
- p1-check-if-cc-external-label-equal-to-mount-point  
- p1-check-if-operational-tx-power-is-below-planned  

### Implementation  
- p1-implementation-orchestrator (cyclic operation)  
  - Picks next FC object from rolling list in CurrentAlarms  
  - Identifies errored object and checks dateOfNextAttemptToFix  
    - IF currentDate > dateOfNextAttemptToFix  
      - Requests ImplementationFunction according definitions in ErrorCode table  
      - Receives Response
  - Restart cycle

- p1-update-ltp-external-label  
  - Reads calculatedLinkId attribute from Link and mountName + AirInterfaceUuid from AirInterface in OperationalDS
  - Sends PUT request to MWDG://live/mountName/AirInterfaceUuid/externalLabel with calculatedLinkId from Link in Operational  
    - Documents date-of-attempt and response-code in past-attempts-to-fix in CurrentAlarms  
    - IF ResponseCode==204  
      - Documents date-of-next-attempt-to-fix in CurrentAlarms to a bit more remote future (so completion will certainly be measured before next attempt)  
      ELSE  
      - Documents date-of-next-attempt-to-fix also considering the history of attempts  
  IF request was properly made, could be executed and result could be documented into CurrentAlarms  
    - Returns ResponseCode=204 (independently from the ResponseCode of MWDG)  
    ELSE  
    - Returns a ResponseCode that relates to problems with executing the p1-update-ltp-external-label  

### Concepts for defining ImplementationFunctions  
**Shaping**
During discussions we found out that:  
- With increasing number of deviations between RunningDS and OperationalDS some deadlock might occur.  
- It is unclear how roll-back of partly executed implementation sequences could be defined in case of idempotent functions.  

The following concepts should help minimizing the risk of dead lock and partly executed implementation sequences:  
- Implementation sequences should be short (this is why the information structure is now limiting to a single function).  
- Each implementation sequence shall terminate in a stable state more close to the target state defined in the RunningDS.  
- Steps that are increasing the options in the total system (e.g. releasing limited resources) shall be done first. Steps that are narrowing down the options in the total system (e.g. allocating resources) shall be done in a separated sequence later.  

**ResponseCode**
The results of the implementation attempts shall be documented into the CurrentAlarms.  
The ResponseCodes of the ImplementationFunctions shall relate to the processing of the request, not to the success or failure of the configuration task.  