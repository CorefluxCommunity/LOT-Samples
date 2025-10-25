---
title: Production Line Simulator
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Realistic production simulation with metrics, calculations, and status tracking
related_tutorials: Tutorial/01-basics/timed-actions.lotnb
---

# Production Line Simulator

## Overview

The Production Line Simulator demonstrates a realistic time-based action that models manufacturing operations. It tracks production metrics, calculates rates, maintains state, and provides comprehensive status updates—showcasing advanced patterns for industrial IoT applications.

## Pattern

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
    
    PUBLISH TOPIC "factory/line1/status" WITH "Running - Produced " + {parts_produced} + " parts"
```

## How It Works

1. **Periodic Execution**: Runs every 30 seconds to simulate production cycles
2. **State Management**: Tracks parts produced and runtime using GET/PUBLISH TOPIC
3. **Metric Calculation**: Computes production rate (parts per minute)
4. **Status Updates**: Provides comprehensive operational status
5. **Type Casting**: Uses `AS DOUBLE` for accurate rate calculations

## Expected MQTT Topics

### Output Topics
- `factory/line1/parts_produced` - Total parts produced (increments by 5 every 30s)
- `factory/line1/runtime_minutes` - Total runtime in minutes (increments by 0.5)
- `factory/line1/last_update` - Timestamp of last update
- `factory/line1/production_rate` - Calculated parts per minute
- `factory/line1/status` - Human-readable status message

## Example Output

```
After 30 seconds:
  factory/line1/parts_produced: 5
  factory/line1/runtime_minutes: 0.5
  factory/line1/production_rate: 10.0
  factory/line1/status: "Running - Produced 5 parts"

After 60 seconds:
  factory/line1/parts_produced: 10
  factory/line1/runtime_minutes: 1.0
  factory/line1/production_rate: 10.0
  factory/line1/status: "Running - Produced 10 parts"
```

## Use Cases

- **Production Simulation**: Test dashboards and monitoring systems
- **Training Environments**: Demonstrate LOT capabilities with realistic data
- **System Testing**: Generate consistent data streams for testing
- **Metric Calculation**: Show advanced calculation patterns

## Variations

### Multi-Line Production System

```lot
DEFINE ACTION MultiLineProduction
ON EVERY 30 SECONDS DO
    // Line 1 - Fast production
    SET "line1_parts" WITH (GET TOPIC "factory/line1/parts" + 8)
    PUBLISH TOPIC "factory/line1/parts" WITH {line1_parts}
    PUBLISH TOPIC "factory/line1/rate" WITH 16.0
    
    // Line 2 - Standard production
    SET "line2_parts" WITH (GET TOPIC "factory/line2/parts" + 5)
    PUBLISH TOPIC "factory/line2/parts" WITH {line2_parts}
    PUBLISH TOPIC "factory/line2/rate" WITH 10.0
    
    // Total production
    SET "total_parts" WITH ({line1_parts} + {line2_parts})
    PUBLISH TOPIC "factory/total_production" WITH {total_parts}
```

**Demonstrates:** Multiple production lines with aggregation

### Production with Quality Tracking

```lot
DEFINE ACTION ProductionWithQuality
ON EVERY 30 SECONDS DO
    SET "parts_produced" WITH (GET TOPIC "factory/line1/parts_produced" + 5)
    SET "parts_passed" WITH (GET TOPIC "factory/line1/parts_passed" + 4)
    SET "parts_failed" WITH (GET TOPIC "factory/line1/parts_failed" + 1)
    
    PUBLISH TOPIC "factory/line1/parts_produced" WITH {parts_produced}
    PUBLISH TOPIC "factory/line1/parts_passed" WITH {parts_passed}
    PUBLISH TOPIC "factory/line1/parts_failed" WITH {parts_failed}
    
    SET "quality_rate" WITH ({parts_passed} AS DOUBLE / {parts_produced} AS DOUBLE * 100)
    PUBLISH TOPIC "factory/line1/quality_rate" WITH {quality_rate}
    
    IF {quality_rate} < 90 THEN
        PUBLISH TOPIC "factory/line1/alerts" WITH "Quality rate below 90%: " + {quality_rate} + "%"
```

**Demonstrates:** Quality metrics with conditional alerts

### Production with Shift Tracking

```lot
DEFINE ACTION ProductionWithShifts
ON EVERY 30 SECONDS DO
    SET "hour" WITH (TIMESTAMP "HOUR")
    SET "shift" WITH IF {hour} >= 6 AND {hour} < 14 THEN "Morning" 
                    ELSE IF {hour} >= 14 AND {hour} < 22 THEN "Afternoon" 
                    ELSE "Night"
    
    SET "parts_produced" WITH (GET TOPIC "factory/line1/parts_produced" + 5)
    SET "shift_parts" WITH (GET TOPIC "factory/line1/shift/" + {shift} + "/parts" + 5)
    
    PUBLISH TOPIC "factory/line1/parts_produced" WITH {parts_produced}
    PUBLISH TOPIC "factory/line1/current_shift" WITH {shift}
    PUBLISH TOPIC "factory/line1/shift/" + {shift} + "/parts" WITH {shift_parts}
```

**Demonstrates:** Time-based shift tracking and per-shift metrics

### Production with Efficiency Monitoring

```lot
DEFINE ACTION ProductionEfficiency
ON EVERY 30 SECONDS DO
    SET "parts_produced" WITH (GET TOPIC "factory/line1/parts_produced" + 5)
    SET "runtime_minutes" WITH (GET TOPIC "factory/line1/runtime_minutes" + 0.5)
    SET "target_parts" WITH (GET TOPIC "factory/line1/target_parts" + 6)
    
    PUBLISH TOPIC "factory/line1/parts_produced" WITH {parts_produced}
    PUBLISH TOPIC "factory/line1/runtime_minutes" WITH {runtime_minutes}
    PUBLISH TOPIC "factory/line1/target_parts" WITH {target_parts}
    
    SET "efficiency" WITH ({parts_produced} AS DOUBLE / {target_parts} AS DOUBLE * 100)
    PUBLISH TOPIC "factory/line1/efficiency" WITH {efficiency}
    
    SET "status" WITH IF {efficiency} >= 100 THEN "Excellent" 
                     ELSE IF {efficiency} >= 90 THEN "Good" 
                     ELSE IF {efficiency} >= 75 THEN "Fair" 
                     ELSE "Poor"
    
    PUBLISH TOPIC "factory/line1/efficiency_status" WITH {status}
```

**Demonstrates:** Efficiency calculation with status classification

## Integration Patterns

### Dashboard Integration

This simulator provides perfect data for dashboards:
- Real-time production counts
- Live rate calculations
- Status updates
- Trend data over time

### Database Storage

Combine with OPENSEARCH route to store production history:
```lot
DEFINE ROUTE ProductionStorage WITH TYPE OPENSEARCH
    ADD OPENSEARCH_CONFIG
        WITH BASE_URL "http://opensearch:9200"
        WITH INDEX "production-data"
    ADD MAPPING production
        WITH TOPIC "factory/+/parts_produced"
```

## Related Patterns

- [Simple Heartbeat](./Simple-Heartbeat.md) - Basic periodic operations
- [Counter Action](./Counter-Action.md) - State persistence fundamentals

[Back to Actions](../DEFINE%20ACTION.md)

