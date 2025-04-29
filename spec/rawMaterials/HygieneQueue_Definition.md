# Hygiene Queue: Description and definition
## Overview
* The HygieneQueue is implemented as an ElasticSearch data base

* Each queue entry is defined by the set of parameters indicated below. 
Each queue entry consists of the following parameters:

    - LinkID
    - Mount NameA
    - Mount NameB
    - Planning Data:
        - Modem Card
        - Card Slot
        - ODU Product Code
        - Access Port
        - Upper Frequency
        - Lower Frequency
        - Band Position
        - Bandwidth
        - Minimum Modulation Scheme
        - Maximum Modulation Scheme
        - Clear Sky Modulation
    - Reliability Score
    - Priority Band

For exact schema definitions, refer to the provided json and yaml schemas.

## Priority bands

* Entries of the Hygiene Queue are grouped into priority bands, ensuring that higher-priority tasks are processed first while preventing starvation of lower-priority tasks.
Entries are assigned a **priority band** based on their **reliability score**:  

| Priority Band | Reliability Score | Cooldown Before Retry |
|--------------|------------------|----------------------|
| **High** | `< 30` | 3 cycles |
| **Medium** | `30 - 49` | 2 cycles |
| **Low** | `50 - 64` | 1 cycles |
| **Very Low** | `> 64` | 4 cycles |


## Processing Flow  
1. The system processes all tasks within a priority band before moving to the next band.  
2. Failed tasks enter a cooldown period based on their priority band before re-evaluation.  
3. New tasks are automatically categorized into priority bands when added to the queue (as their initial reliability score equals 0).  
4. Multiple tasks can be processed in parallel, controlled by a **sliding window configuration**.  