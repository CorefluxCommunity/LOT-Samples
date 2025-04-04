# Functional Keyword: `GET TOPIC`

> **Available in:**
> - Coreflux MQTT Broker >v1.4.6  
> - Linux ARM64 
> - Linux x64 
> - Windows x64 

## 1. Overview
- **Description:**  
  The primary method to retrieve data from a specified topic, whether it is a Published or Kept Topic.
  



## 2. Signature
- **Syntax:**  
  ```lot
  GET TOPIC <"topic_name" | {variable_name}> AS {INT|DOUBLE|STRING|TIMESTAMP|BOOL}
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used in conjunction:
- **`PUBLISH TOPIC`**: Can be used with GET TOPIC to copy data from one topic to another.
- **`KEEP TOPIC`**: Stores topic data internally without broadcasting, ideal for local memory usage.
- **`IF`**: Useful for conditional operations based on topic data.
- **`EMPTY`**: Checks if the specified topic has ever been used or contains data.

## 4. Return Value
- **Description:** Returns the last payload known to the broker for the specified topic.

## 5. Usage Examples

### Simple Example
Retrieving data from topic "Teste" and comparing it to the value 42. If the condition is true, another topic, "Meaning/of/life", is published with the payload "found".

```lot
DEFINE ACTION TestGetTopic
ON EVERY 5 SECONDS DO
    IF (GET TOPIC "Teste" EQUALS 42) THEN
        PUBLISH TOPIC "Meaning/of/life" WITH "found"
```

### Intermediate Example
When data arrives on "MachineData/data", the action checks if the topic "MachineData/id" equals "1". If true, it publishes one topic externally and stores another topic internally.

Note: `KEEP TOPIC` stores data locally (CPU/Memory), while `PUBLISH TOPIC` broadcasts data externally (CPU/Memory/Network ).

```lot
DEFINE ACTION MachineData
ON TOPIC "MachineData/data"
DO
    IF (GET TOPIC "MachineData/id" EQUALS "1") THEN
        KEEP TOPIC "Current/timestamp" WITH (GET TOPIC "MachineData/timestamp")
        PUBLISH TOPIC "Show/timestamp" WITH (GET TOPIC "MachineData/timestamp")
```

### Advanced Example
Verifies if the topic "test/10seconds" is empty, initializing it with 0 if true. Each subsequent invocation (every 10 seconds) increments the value by 1, resetting it to 0 if it exceeds 10. Demonstrates operational capabilities.

```lot
DEFINE ACTION IncrementAction10Seconds
ON EVERY 10 SECONDS DO
    IF GET TOPIC "test/10seconds" == EMPTY THEN
        PUBLISH TOPIC "test/10seconds" WITH 0
    ELSE
        IF GET TOPIC "test/10seconds" > 10 THEN
            PUBLISH TOPIC "test/10seconds" WITH 0
        ELSE
            PUBLISH TOPIC "test/10seconds" WITH (GET TOPIC "test/10seconds" + 1)
```

## 6. Notes & Additional Information
- **Related Methods:**

  - [PUBLISH TOPIC](../PUBLISH%20TOPIC/PUBLISH%20TOPIC.md)
  - [KEEP TOPIC](../KEEP%20TOPIC/KEEP%20TOPIC.md)

[Back to Functions](../Functional.md)

