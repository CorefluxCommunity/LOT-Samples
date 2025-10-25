---
title: Factory Alarm Processor
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Process alarms from complex topic hierarchies with context extraction and multi-level routing
related_tutorials: Tutorial/01-basics/topic-actions.lotnb
---

# Factory Alarm Processor

## Overview

The Factory Alarm Processor pattern demonstrates advanced topic-based processing with multi-level wildcards, context extraction, and hierarchical data routing. It processes alarms from factory equipment and creates rich contextual alarm messages.

## Pattern

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

## How It Works

1. **Multi-Level Wildcard**: Matches `factory/LINE/MACHINE/alarm` structure
2. **Context Extraction**: Extracts both line_id and machine_id from topic
3. **Context Creation**: Builds human-readable alarm context
4. **Hierarchical Routing**: Routes to both specific and global alarm topics
5. **Dashboard Integration**: Sends to central dashboard for monitoring

## Topic Position Indexing

For topic `factory/line1/machine5/alarm`:
- `TOPIC POSITION 0` = `"factory"`
- `TOPIC POSITION 1` = `"line1"` → extracted as line_id
- `TOPIC POSITION 2` = `"machine5"` → extracted as machine_id
- `TOPIC POSITION 3` = `"alarm"`

## Expected MQTT Topics

### Input Topics (Wildcard Pattern)
- `factory/line1/machine5/alarm`
- `factory/line2/machine3/alarm`
- `factory/+/+/alarm` - Matches any line and machine

### Output Topics (Dynamic)
- `alarms/factory/{line_id}/{machine_id}` - Alarm payload
- `alarms/factory/{line_id}/{machine_id}/context` - Contextual information
- `alarms/factory/{line_id}/{machine_id}/timestamp` - Alarm timestamp
- `dashboard/alarms/latest` - Global dashboard feed

## Example Payloads

**Publish to:** `factory/line1/machine5/alarm`  
**Payload:** `"Overheating detected"`

**Results:**
- `alarms/factory/line1/machine5`: `"Overheating detected"`
- `alarms/factory/line1/machine5/context`: `"Line line1 Machine machine5"`
- `alarms/factory/line1/machine5/timestamp`: `"2025-10-25T14:30:15Z"`
- `dashboard/alarms/latest`: `"Line line1 Machine machine5: Overheating detected"`

## Use Cases

- **Hierarchical Alarm Management**: Process alarms from complex facility structures
- **Context Enrichment**: Add location and equipment context to alarms
- **Central Monitoring**: Aggregate alarms to central dashboard
- **Audit Trails**: Track alarm source and timing

## Variations

### Alarm with Severity Classification

```lot
DEFINE ACTION AlarmWithSeverity
ON TOPIC "factory/+/+/alarm" DO
    SET "line_id" WITH TOPIC POSITION 1
    SET "machine_id" WITH TOPIC POSITION 2
    
    SET "severity" WITH IF PAYLOAD CONTAINS "critical" THEN "CRITICAL" 
                       ELSE IF PAYLOAD CONTAINS "warning" THEN "WARNING" 
                       ELSE "INFO"
    
    IF {severity} EQUALS "CRITICAL" THEN
        PUBLISH TOPIC "alarms/critical/" + {line_id} + "/" + {machine_id} WITH PAYLOAD
        PUBLISH TOPIC "notifications/sms" WITH "CRITICAL: " + {line_id} + "/" + {machine_id} + " - " + PAYLOAD
    
    PUBLISH TOPIC "alarms/all/" + {severity} + "/" + {line_id} + "/" + {machine_id} WITH PAYLOAD
```

**Demonstrates:** Conditional routing based on alarm content

### Alarm Counter

```lot
DEFINE ACTION AlarmCounter
ON TOPIC "factory/+/+/alarm" DO
    SET "line_id" WITH TOPIC POSITION 1
    SET "machine_id" WITH TOPIC POSITION 2
    
    SET "alarm_count" WITH (GET TOPIC "alarms/count/" + {line_id} + "/" + {machine_id} + 1)
    PUBLISH TOPIC "alarms/count/" + {line_id} + "/" + {machine_id} WITH {alarm_count}
    
    PUBLISH TOPIC "alarms/factory/" + {line_id} + "/" + {machine_id} WITH PAYLOAD
    PUBLISH TOPIC "alarms/factory/" + {line_id} + "/" + {machine_id} + "/count" WITH {alarm_count}
```

**Demonstrates:** State tracking with alarm counting

## Related Patterns

- [Sensor Data Router](./Sensor-Data-Router.md) - Multi-level hierarchy routing
- [Command Processor](./Command-Processor.md) - Bidirectional communication
- [Simple Echo Action](./Simple-Echo-Action.md) - Basic message forwarding

[Back to Actions](../DEFINE%20ACTION.md)

