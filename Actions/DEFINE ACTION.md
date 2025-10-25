---
title: DEFINE ACTION
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Define executable logic triggered by events such as timed intervals, topic updates, or explicit calls
related_tutorials: Tutorial/01-basics/timed-actions.lotnb, Tutorial/01-basics/topic-actions.lotnb
---

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
  DEFINE ACTION SimpleCounter
  ON TOPIC "sensor/trigger" DO
      PUBLISH TOPIC "sensor/counter" WITH (GET TOPIC "sensor/counter" + 1)

  DEFINE ACTION PeriodicHeartbeat
  ON EVERY 5 SECONDS DO
      PUBLISH TOPIC "system/heartbeat" WITH "alive"
  ```

## 3. Action Types

### 3.1 Time-Based Actions
Actions triggered at regular intervals using `ON EVERY`:

```lot
DEFINE ACTION SimpleHeartbeat
ON EVERY 5 SECONDS DO
    PUBLISH TOPIC "system/heartbeat" WITH "alive"
```

**Supported Time Units:**
- `SECOND` / `SECONDS`
- `MINUTE` / `MINUTES`
- `HOUR` / `HOURS`
- `DAY` / `DAYS`
- `WEEK` / `WEEKS`

**Use Cases:**
- System heartbeat signals
- Periodic monitoring and reporting
- Scheduled maintenance tasks
- Regular status updates

[Learn more about Time-Based Actions →](./Time-Based-Actions/)

### 3.2 Topic-Based Actions
Actions triggered when MQTT topics receive data using `ON TOPIC`:

```lot
DEFINE ACTION SensorDataRouter
ON TOPIC "sensors/+/temperature" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    PUBLISH TOPIC "processed/temperature/" + {sensor_id} WITH PAYLOAD
```

**Topic Patterns:**
- Exact topics: `"sensor/temperature"`
- Single-level wildcards: `"sensors/+/temperature"`
- Multi-level wildcards: `"factory/+/+/status"`

**Use Cases:**
- Real-time data processing
- Event-driven automation
- Message routing and transformation
- Context-aware responses

[Learn more about Topic-Based Actions →](./Topic-Based-Actions/)

### 3.3 Callable Actions
Actions without event triggers, callable from other actions or external systems:

```lot
DEFINE ACTION LogData
DO
    KEEP TOPIC "log/data" WITH (GET TOPIC "sensor/latest")
```

**Invocation:**
Can be called by publishing to `$SYS/Coreflux/Command` with payload `-runAction <action_name>`

**Use Cases:**
- Utility functions
- Reusable logic blocks
- Manual interventions
- Testing and debugging

## 4. Compatible Keywords

Keywords that can be used within `DEFINE ACTION`:

- **`ON EVERY`** (*Event Trigger*): Triggers the action at regular intervals
- **`ON TOPIC`** (*Event Trigger*): Triggers when a specified topic receives data
- **`SET`** (*Keyword*): Defines or assigns values to internal variables
- **`PUBLISH TOPIC`** (*Keyword*): Broadcasts data to MQTT subscribers
- **`KEEP TOPIC`** (*Keyword*): Stores data internally within the broker
- **`GET TOPIC`** (*Keyword*): Retrieves current topic data for processing
- **`GET JSON`** (*Keyword*): Extracts fields from JSON payloads with type casting
- **`IF ... THEN ... ELSE`** (*Control Structure*): Implements conditional logic
- **`REPLACE`** (*Keyword*): Performs dynamic text replacements
- **`PAYLOAD`** (*Entity*): Accesses the payload of the triggering event
- **`TOPIC POSITION`** (*Entity*): Extracts parts of topic paths from wildcards
- **`TIMESTAMP`** (*Entity*): Generates timestamps in various formats
- **`CALL PYTHON`** (*Keyword*): Executes Python functions for complex processing

## 5. Usage Examples

### 5.1 Time-Based Example: Production Line Simulator

```lot
DEFINE ACTION ProductionLineSimulator
ON EVERY 30 SECONDS DO
    SET "parts_produced" WITH (GET TOPIC "factory/line1/parts_produced" + 5)
    SET "total_runtime" WITH (GET TOPIC "factory/line1/runtime_minutes" + 0.5)
    
    PUBLISH TOPIC "factory/line1/parts_produced" WITH {parts_produced}
    PUBLISH TOPIC "factory/line1/runtime_minutes" WITH {total_runtime}
    PUBLISH TOPIC "factory/line1/last_update" WITH TIMESTAMP "UTC"
    
    SET "production_rate" WITH (GET TOPIC "factory/line1/parts_produced" AS DOUBLE / GET TOPIC "factory/line1/runtime_minutes" AS DOUBLE)
    PUBLISH TOPIC "factory/line1/production_rate" WITH {production_rate}
```

**What this does:**
- Simulates production every 30 seconds
- Tracks parts produced and runtime
- Calculates production rate
- Updates timestamps

### 5.2 Topic-Based Example: Sensor Data Router

```lot
DEFINE ACTION SensorDataRouter
ON TOPIC "sensors/+/temperature" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    
    PUBLISH TOPIC "processed/temperature/" + {sensor_id} WITH PAYLOAD
    PUBLISH TOPIC "processed/temperature/" + {sensor_id} + "/timestamp" WITH TIMESTAMP "UTC"
```

**What this does:**
- Listens to any sensor's temperature topic
- Extracts sensor ID from topic structure
- Routes data to processed topics
- Adds timestamps

### 5.3 Topic-Based with Conditional Logic

```lot
DEFINE ACTION FactoryAlarmProcessor
ON TOPIC "factory/+/+/alarm" DO
    SET "line_id" WITH TOPIC POSITION 1
    SET "machine_id" WITH TOPIC POSITION 2
    
    SET "alarm_context" WITH "Line " + {line_id} + " Machine " + {machine_id}
    
    PUBLISH TOPIC "alarms/factory/" + {line_id} + "/" + {machine_id} WITH PAYLOAD
    PUBLISH TOPIC "alarms/factory/" + {line_id} + "/" + {machine_id} + "/context" WITH {alarm_context}
    PUBLISH TOPIC "alarms/factory/" + {line_id} + "/" + {machine_id} + "/timestamp" WITH TIMESTAMP "UTC"
    
    PUBLISH TOPIC "dashboard/alarms/latest" WITH {alarm_context} + ": " + PAYLOAD
```

**What this does:**
- Processes alarms from factory hierarchy
- Extracts line and machine IDs
- Creates contextual alarm messages
- Routes to both specific and global topics

## 6. Python Integration

Actions can call Python functions for complex processing:

```lot
DEFINE ACTION DataValidationProcessor
ON TOPIC "validation/+/check" DO
    SET "validation_type" WITH TOPIC POSITION 1
    
    CALL PYTHON "DataValidator.validate_sensor_reading"
        WITH (PAYLOAD, GET TOPIC "validation/sensor/min", GET TOPIC "validation/sensor/max")
        RETURN AS {validation_result}
    
    SET "is_valid" WITH (GET JSON "valid" IN {validation_result} AS BOOL)
    IF {is_valid} EQUALS TRUE THEN
        PUBLISH TOPIC "validation/sensor/passed" WITH {validation_result}
    ELSE
        PUBLISH TOPIC "validation/sensor/failed" WITH {validation_result}
```

[Learn more about Python Actions →](./Python-Actions/)

## 7. Notes & Additional Information

### Trigger Types
- **Time-based Triggers** (`ON EVERY`): Execute periodically based on time intervals
- **Topic-based Triggers** (`ON TOPIC`): Execute on topic data reception with wildcard support
- **Callable Actions** (No explicit trigger): Manually invoked from other actions or external systems

### State Management
- Use `GET TOPIC` to retrieve previously stored values
- Use `PUBLISH TOPIC` to store state for future executions
- Variables created with `SET` exist only within the action execution
- State persists between executions via MQTT topics

### Performance Considerations
- Time-based actions run independently of message traffic
- Topic-based actions scale with message volume
- Use wildcards strategically to avoid excessive triggers
- Consider action execution time for high-frequency events

## 8. Future Extensions

<!-- Future Integration: Python callable example block -->
```python
# Example: Advanced data processing with Python
# Script Name: AdvancedProcessor
def process_complex_data(json_data):
    import json
    data = json.loads(json_data)
    # Complex processing logic
    return processed_result
```
<!-- Compatible with VS Code LOT Notebook syntax highlighting -->

### Planned Features
- Machine learning model integration
- Advanced statistical analysis
- External API interactions
- Complex event processing patterns

## 9. Related Keywords & Entities

- [DEFINE MODEL](../Models/DEFINE%20MODEL.md)
- [DEFINE ROUTE](../Routes/DEFINE%20ROUTE.md)
- [SET](../Syntax/Functional/SET/SET.md)
- [GET TOPIC](../Syntax/Functional/GET%20TOPIC/GET%20TOPIC.md)
- [PUBLISH TOPIC](../Syntax/Functional/PUBLISH%20TOPIC/PUBLISH%20TOPIC.md)
- [PAYLOAD](../Syntax/Entities/PAYLOAD/PAYLOAD.md)
- [TIMESTAMP](../Syntax/Entities/TIMESTAMP/TIMESTAMP.md)
- [TOPIC](../Syntax/Entities/TOPIC/TOPIC.md)

[Back to Main](../README.md)
