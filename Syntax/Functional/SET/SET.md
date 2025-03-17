# Functional Keyword: `SET`

## 1. Overview
- **Description:**  
  Creates an internal variable within the broker to store data. This internal variable can be dynamically accessed or reused throughout actions by referencing it with `{variable_name}`.

## 2. Signature
- **Syntax:**  
  ```lot
    SET "variable_name" WITH <FILTER|SPLIT|REPLACE|TRIM> <GET TOPIC | PAYLOAD | {variable_name}|(Expression)> USING <CSV|REGEX>
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used together:
- **`GET TOPIC`**: Stores data retrieved from a specified topic into a variable.
- **`PAYLOAD`**: Stores incoming payload data into a variable.
- **`REPLACE`**: Dynamically modifies content before assigning it to a variable.
- **`PUBLISH TOPIC`**: Publishes content using the internally stored variable.
- **`KEEP TOPIC`**: Stores data internally using the variable's value.

## 4. Return Value
- **Description:**  
  No explicit return value. The specified content is stored in an internal variable for later use.

## 5. Usage Examples

### Simple Example
Assigns the value of an incoming payload from the topic `sensor/value` to an internal variable named `currentSensorValue`, then publishes it:

```lot
DEFINE ACTION StoreSensorValue
ON TOPIC "sensor/value"
DO
    SET "currentSensorValue" WITH PAYLOAD
    PUBLISH TOPIC "sensor/storedValue" WITH {currentSensorValue}
```
- **Example Scenario:**
  - Suppose the payload received from `sensor/value` is `23.5`.
  - Result: The topic `sensor/storedValue` publishes the payload `23.5`.

### Advanced Example with `REPLACE`
Creates a dynamic topic name based on payload data and assigns it to a variable:

```lot
DEFINE ACTION DynamicTopicAssignment
ON TOPIC "device/id"
DO
    SET "dynamicTopic" WITH REPLACE "xx" WITH PAYLOAD IN "devices/xx/status"
    PUBLISH TOPIC {dynamicTopic} WITH "active"
```
- **Example Scenario:**
  - Suppose the payload from `device/id` is `device123`.
  - Result: Publishes `"active"` to the dynamically created topic `devices/device123/status`.

## 6. Notes & Additional Information
- **Related Methods:**

  - [REPLACE](../REPLACE/REPLACE.md)
  - [GET TOPIC](../GET%20TOPIC/GET%20TOPIC.md)
  - [PUBLISH TOPIC](../PUBLISH%20TOPIC/PUBLISH%20TOPIC.md)
  - [KEEP TOPIC](../KEEP%20TOPIC/KEEP%20TOPIC.md)

[Back to Functions](../Functions.md)

