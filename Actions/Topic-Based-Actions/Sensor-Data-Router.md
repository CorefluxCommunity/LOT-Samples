---
title: Sensor Data Router
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Route sensor data using wildcards to extract sensor IDs and create structured topic hierarchies
related_tutorials: Tutorial/01-basics/topic-actions.lotnb
---

# Sensor Data Router

## Overview

The Sensor Data Router pattern demonstrates how to use wildcard topics (`+`) to match multiple sensors and extract context from the topic structure using `TOPIC POSITION`. This enables scalable, context-aware data routing for any number of sensors.

## Pattern

```lot
DEFINE ACTION SensorDataRouter
ON TOPIC "sensors/+/temperature" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    
    PUBLISH TOPIC "processed/temperature/" + {sensor_id} WITH PAYLOAD
    PUBLISH TOPIC "processed/temperature/" + {sensor_id} + "/timestamp" WITH TIMESTAMP "UTC"
```

## How It Works

1. **Wildcard Matching**: `sensors/+/temperature` matches any sensor ID in the middle position
2. **Context Extraction**: `TOPIC POSITION 1` extracts the sensor ID (0-indexed positions)
3. **Dynamic Routing**: Routes data to sensor-specific processed topics
4. **Timestamp Addition**: Adds timestamps for data tracking

## Topic Position Indexing

For topic `sensors/temp001/temperature`:
- `TOPIC POSITION 0` = `"sensors"`
- `TOPIC POSITION 1` = `"temp001"`
- `TOPIC POSITION 2` = `"temperature"`

## Expected MQTT Topics

### Input Topics (Wildcard Pattern)
- `sensors/temp001/temperature` - Temperature from sensor temp001
- `sensors/temp002/temperature` - Temperature from sensor temp002
- `sensors/pressXYZ/temperature` - Temperature from any sensor

### Output Topics (Dynamic)
- `processed/temperature/{sensor_id}` - Processed temperature reading
- `processed/temperature/{sensor_id}/timestamp` - Processing timestamp

## Example Payloads

**Publish to:** `sensors/temp001/temperature`  
**Payload:** `25.5`

**Results:**
- `processed/temperature/temp001`: `25.5`
- `processed/temperature/temp001/timestamp`: `"2025-10-25T14:30:15Z"`

## Use Cases

- **Multi-Sensor Systems**: Handle any number of sensors with one action
- **Data Normalization**: Transform raw sensor topics to standardized structure
- **Timestamp Tracking**: Add processing timestamps for audit trails
- **Topic Namespace Mapping**: Route from raw to processed topic hierarchies

## Variations

### Multi-Level Sensor Hierarchy

```lot
DEFINE ACTION MultiLevelSensorRouter
ON TOPIC "factory/+/+/temperature" DO
    SET "line_id" WITH TOPIC POSITION 1
    SET "machine_id" WITH TOPIC POSITION 2
    
    PUBLISH TOPIC "processed/" + {line_id} + "/" + {machine_id} + "/temperature" WITH PAYLOAD
    PUBLISH TOPIC "processed/" + {line_id} + "/" + {machine_id} + "/timestamp" WITH TIMESTAMP "UTC"
```

**Matches topics like:** `factory/line1/machine5/temperature`

### Router with Data Transformation

```lot
DEFINE ACTION TemperatureConverter
ON TOPIC "sensors/+/temperature/celsius" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    
    SET "fahrenheit" WITH (PAYLOAD * 9 / 5 + 32)
    
    PUBLISH TOPIC "sensors/" + {sensor_id} + "/temperature/fahrenheit" WITH {fahrenheit}
    PUBLISH TOPIC "sensors/" + {sensor_id} + "/temperature/both" WITH "C: " + PAYLOAD + ", F: " + {fahrenheit}
```

**Demonstrates:** Unit conversion during routing

## Related Patterns

- [Simple Echo Action](./Simple-Echo-Action.md) - Basic message forwarding
- [Alarm Processor](./Alarm-Processor.md) - Processing with conditional logic
- [Command Processor](./Command-Processor.md) - Bidirectional communication patterns

[Back to Actions](../DEFINE%20ACTION.md)

