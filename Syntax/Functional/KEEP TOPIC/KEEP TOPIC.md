## Functional Keyword: `KEEP TOPIC`

### 1. Overview
- **Description:**  
  Stores data internally within the broker, making it accessible for internal processes without broadcasting it externally. Topic names can be specified directly or dynamically through variables.

### 2. Signature
- **Syntax:**  
  ```lot
  KEEP TOPIC "topic_name" WITH payload
  KEEP TOPIC {variable_name} WITH payload
  ```

### 3. Compatible Keywords
List and description of each KEYWORD that can be used together:
- **`GET TOPIC`**: Retrieve existing topic data for internal storage.
- **`PUBLISH TOPIC`**: Store data internally while simultaneously publishing it externally.
- **`IF`**: Enables conditional storage based on specific criteria.
- **`EMPTY`**: Initializes internal topics by storing payloads if the topic data is empty.
- **`SET`**: Dynamically defines or modifies topic names.

### 4. Return Value
- **Description:**  
  No explicit return value. Stores the payload internally within the broker.

### 5. Usage Examples

#### Simple Example
Periodically stores a system status message internally under the topic "system/status/internal":

```lot
DEFINE ACTION StatusInternalUpdate
ON EVERY 10 SECONDS DO
    KEEP TOPIC "system/status/internal" WITH "Online"
```
- **Output Example:** Every 10 seconds, the internal topic "system/status/internal" is updated with the message "Online".

#### Dynamic Topic Example
Stores data internally using a dynamically determined topic. The topic's name is retrieved from another topic, "VarToUse". The retrieved topic name is then used to store the timestamp from "MachineData/timestamp":

```lot
DEFINE ACTION ProvideExampleInternal
ON TOPIC "MachineData/payload"
DO
    SET "topicTimestamp" WITH GET TOPIC "VarToUse"
    KEEP TOPIC {topicTimestamp} WITH (GET TOPIC "MachineData/timestamp")
```
- **Example Scenario:**
  - Suppose "VarToUse" contains "ProvideExample/1/internalTimestamp".
  - Suppose "MachineData/timestamp" contains "2025-03-15T12:30:00Z".
  - Result: The topic "ProvideExample/1/internalTimestamp" will be stored internally with the payload "2025-03-15T12:30:00Z", accessible only to internal broker operations.

### 6. Notes & Additional Information
- **Related Methods:**  
  - [GET TOPIC](../GET%20TOPIC/GET%20TOPIC.md)  
  - [PUBLISH TOPIC](../PUBLISH%20TOPIC/PUBLISH%20TOPIC.md)  

[Back to Functions](../Functional.md)

