---
title: Simple Heartbeat Action
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Periodic heartbeat signal to monitor system health and availability
related_tutorials: Tutorial/01-basics/timed-actions.lotnb
---

# Simple Heartbeat Action

## Overview

A heartbeat action is the most fundamental time-based pattern in LOT. It periodically publishes a signal to confirm that the system is alive and operational. This is essential for monitoring, health checks, and detecting system failures.

## Pattern

```lot
DEFINE ACTION SimpleHeartbeat
ON EVERY 5 SECONDS DO
    PUBLISH TOPIC "system/heartbeat" WITH "alive"
```

## How It Works

1. **Time Trigger**: Executes automatically every 5 seconds
2. **Signal Publication**: Publishes "alive" message
3. **Continuous Operation**: Runs indefinitely without external triggers

## Time Unit Options

```lot
ON EVERY 1 SECOND   // High-frequency monitoring
ON EVERY 30 SECONDS // Standard heartbeat
ON EVERY 5 MINUTES  // Regular check-ins
ON EVERY 1 HOUR     // Hourly status
ON EVERY 1 DAY      // Daily reports
```

## Expected MQTT Topics

### Output Topics
- `system/heartbeat` - Regular heartbeat signal

## Example Output

Every 5 seconds:
```
system/heartbeat: "alive"
```

## Use Cases

- **System Monitoring**: Detect if LOT system is running
- **Health Checks**: Verify broker and action execution
- **Connection Verification**: Confirm MQTT connectivity
- **Watchdog Integration**: Feed external monitoring systems

## Variations

### Heartbeat with Timestamp

```lot
DEFINE ACTION HeartbeatWithTimestamp
ON EVERY 10 SECONDS DO
    PUBLISH TOPIC "system/heartbeat/status" WITH "alive"
    PUBLISH TOPIC "system/heartbeat/timestamp" WITH TIMESTAMP "UTC"
    PUBLISH TOPIC "system/heartbeat/unix" WITH TIMESTAMP "UNIX"
```

**Output Topics:**
- `system/heartbeat/status`: `"alive"`
- `system/heartbeat/timestamp`: `"2025-10-25T14:30:15Z"`
- `system/heartbeat/unix`: `1729864215`

### Heartbeat with Counter

```lot
DEFINE ACTION HeartbeatWithCounter
ON EVERY 5 SECONDS DO
    SET "count" WITH (GET TOPIC "system/heartbeat/count" + 1)
    
    PUBLISH TOPIC "system/heartbeat" WITH "alive"
    PUBLISH TOPIC "system/heartbeat/count" WITH {count}
    PUBLISH TOPIC "system/heartbeat/timestamp" WITH TIMESTAMP "UTC"
```

**Demonstrates:** Tracking number of heartbeat cycles

### Multi-System Heartbeat

```lot
DEFINE ACTION MultiSystemHeartbeat
ON EVERY 10 SECONDS DO
    PUBLISH TOPIC "system/edge/heartbeat" WITH "alive"
    PUBLISH TOPIC "system/actions/heartbeat" WITH "active"
    PUBLISH TOPIC "system/timestamp" WITH TIMESTAMP "UTC"
```

**Demonstrates:** Multiple status signals in one action

## Monitoring Integration

### Detecting Failures

External monitoring can detect system failure if heartbeat stops:

```python
# Pseudocode for external monitor
last_heartbeat = mqtt.subscribe("system/heartbeat")
if time_since(last_heartbeat) > 15_seconds:
    alert("System heartbeat lost - possible failure")
```

### Dashboard Integration

```lot
DEFINE ACTION DashboardHeartbeat
ON EVERY 30 SECONDS DO
    SET "uptime_minutes" WITH (GET TOPIC "system/uptime_minutes" + 0.5)
    
    PUBLISH TOPIC "dashboard/status/alive" WITH TRUE
    PUBLISH TOPIC "dashboard/status/uptime" WITH {uptime_minutes}
    PUBLISH TOPIC "dashboard/status/last_seen" WITH TIMESTAMP "UTC"
```

## Related Patterns

- [Counter Action](./Counter-Action.md) - State management with timed increments
- [Production Simulator](./Production-Simulator.md) - Complex periodic processing

[Back to Actions](../DEFINE%20ACTION.md)

