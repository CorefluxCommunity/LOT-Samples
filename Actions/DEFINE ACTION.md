# KEYWORD: `DEFINE ACTION`

## 1. Overview
- **Description:**  
  Defines executable logic that is triggered by specific events such as timed intervals, topic updates, or explicit calls from other actions or system components. Actions allow dynamic interaction with MQTT topics, internal variables, and payloads, facilitating complex IoT automation workflows.

## 2. Signature
- **Syntax:**  
  ```lot
  DEFINE ACTION <action_name>
  ON <EVENT TRIGGER>
  DO
      <action_logic>
  ```
  *Examples:*  
  ```lot
  DEFINE ACTION IncrementCounter
  ON TOPIC "sensor/trigger" DO
      PUBLISH TOPIC "sensor/counter" WITH (GET TOPIC "sensor/counter" + 1)

  DEFINE ACTION PeriodicCheck
  ON EVERY 5 SECONDS DO
      IF GET TOPIC "device/status" EQUALS "offline" THEN
          PUBLISH TOPIC "alerts" WITH "Device is offline!"
  ```

## 3. Compatible Keywords
List and describe each keyword that can be used within `DEFINE ACTION`:
- **`ON EVERY`** (*Event Trigger*):  
  Triggers the action at regular intervals (seconds, minutes, hours).
- **`ON TOPIC`** (*Event Trigger*):  
  Triggers the action when a specified topic receives data.
- **`SET`** (*Keyword*):  
  Defines or assigns values to internal variables.
- **`PUBLISH TOPIC`** (*Keyword*):  
  Broadcasts data to MQTT subscribers.
- **`KEEP TOPIC`** (*Keyword*):  
  Stores data internally within the broker.
- **`GET TOPIC`** (*Keyword*):  
  Retrieves current topic data for processing.
- **`IF ... THEN ... ELSE`** (*Control Structure*):  
  Implements conditional logic.
- **`REPLACE`** (*Keyword*):  
  Performs dynamic text replacements based on variables or payloads.
- **`PAYLOAD`** (*Entity*):  
  Accesses the payload of the triggering event.

## 4. Return Value
- **Description:**  
  Actions do not return explicit values. They execute defined logic to interact with topics, manage data internally, or trigger further events.

## 5. Usage Examples

### 5.1 Timed Action Example
Executes every 10 seconds, checking a status and publishing a message if conditions are met:

```lot
DEFINE ACTION StatusCheck
ON EVERY 10 SECONDS DO
    IF GET TOPIC "device/status" EQUALS "inactive" THEN
        PUBLISH TOPIC "device/alerts" WITH "Device inactive!"
```

### 5.2 Topic-Triggered Action Example
Triggers whenever a sensor topic receives a new payload:

```lot
DEFINE ACTION SensorUpdate
ON TOPIC "sensor/temperature" DO
    IF PAYLOAD > 30 THEN
        PUBLISH TOPIC "alerts" WITH "Temperature is too high!"
```

### 5.3 Callable Action Example
An action defined without an event trigger, callable from other actions:

```lot
DEFINE ACTION LogData
DO
    KEEP TOPIC "log/data" WITH (GET TOPIC "sensor/latest")
```
Can be called by publishing in the `$SYS/Coreflux/Command` and publish `-runAction <action_name>`


## 6. Notes & Additional Information
- **Callable Actions:**  
  Actions without event triggers are callable actions and can be invoked directly from other actions or external systems like internal routers or MCP servers.

- **Trigger Types:**
  - **Time-based Triggers** (`ON EVERY`): Execute periodically.
  - **Topic-based Triggers** (`ON TOPIC`): Execute on topic data reception.
  - **Callable Actions** (No explicit trigger): Manually called within other actions or processes.

- **Related Keywords & Entities:**
  - [DEFINE MODEL](../Models/MODEL.md)
  - [SET](../Syntax/Functional/SET/SET.md)
  - [GET TOPIC](../Syntax/Functional/GET%20TOPIC/GET%20TOPIC.md)
  - [PAYLOAD](../Syntax/Entities/PAYLOAD/PAYLOAD.md)

[Back to Main](../README.md)