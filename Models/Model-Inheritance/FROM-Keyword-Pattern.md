---
title: FROM Keyword Pattern
category: Model
version: 1.0.0
last_updated: 2025-10-25
description: Create model families with shared characteristics using inheritance for code reuse and consistency
related_tutorials: Tutorial/03-models/model-inheritance.lotnb
---

# FROM Keyword Pattern - Model Inheritance

## Overview

**Model Inheritance allows you to create model families that share common characteristics while adding specialized functionality.** This enables:
- Code reuse and consistency
- Easier maintenance and updates
- Logical organization of related data types
- Polymorphic data handling

Instead of duplicating common fields across multiple models, you define a base model with shared fields and extend it with specialized child models that add specific fields for different use cases.

## Pattern Syntax

### Base Model Definition
```lot
DEFINE MODEL <BaseModelName> COLLAPSED
    ADD <FIELD_TYPE> "<common_field_1>"
    ADD <FIELD_TYPE> "<common_field_2>"
    ...
```

### Child Model Definition
```lot
DEFINE MODEL <ChildModelName> FROM <BaseModelName>
    ADD <FIELD_TYPE> "<specialized_field_1>"
    ADD <FIELD_TYPE> "<specialized_field_2>"
    ...
```

**Child inherits all fields from base model** plus adds its own specialized fields.

## Inheritance Benefits

### 1. Field Inheritance
Child models automatically get all parent fields:

```lot
DEFINE MODEL BaseAlert COLLAPSED
    ADD STRING "alert_id"
    ADD STRING "timestamp"
    ADD STRING "severity"

DEFINE MODEL TemperatureAlert FROM BaseAlert
    ADD DOUBLE "temperature_value"
    ADD DOUBLE "threshold"
```

**TemperatureAlert has:**
- `alert_id` (from BaseAlert)
- `timestamp` (from BaseAlert)
- `severity` (from BaseAlert)
- `temperature_value` (specialized)
- `threshold` (specialized)

### 2. Consistency
Common fields have the same structure across all child models

### 3. Maintainability
Update base model to affect all children simultaneously

### 4. Specialization
Add specific fields only where needed

## Pattern Examples

### Example 1: Alert Model Family

```lot
DEFINE MODEL BaseAlert COLLAPSED
    ADD STRING "alert_id"
    ADD STRING "source_system"
    ADD STRING "timestamp"
    ADD STRING "severity"
    ADD STRING "message"
    ADD BOOL "acknowledged"
    ADD STRING "acknowledged_by"
```

```lot
DEFINE MODEL TemperatureAlert FROM BaseAlert
    ADD DOUBLE "temperature_value"
    ADD DOUBLE "threshold_exceeded"
    ADD STRING "sensor_location"
    ADD STRING "unit"
```

```lot
DEFINE MODEL PressureAlert FROM BaseAlert
    ADD DOUBLE "pressure_value"
    ADD DOUBLE "max_safe_pressure"
    ADD STRING "pressure_unit"
    ADD STRING "system_affected"
```

```lot
DEFINE MODEL MaintenanceAlert FROM BaseAlert
    ADD STRING "equipment_id"
    ADD STRING "maintenance_type"
    ADD INT "hours_since_last_service"
    ADD INT "recommended_service_interval"
    ADD STRING "priority_level"
```

**Usage in Action:**

```lot
DEFINE ACTION SpecializedAlertGenerator
ON TOPIC "sensors/+/+/alert" DO
    SET "sensor_type" WITH TOPIC POSITION 1
    SET "sensor_id" WITH TOPIC POSITION 2
    
    IF {sensor_type} EQUALS "temperature" THEN
        PUBLISH MODEL TemperatureAlert TO "alerts/temperature/" + {sensor_id} WITH
            alert_id = (RANDOM UUID)
            source_system = "TemperatureMonitor"
            timestamp = TIMESTAMP "UTC"
            severity = "HIGH"
            message = "Temperature threshold exceeded"
            acknowledged = FALSE
            acknowledged_by = ""
            temperature_value = GET TOPIC "sensors/temperature/" + {sensor_id} + "/current"
            threshold_exceeded = 75.0
            sensor_location = GET TOPIC "sensors/temperature/" + {sensor_id} + "/location"
            unit = "celsius"
    
    ELSE IF {sensor_type} EQUALS "pressure" THEN
        PUBLISH MODEL PressureAlert TO "alerts/pressure/" + {sensor_id} WITH
            alert_id = (RANDOM UUID)
            source_system = "PressureMonitor"
            timestamp = TIMESTAMP "UTC"
            severity = "CRITICAL"
            message = "Pressure limit exceeded"
            acknowledged = FALSE
            acknowledged_by = ""
            pressure_value = GET TOPIC "sensors/pressure/" + {sensor_id} + "/current"
            max_safe_pressure = 100.0
            pressure_unit = "PSI"
            system_affected = GET TOPIC "sensors/pressure/" + {sensor_id} + "/system"
```

**Benefits:**
- All alerts have consistent base fields (alert_id, timestamp, severity)
- Each alert type has relevant specialized fields
- Easy to process all alerts using common fields
- Easy to add new alert types by extending BaseAlert

### Example 2: Equipment Model Hierarchy

```lot
DEFINE MODEL BaseEquipment COLLAPSED
    ADD STRING "equipment_id"
    ADD STRING "equipment_name"
    ADD STRING "location"
    ADD STRING "status"
    ADD INT "runtime_hours"
    ADD STRING "last_maintenance"
    ADD STRING "operator"
    ADD STRING "timestamp"
```

```lot
DEFINE MODEL PumpEquipment FROM BaseEquipment
    ADD DOUBLE "flow_rate"
    ADD DOUBLE "pressure_output"
    ADD INT "rpm"
    ADD DOUBLE "power_consumption"
    ADD STRING "pump_type"
```

```lot
DEFINE MODEL ConveyorEquipment FROM BaseEquipment
    ADD DOUBLE "belt_speed"
    ADD INT "items_per_minute"
    ADD DOUBLE "belt_tension"
    ADD BOOL "emergency_stop_active"
    ADD STRING "direction"
```

```lot
DEFINE ACTION EquipmentStatusReporter
ON TOPIC "equipment/+/+/status_update" DO
    SET "equipment_type" WITH TOPIC POSITION 1
    SET "equipment_id" WITH TOPIC POSITION 2
    
    IF {equipment_type} EQUALS "pump" THEN
        PUBLISH MODEL PumpEquipment TO "equipment/reports/pump/" + {equipment_id} WITH
            equipment_id = {equipment_id}
            equipment_name = GET TOPIC "equipment/pump/" + {equipment_id} + "/name"
            location = GET TOPIC "equipment/pump/" + {equipment_id} + "/location"
            status = GET TOPIC "equipment/pump/" + {equipment_id} + "/status"
            runtime_hours = GET TOPIC "equipment/pump/" + {equipment_id} + "/runtime"
            last_maintenance = GET TOPIC "equipment/pump/" + {equipment_id} + "/last_service"
            operator = GET TOPIC "equipment/pump/" + {equipment_id} + "/operator"
            timestamp = TIMESTAMP "UTC"
            flow_rate = GET TOPIC "equipment/pump/" + {equipment_id} + "/flow_rate"
            pressure_output = GET TOPIC "equipment/pump/" + {equipment_id} + "/pressure"
            rpm = GET TOPIC "equipment/pump/" + {equipment_id} + "/rpm"
            power_consumption = GET TOPIC "equipment/pump/" + {equipment_id} + "/power"
            pump_type = "Centrifugal"
    
    ELSE IF {equipment_type} EQUALS "conveyor" THEN
        PUBLISH MODEL ConveyorEquipment TO "equipment/reports/conveyor/" + {equipment_id} WITH
            equipment_id = {equipment_id}
            equipment_name = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/name"
            location = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/location"
            status = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/status"
            runtime_hours = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/runtime"
            last_maintenance = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/last_service"
            operator = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/operator"
            timestamp = TIMESTAMP "UTC"
            belt_speed = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/speed"
            items_per_minute = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/throughput"
            belt_tension = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/tension"
            emergency_stop_active = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/estop"
            direction = GET TOPIC "equipment/conveyor/" + {equipment_id} + "/direction"
```

**Demonstrates:**
- Common equipment management fields in base model
- Equipment-specific operational fields in child models
- One action handling multiple equipment types
- Scalable architecture for adding new equipment types

### Example 3: Event Model Hierarchy

```lot
DEFINE MODEL BaseEvent COLLAPSED
    ADD STRING "event_id"
    ADD STRING "event_category"
    ADD STRING "source"
    ADD STRING "timestamp"
    ADD STRING "severity"
    ADD STRING "description"
    ADD OBJECT "metadata"
```

```lot
DEFINE MODEL ProductionEvent FROM BaseEvent
    ADD STRING "line_id"
    ADD STRING "batch_id"
    ADD INT "parts_affected"
    ADD STRING "operator"
    ADD DOUBLE "production_impact"
```

```lot
DEFINE MODEL MaintenanceEvent FROM BaseEvent
    ADD STRING "equipment_id"
    ADD STRING "maintenance_type"
    ADD INT "downtime_minutes"
    ADD STRING "technician"
    ADD DOUBLE "cost_estimate"
```

```lot
DEFINE MODEL QualityEvent FROM BaseEvent
    ADD STRING "inspection_station"
    ADD STRING "product_batch"
    ADD INT "items_tested"
    ADD INT "items_failed"
    ADD STRING "failure_reason"
```

**Usage:**

```lot
DEFINE ACTION ComprehensiveEventTracker
ON TOPIC "events/+/+/occurred" DO
    SET "event_category" WITH TOPIC POSITION 1
    SET "event_subtype" WITH TOPIC POSITION 2
    
    IF {event_category} EQUALS "production" THEN
        PUBLISH MODEL ProductionEvent TO "events/structured/production/" + {event_subtype} WITH
            event_id = (RANDOM UUID)
            event_category = "PRODUCTION"
            source = "ProductionSystem"
            timestamp = TIMESTAMP "UTC"
            severity = IF PAYLOAD CONTAINS "error" THEN "HIGH" ELSE "INFO"
            description = PAYLOAD
            metadata = {"system_version": "v2.1", "plant_id": "PLANT_001"}
            line_id = GET TOPIC "production/current/line_id"
            batch_id = GET TOPIC "production/current/batch_id"
            parts_affected = GET TOPIC "production/current/parts_in_process"
            operator = GET TOPIC "production/current/operator"
            production_impact = IF PAYLOAD CONTAINS "stop" THEN 100.0 ELSE 0.0
```

**Demonstrates:**
- Base event structure for all event types
- Domain-specific extensions (production, maintenance, quality)
- Consistent event processing across categories

### Example 4: Sensor Model Family

```lot
DEFINE MODEL BaseSensor COLLAPSED
    ADD STRING "sensor_id"
    ADD STRING "sensor_name"
    ADD STRING "location"
    ADD STRING "status"
    ADD STRING "last_calibration"
    ADD BOOL "calibration_valid"
    ADD STRING "timestamp"
    ADD INT "reading_count"
```

```lot
DEFINE MODEL TemperatureSensor FROM BaseSensor
    ADD DOUBLE "temperature"
    ADD STRING "temperature_unit"
    ADD DOUBLE "min_range"
    ADD DOUBLE "max_range"
    ADD DOUBLE "accuracy"
```

```lot
DEFINE MODEL PressureSensor FROM BaseSensor
    ADD DOUBLE "pressure"
    ADD STRING "pressure_unit"
    ADD DOUBLE "max_working_pressure"
    ADD STRING "connection_type"
    ADD BOOL "overpressure_protection"
```

```lot
DEFINE MODEL FlowSensor FROM BaseSensor
    ADD DOUBLE "flow_rate"
    ADD STRING "flow_unit"
    ADD DOUBLE "pipe_diameter"
    ADD STRING "fluid_type"
    ADD DOUBLE "viscosity"
```

**All sensor types share common management fields while adding measurement-specific data**

## Multi-Level Inheritance

LOT supports inheritance chains (grandparent → parent → child):

```lot
DEFINE MODEL BaseEntity COLLAPSED
    ADD STRING "entity_id"
    ADD STRING "entity_name"
    ADD STRING "timestamp"

DEFINE MODEL BaseEquipment FROM BaseEntity
    ADD STRING "location"
    ADD STRING "status"
    ADD STRING "operator"

DEFINE MODEL RotatingEquipment FROM BaseEquipment
    ADD INT "rpm"
    ADD DOUBLE "vibration"
    ADD DOUBLE "temperature"

DEFINE MODEL PumpEquipment FROM RotatingEquipment
    ADD DOUBLE "flow_rate"
    ADD DOUBLE "pressure_output"
```

**PumpEquipment inherits fields from:**
- BaseEntity (entity_id, entity_name, timestamp)
- BaseEquipment (location, status, operator)
- RotatingEquipment (rpm, vibration, temperature)
- Plus its own fields (flow_rate, pressure_output)

## Use Cases

### Alert Management Systems
- Base alert with common fields
- Specialized alerts for different event types
- Consistent alert processing infrastructure

### Equipment Monitoring
- Base equipment model with operational fields
- Equipment-type-specific models
- Unified equipment management interface

### Event Tracking
- Base event structure
- Domain-specific event types
- Enterprise event sourcing patterns

### Sensor Networks
- Base sensor model with management fields
- Sensor-type-specific measurement fields
- Scalable sensor architecture

## Best Practices

### 1. Design Base Models Carefully
```lot
// Good - common fields all children need
DEFINE MODEL BaseAlert COLLAPSED
    ADD STRING "alert_id"
    ADD STRING "timestamp"
    ADD STRING "severity"
    ADD STRING "message"

// Bad - too specific for base model
DEFINE MODEL BaseAlert COLLAPSED
    ADD STRING "alert_id"
    ADD DOUBLE "temperature"  // Not all alerts are temperature-related!
```

### 2. Use Meaningful Inheritance Hierarchies
```lot
// Good - logical hierarchy
BaseEntity → BaseEquipment → RotatingEquipment → PumpEquipment

// Bad - no logical relationship
BaseModel → RandomModel → AnotherModel
```

### 3. Keep Inheritance Depth Reasonable
- Maximum 3-4 levels deep
- Too deep = hard to understand and maintain
- Too shallow = duplicate code

### 4. Document Field Sources
When using inherited models, document which fields come from which level:

```lot
// PumpEquipment fields:
// From BaseEntity: entity_id, entity_name, timestamp
// From BaseEquipment: location, status, operator  
// From RotatingEquipment: rpm, vibration, temperature
// Specialized: flow_rate, pressure_output
```

## Field Overriding

Child models can redefine parent fields:

```lot
DEFINE MODEL BaseModel COLLAPSED
    ADD STRING "status" WITH "UNKNOWN"

DEFINE MODEL ChildModel FROM BaseModel
    ADD STRING "status" WITH "ACTIVE"  // Overrides parent's default
```

**Use sparingly** - can create confusion

## Related Patterns

- [DEFINE MODEL Pattern](../Basic-Models/DEFINE-MODEL-Pattern.md) - Basic model structure
- [PUBLISH MODEL Pattern](../Action-Models/PUBLISH-MODEL-Pattern.md) - Dynamic publishing

[Back to Models](../DEFINE%20MODEL.md)

