---
title: Counter Action
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Persistent counter that increments on time intervals, demonstrating state management in LOT
related_tutorials: Tutorial/01-basics/timed-actions.lotnb
---

# Counter Action

## Overview

The Counter Action pattern demonstrates state persistence in LOT using `GET TOPIC` and `PUBLISH TOPIC`. It maintains a counter that survives between action executions, enabling stateful time-based operations without external storage.

## Pattern

```lot
DEFINE ACTION SimpleCounter
ON EVERY 1 SECOND DO
    PUBLISH TOPIC "counter/value" WITH (GET TOPIC "counter/value" + 1)
```

## How It Works

1. **Time Trigger**: Executes every second
2. **State Retrieval**: `GET TOPIC` retrieves the last published value
3. **Increment**: Adds 1 to the retrieved value
4. **State Persistence**: `PUBLISH TOPIC` stores the new value

### State Persistence Mechanism

- `GET TOPIC "counter/value"` returns the last published value
- If topic doesn't exist, returns 0 for numeric operations
- `PUBLISH TOPIC` stores the value for next execution
- State persists across action executions and system restarts (if MQTT broker persists topics)

## Expected MQTT Topics

### Output Topics
- `counter/value` - Current counter value (increments every second)

## Example Output

```
After 1 second: counter/value = 1
After 2 seconds: counter/value = 2
After 3 seconds: counter/value = 3
...
After 60 seconds: counter/value = 60
```

## Use Cases

- **Cycle Counting**: Track number of executions
- **Uptime Tracking**: Measure system runtime
- **Event Counting**: Count occurrences over time
- **Testing**: Verify action execution frequency

## Variations

### Counter with Reset at Limit

```lot
DEFINE ACTION CounterWithReset
ON EVERY 1 SECOND DO
    SET "current_count" WITH GET TOPIC "counter/reset_demo"
    SET "new_count" WITH ({current_count} + 1)
    
    IF {new_count} >= 100 THEN
        PUBLISH TOPIC "counter/reset_demo" WITH 0
        PUBLISH TOPIC "counter/reset_demo/status" WITH "Counter reset at 100!"
        PUBLISH TOPIC "counter/reset_demo/resets" WITH (GET TOPIC "counter/reset_demo/resets" + 1)
    ELSE
        PUBLISH TOPIC "counter/reset_demo" WITH {new_count}
```

**Demonstrates:** Conditional logic and automatic reset

### Multi-Counter System

```lot
DEFINE ACTION MultiCounter
ON EVERY 1 SECOND DO
    PUBLISH TOPIC "counters/seconds" WITH (GET TOPIC "counters/seconds" + 1)
    
    IF (GET TOPIC "counters/seconds" % 60) = 0 THEN
        PUBLISH TOPIC "counters/minutes" WITH (GET TOPIC "counters/minutes" + 1)
    
    IF (GET TOPIC "counters/minutes" % 60) = 0 THEN
        PUBLISH TOPIC "counters/hours" WITH (GET TOPIC "counters/hours" + 1)
```

**Demonstrates:** Hierarchical counting (seconds → minutes → hours)

### Counter with Statistics

```lot
DEFINE ACTION CounterWithStats
ON EVERY 1 SECOND DO
    SET "count" WITH (GET TOPIC "counter/value" + 1)
    SET "total_runtime" WITH (GET TOPIC "counter/runtime_minutes" + 0.0167)
    
    PUBLISH TOPIC "counter/value" WITH {count}
    PUBLISH TOPIC "counter/runtime_minutes" WITH {total_runtime}
    PUBLISH TOPIC "counter/rate_per_minute" WITH ({count} / {total_runtime})
    PUBLISH TOPIC "counter/last_update" WITH TIMESTAMP "UTC"
```

**Demonstrates:** Derived metrics from counter state

## Advanced Patterns

### Conditional Counter

```lot
DEFINE ACTION ConditionalCounter
ON EVERY 5 SECONDS DO
    SET "system_active" WITH GET TOPIC "system/status"
    
    IF {system_active} EQUALS "running" THEN
        SET "active_count" WITH (GET TOPIC "counter/active_time" + 5)
        PUBLISH TOPIC "counter/active_time" WITH {active_count}
    ELSE
        SET "idle_count" WITH (GET TOPIC "counter/idle_time" + 5)
        PUBLISH TOPIC "counter/idle_time" WITH {idle_count}
    
    PUBLISH TOPIC "counter/total_time" WITH (GET TOPIC "counter/total_time" + 5)
```

**Demonstrates:** Different counters based on system state

### Counter with Alerts

```lot
DEFINE ACTION CounterWithAlerts
ON EVERY 1 SECOND DO
    SET "count" WITH (GET TOPIC "counter/value" + 1)
    PUBLISH TOPIC "counter/value" WITH {count}
    
    IF {count} = 60 THEN
        PUBLISH TOPIC "alerts/counter" WITH "Counter reached 1 minute"
    ELSE IF {count} = 300 THEN
        PUBLISH TOPIC "alerts/counter" WITH "Counter reached 5 minutes"
    ELSE IF {count} = 3600 THEN
        PUBLISH TOPIC "alerts/counter" WITH "Counter reached 1 hour"
```

**Demonstrates:** Milestone-based notifications

## Related Patterns

- [Simple Heartbeat](./Simple-Heartbeat.md) - Basic periodic signaling
- [Production Simulator](./Production-Simulator.md) - Complex stateful operations

[Back to Actions](../DEFINE%20ACTION.md)

