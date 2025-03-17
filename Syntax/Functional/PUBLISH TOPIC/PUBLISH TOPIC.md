## Functional Keyword: `PUBLISH TOPIC`

### 1. Overview
- **Description:**  
  Broadcasts data to a specified topic, making it accessible to all subscribed clients. Topic names can be specified directly or dynamically through variables.

### 2. Signature
- **Syntax:**  
  ```lot
  PUBLISH TOPIC <"topic_name"|{variable_name}> WITH <FILTER|SPLIT|REPLACE|TRIM> <GET TOPIC | PAYLOAD | {variable_name}|(Expression)> USING <CSV|REGEX>
  ```

### 3. Compatible Keywords
List and description of each KEYWORD that can be used together:
- **`GET TOPIC`**: Retrieve existing topic data for republishing.
- **`KEEP TOPIC`**: Store data internally while simultaneously publishing it externally.
- **`IF`**: Enables conditional publishing based on specific criteria.
- **`EMPTY`**: Initializes topics by publishing payloads if the topic data is empty.
- **`SET`**: Dynamically defines or modifies topic names.

### 4. Return Value
- **Description:**  
  No explicit return value. Broadcasts the payload to all subscribed clients.

### 5. Usage Examples

#### Simple Example
Periodically publishes a system status message to the "system/status" topic:

```lot
DEFINE ACTION StatusUpdate
ON EVERY 10 SECONDS DO
    PUBLISH TOPIC "system/status" WITH "Online"
```
- **Output Example:** Every 10 seconds, subscribers of "system/status" will receive the message "Online".

#### Dynamic Topic Example
Publishes data to a dynamically determined topic. The topic's name is retrieved from another topic, "VarToUse". The retrieved topic name is then used to publish the timestamp from "MachineData/timestamp":

```lot
DEFINE ACTION ProvideExample
ON TOPIC "MachineData/payload"
DO
    SET "topicTimestamp" WITH GET TOPIC "VarToUse"
    PUBLISH TOPIC {topicTimestamp} WITH (GET TOPIC "MachineData/timestamp")
```
- **Example Scenario:**
  - Suppose "VarToUse" contains "ProvideExample/1/timestamp".
  - Suppose "MachineData/timestamp" contains "2025-03-15T12:30:00Z".
  - Result: The topic "ProvideExample/1/timestamp" will be published with the payload "2025-03-15T12:30:00Z", making this timestamp available to all subscribed clients.

### 6. Notes & Additional Information
- **Related Methods:**  
    
    - [GET TOPIC](../GET%20TOPIC/GET%20TOPIC.md)  
    - [KEEP TOPIC](../KEEP%20TOPIC/KEEP%20TOPIC.md)  

[Back to Functions](../Functional.md)
