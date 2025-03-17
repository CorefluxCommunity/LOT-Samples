# Entity Keyword: `TOPIC`
> **Available in:**
> - Coreflux MQTT Broker >v1.4.4  
> - Linux ARM64 
> - Linux x64 
> - Windows x64 
## 1. Overview
- **Description:**  
  Represents an MQTT topic within the broker, specifying the address or channel through which data is published or received. Topics organize data streams and facilitate communication between clients and the broker.

## 2. Signature
- **Syntax:**  
  ```lot
  TOPIC
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used in combination:
- **`SET`**: Assigns a dynamically created topic address to a variable.
- **`PUBLISH TOPIC`**: Publishes data to the specified MQTT topic.
- **`KEEP TOPIC`**: Stores data internally associated with the specified MQTT topic.
- **`GET TOPIC`**: Retrieves data from the specified MQTT topic.
- **`IF`**: Performs conditional logic based on topic data or topic existence.
- **`REPLACE`**: Dynamically modifies topic strings.

## 4. Return Value
- **Description:**
  Provides the specific MQTT topic address used within an action or model.

## 5. Usage Examples

### Basic Example
Periodically publishes data to a fixed MQTT topic:

```lot
DEFINE ACTION PublishStatus
ON EVERY 5 SECONDS DO
    PUBLISH TOPIC "system/status" WITH "running"
```
- **Scenario Example:**
  - Result: Every 5 seconds, the topic "system/status" broadcasts the message "running".

### Intermediate Example
Uses conditional logic to publish alerts based on topic data:

```lot
DEFINE ACTION AlertIfOffline
ON EVERY 10 SECONDS DO
    IF GET TOPIC "device/status" EQUALS "offline" THEN
        PUBLISH TOPIC "alerts" WITH "Device offline!"
```
- **Scenario Example:**
  - Topic "device/status" contains: "offline"
  - Result: Topic "alerts" publishes "Device offline!"

### Advanced Example
Creates and uses dynamic topic addresses:

```lot
DEFINE ACTION DynamicTopic
ON TOPIC "devices/+/id"
DO
    SET "dynamicAddress" WITH REPLACE "+" WITH PAYLOAD IN "devices/+/data"
    PUBLISH TOPIC {dynamicAddress} WITH "Device data updated."
```
- **Scenario Example:**
  - Received payload on "devices/device123/id": "device123"
  - Result: "devices/device123/data" publishes "Device data updated."

### Direct Usage with `PUBLISH` and `KEEP`
Directly integrates dynamic topic manipulation into PUBLISH and KEEP operations:

```lot
DEFINE ACTION DirectPublishKeep
ON TOPIC "sensor/id"
DO
    PUBLISH TOPIC REPLACE "id" WITH PAYLOAD IN "sensor/id/status" WITH "active"
    KEEP TOPIC REPLACE "id" WITH PAYLOAD IN "sensor/id/status" WITH "active"
```
- **Scenario Example:**
  - Received payload: `sensor456`
  - Result:
    - "sensor/sensor456/status" broadcasts "active"
    - "sensor/sensor456/status" stores "active" internally

## 6. Notes & Additional Information
- **Related Entities:**

  - [PAYLOAD](../PAYLOAD/PAYLOAD.md)
  - [TIMESTAMP](../TIMESTAMP/TIMESTAMP.md)
  - [GET TOPIC](../../Functional/GET%20TOPIC/GET%20TOPIC.md)  
  - [PUBLISH TOPIC](../../Functional/PUBLISH%20TOPIC/PUBLISH%20TOPIC.md)  

[Back to Entities](../Entities.md)

