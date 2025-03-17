# Entity Keyword: `PAYLOAD`
> **Available in:**
> - Coreflux MQTT Broker >v1.5.1  
> - Linux ARM64 
> - Linux x64 
> - Windows x64 
## 1. Overview
- **Description:**  
  Represents the content or data that triggers an action upon the reception of a topic event. It specifically holds the payload of the triggering topic.

## 2. Signature
- **Syntax:**  
  ```lot
  PAYLOAD
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used in combination:
- **`SET`**: Assigns the PAYLOAD to a variable for reuse.
- **`PUBLISH TOPIC`**: Broadcasts the payload to another topic.
- **`KEEP TOPIC`**: Internally stores the payload.
- **`GET TOPIC`**: Retrieves additional topic data to use in conjunction with the PAYLOAD.
- **`IF`**: Performs conditional actions based on the PAYLOAD content.
- **`REPLACE`**: Modifies strings dynamically using the PAYLOAD.

## 4. Return Value
- **Description:**
  Provides the exact payload content of the triggering topic.

## 5. Usage Examples

### Basic Example
Publishes an incremented payload value when triggered:

```lot
DEFINE ACTION IncrementActionTriggered
ON TOPIC "input/topic" DO
     PUBLISH TOPIC "output/topic" WITH (PAYLOAD + 1)
```
- **Scenario Example:**
  - Received payload: `10`
  - Result: "output/topic" publishes `11`

### Intermediate Example
Checks a counter topic, initializes it if empty, and increments based on the PAYLOAD:

```lot
DEFINE ACTION CountTrigger
ON TOPIC "Plant/Equipments/+/di1" DO
    IF GET TOPIC "Plant/Machines/+/Production/Counter" EQUALS EMPTY THEN
        PUBLISH TOPIC "Plant/Machines/+/Production/Counter" WITH 0
    IF PAYLOAD == "True" THEN
        PUBLISH TOPIC "Plant/Machines/+/Production/Counter" WITH (GET TOPIC "Plant/Machines/+/Production/Counter" + 1)
```
- **Scenario Example:**
  - Received payload: `True`
  - Initial Counter: `5`
  - Result: Counter topic updated to `6`

### Advanced Example
Dynamically replaces a placeholder in a message with the PAYLOAD value:

```lot
DEFINE ACTION TestReplace
ON TOPIC "Selected/Machine"
DO
    SET "Message" WITH REPLACE "xx" WITH PAYLOAD IN "Machine xx was Selected!"
    PUBLISH TOPIC "Message" WITH {Message}
```
- **Scenario Example:**
  - Received payload: `12`
  - Result: "Message" topic publishes "Machine 12 was Selected!"

### Direct Usage with `PUBLISH` and `KEEP`
Directly integrates `REPLACE` and PAYLOAD into PUBLISH and KEEP operations:

```lot
DEFINE ACTION TestReplaceDirect
ON TOPIC "Selected/Machine"
DO
    PUBLISH TOPIC "MessageExternally" WITH REPLACE "xx" WITH PAYLOAD IN "Machine xx was Selected!"
    KEEP TOPIC "MessageInternally" WITH REPLACE "xx" WITH PAYLOAD IN "Machine xx was Selected!"
```
- **Scenario Example:**
  - Received payload: `07`
  - Result:
    - "MessageExternally" broadcasts "Machine 07 was Selected!"
    - "MessageInternally" stores "Machine 07 was Selected!" internally

## 6. Notes & Additional Information
- **Related Entities:**

  - [TIMESTAMP](../TIMESTAMP/TIMESTAMP.md)
  - [TOPIC](../TOPIC/TOPIC.md)

[Back to Entities](../Entities.md)

