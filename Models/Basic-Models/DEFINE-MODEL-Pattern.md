---
title: DEFINE MODEL Pattern
category: Model
version: 1.0.0
last_updated: 2025-10-25
description: Core pattern for defining data structures that automatically handle JSON payloads and trigger on topic changes
related_tutorials: Tutorial/03-models/basic-models.lotnb
---

# DEFINE MODEL Pattern - Basic Models

## Overview

**Basic Models are like database schemas for MQTT data.** They define the structure of your data, ensure consistency across your system, provide automatic JSON formatting, enable data validation and defaults, and create reusable data templates.

The basic model pattern establishes automatic data pipelines that:
- Trigger on topic data changes
- Combine data from multiple sources
- Perform calculations and transformations
- Output structured JSON to MQTT topics
- Automatically handle JSON payload reception

## Pattern Syntax

```lot
DEFINE MODEL <ModelName> WITH TOPIC "<output/topic/pattern>"
    ADD <FIELD_TYPE> "<field_name>" WITH <data_source> [AS TRIGGER]
    ADD <FIELD_TYPE> "<field_name>" WITH <data_source>
    ...
```

## Field Types

- **`STRING`** - Text data
- **`INT`** - Integer numbers
- **`DOUBLE`** - Decimal numbers (floating-point)
- **`BOOL`** - Boolean values (true/false)
- **`OBJECT`** - JSON objects (nested structures)
- **`ARRAY`** - JSON arrays

## Data Sources

### 1. Static Values
Fixed values that don't change:

```lot
ADD STRING "unit" WITH "celsius"
ADD INT "default_threshold" WITH 100
ADD STRING "location" WITH "Factory Floor A"
ADD BOOL "enabled" WITH TRUE
```

### 2. Topic Data
Values from other MQTT topics:

```lot
ADD DOUBLE "temperature" WITH TOPIC "sensors/raw/temperature"
ADD STRING "status" WITH TOPIC "equipment/current/status"
ADD INT "count" WITH TOPIC "production/current/count"
```

### 3. Trigger Fields
Fields that cause the model to publish when they change:

```lot
ADD DOUBLE "value" WITH TOPIC "sensors/raw/temperature" AS TRIGGER
```

**Only publish when this field updates**

### 4. Timestamps
Built-in timestamp generation:

```lot
ADD STRING "timestamp" WITH TIMESTAMP "UTC"         // "2025-10-25T14:30:15Z"
ADD STRING "iso_time" WITH TIMESTAMP "ISO"          // ISO 8601 format
ADD INT "unix_time" WITH TIMESTAMP "UNIX"           // 1729864215
ADD INT "unix_ms" WITH TIMESTAMP "UNIX-MS"          // Milliseconds
```

### 5. Calculations
Computed values from other topics:

```lot
ADD DOUBLE "efficiency" WITH (GET TOPIC "produced" AS DOUBLE / GET TOPIC "target" AS DOUBLE * 100)
ADD INT "total" WITH (GET TOPIC "passed" + GET TOPIC "failed")
ADD DOUBLE "fahrenheit" WITH (GET TOPIC "celsius" AS DOUBLE * 9 / 5 + 32)
```

### 6. Conditional Values
Dynamic values based on conditions:

```lot
ADD STRING "status" WITH IF (GET TOPIC "value" > 80) THEN "HIGH" ELSE "NORMAL"
ADD STRING "quality" WITH IF (GET TOPIC "defects" = 0) THEN "PASS" ELSE "FAIL"
ADD INT "priority" WITH IF (GET TOPIC "severity" EQUALS "CRITICAL") THEN 1 ELSE IF (GET TOPIC "severity" EQUALS "HIGH") THEN 2 ELSE 3
```

### 7. Nested Objects
Complex JSON structures:

```lot
ADD OBJECT "metadata" WITH {
    "location": TOPIC "equipment/location",
    "installation_date": "2025-01-01",
    "last_calibration": TOPIC "equipment/last_cal"
}
```

## Normal Model Behavior: JSON Payload Reception

**All basic models automatically handle JSON payloads.** When a JSON payload arrives at the model's topic, LOT explodes it into individual sub-topics for each field.

### Singleton Reception (Specific Topic)

Model with exact topic path:

```lot
DEFINE MODEL MaterialContentData WITH TOPIC "Process/Machine/1/Material/Content"
```

**JSON arrives at:** `Process/Machine/1/Material/Content`
```json
{
   "MaterialID": 132323,
   "Name": "Peppers",
   "Quantity": 21,
   "ExpiryDate": "2025-12-31"
}
```

**Explodes into:**
- `Process/Machine/1/Material/Content/MaterialID` → `132323`
- `Process/Machine/1/Material/Content/Name` → `Peppers`
- `Process/Machine/1/Material/Content/Quantity` → `21`
- `Process/Machine/1/Material/Content/ExpiryDate` → `"2025-12-31"`

### Multi-Instance Reception (Wildcard Topics)

Model with wildcard patterns:

```lot
DEFINE MODEL MaterialContentData WITH TOPIC "Process/+/+/Material/Content"
```

**Matches any machine and line:**
- `Process/Machine/1/Material/Content`
- `Process/Machine/33/Material/Content`
- `Process/Mixer/5/Material/Content`

**JSON arrives at:** `Process/Machine/33/Material/Content`
```json
{
   "MaterialID": 132323,
   "Name": "Peppers",
   "Quantity": 21
}
```

**Explodes into:**
- `Process/Machine/33/Material/Content/MaterialID` → `132323`
- `Process/Machine/33/Material/Content/Name` → `Peppers`
- `Process/Machine/33/Material/Content/Quantity` → `21`

**Each unique topic path creates a separate model instance.**

## Pattern Examples

### Example 1: Simple Sensor Model

```lot
DEFINE MODEL SensorReading WITH TOPIC "sensors/formatted/temperature"
    ADD STRING "sensor_id" WITH "TEMP001"
    ADD DOUBLE "value" WITH TOPIC "sensors/raw/temperature" AS TRIGGER
    ADD STRING "unit" WITH "celsius"
    ADD STRING "timestamp" WITH TIMESTAMP "UTC"
    ADD STRING "status" WITH "ACTIVE"
```

**How it works:**
- Triggers when `sensors/raw/temperature` receives data
- Combines sensor ID, value, unit, timestamp, and status
- Publishes structured JSON to `sensors/formatted/temperature`

**Output:**
```json
{
  "sensor_id": "TEMP001",
  "value": 25.5,
  "unit": "celsius",
  "timestamp": "2025-10-25T14:30:15Z",
  "status": "ACTIVE"
}
```

### Example 2: Multi-Source Equipment Status

```lot
DEFINE MODEL EquipmentStatus WITH TOPIC "equipment/status/formatted"
    ADD STRING "equipment_id" WITH TOPIC "equipment/current/id"
    ADD STRING "status" WITH TOPIC "equipment/current/status" AS TRIGGER
    ADD INT "runtime_hours" WITH TOPIC "equipment/current/runtime"
    ADD DOUBLE "efficiency" WITH TOPIC "equipment/current/efficiency"
    ADD BOOL "maintenance_required" WITH TOPIC "equipment/current/maintenance_flag"
    ADD STRING "last_update" WITH TIMESTAMP "UTC"
    ADD STRING "location" WITH "Factory Floor A"
```

**How it works:**
- Combines data from 5 different topics
- Triggers only when `equipment/current/status` changes
- Mixes dynamic (from topics) and static (location) data
- Adds timestamp automatically

**Output:**
```json
{
  "equipment_id": "PUMP001",
  "status": "RUNNING",
  "runtime_hours": 1250,
  "efficiency": 87.5,
  "maintenance_required": false,
  "last_update": "2025-10-25T14:30:15Z",
  "location": "Factory Floor A"
}
```

### Example 3: Production Record with Calculations

```lot
DEFINE MODEL ProductionRecord WITH TOPIC "production/records/completed"
    ADD STRING "batch_id" WITH TOPIC "production/current/batch_id"
    ADD STRING "product_code" WITH TOPIC "production/current/product_code"
    ADD INT "quantity_produced" WITH TOPIC "production/current/quantity" AS TRIGGER
    ADD INT "quantity_target" WITH TOPIC "production/current/target"
    ADD DOUBLE "efficiency_percent" WITH (GET TOPIC "production/current/quantity" AS DOUBLE / GET TOPIC "production/current/target" AS DOUBLE * 100)
    ADD STRING "operator" WITH TOPIC "production/current/operator"
    ADD STRING "shift" WITH TOPIC "production/current/shift"
    ADD STRING "start_time" WITH TOPIC "production/current/start_time"
    ADD STRING "completion_time" WITH TIMESTAMP "UTC"
    ADD BOOL "quality_passed" WITH TOPIC "production/current/quality_ok"
```

**How it works:**
- Triggers when production quantity is updated
- Calculates efficiency percentage automatically
- Combines operational data from multiple sources
- Type casting with `AS DOUBLE` for accurate calculations

**Output:**
```json
{
  "batch_id": "BATCH_2025_001",
  "product_code": "WIDGET_A",
  "quantity_produced": 95,
  "quantity_target": 100,
  "efficiency_percent": 95.0,
  "operator": "John Smith",
  "shift": "Day Shift",
  "start_time": "2025-10-25T08:00:00Z",
  "completion_time": "2025-10-25T14:30:15Z",
  "quality_passed": true
}
```

### Example 4: Alarm with Conditional Logic

```lot
DEFINE MODEL AlarmMessage WITH TOPIC "alarms/formatted/+"
    ADD STRING "alarm_id" WITH (RANDOM UUID)
    ADD STRING "source_equipment" WITH TOPIC "alarms/current/equipment_id"
    ADD STRING "alarm_type" WITH TOPIC "alarms/current/type"
    ADD STRING "severity" WITH TOPIC "alarms/current/severity" AS TRIGGER
    ADD STRING "message" WITH TOPIC "alarms/current/message"
    ADD STRING "timestamp" WITH TIMESTAMP "UTC"
    ADD STRING "acknowledged" WITH "false"
    ADD INT "priority" WITH IF (GET TOPIC "alarms/current/severity" EQUALS "CRITICAL") THEN 1 
                            ELSE IF (GET TOPIC "alarms/current/severity" EQUALS "HIGH") THEN 2 
                            ELSE 3
```

**How it works:**
- Generates unique alarm IDs with `RANDOM UUID`
- Wildcard topic pattern (`+`) for multiple alarm sources
- Conditional priority assignment based on severity
- Includes alarm lifecycle fields (acknowledged, priority)

### Example 5: Complex Sensor with Nested Objects

```lot
DEFINE MODEL ComprehensiveSensorData WITH TOPIC "sensors/comprehensive/+"
    ADD STRING "sensor_id" WITH TOPIC "sensors/current/id"
    ADD STRING "sensor_type" WITH TOPIC "sensors/current/type"
    ADD DOUBLE "primary_value" WITH TOPIC "sensors/current/value" AS TRIGGER
    ADD STRING "unit" WITH TOPIC "sensors/current/unit"
    ADD BOOL "calibrated" WITH TOPIC "sensors/current/calibrated"
    ADD INT "reading_count" WITH (GET TOPIC "sensors/current/reading_count" + 1)
    ADD STRING "quality_status" WITH IF (GET TOPIC "sensors/current/value" > 0 AND GET TOPIC "sensors/current/calibrated" EQUALS TRUE) THEN "GOOD" ELSE "POOR"
    ADD OBJECT "metadata" WITH {
        "location": TOPIC "sensors/current/location",
        "installation_date": TOPIC "sensors/current/install_date",
        "last_calibration": TOPIC "sensors/current/last_cal"
    }
    ADD STRING "timestamp" WITH TIMESTAMP "UTC"
```

**How it works:**
- Automatic reading counter (increments on each trigger)
- Conditional quality status based on multiple factors
- Nested object for metadata
- Combines multiple data types and sources

## Use Cases

### IoT Data Standardization
- Transform raw sensor data into consistent formats
- Add metadata and timestamps
- Combine related measurements

### Equipment Monitoring
- Aggregate equipment status from multiple sources
- Calculate performance metrics
- Track operational parameters

### Production Tracking
- Create comprehensive production records
- Calculate efficiency and yield
- Track batch and shift information

### Alarm Management
- Standardize alarm formats
- Add context and priority
- Include lifecycle tracking fields

## Best Practices

### 1. Consistent Field Naming
```lot
// Good - consistent across models
ADD STRING "timestamp"
ADD STRING "sensor_id"
ADD STRING "status"

// Avoid - inconsistent naming
ADD STRING "time_stamp"
ADD STRING "sensorID"
ADD STRING "current_status"
```

### 2. Include Essential Context
Always include:
- Identification (IDs, names)
- Timestamps
- Units of measurement
- Status or quality indicators

### 3. Use Appropriate Field Types
```lot
// Correct type usage
ADD DOUBLE "temperature"      // Decimal values
ADD INT "count"               // Whole numbers
ADD BOOL "is_active"          // True/false
ADD STRING "status"           // Text
```

### 4. Trigger on the Right Field
```lot
// Trigger on the field that represents the primary event
ADD DOUBLE "value" WITH TOPIC "sensors/temperature" AS TRIGGER

// Not on metadata that changes frequently
// ADD STRING "timestamp" WITH TIMESTAMP "UTC" AS TRIGGER  // Bad!
```

### 5. Type Casting for Calculations
```lot
// Always cast to appropriate types for math
ADD DOUBLE "efficiency" WITH (GET TOPIC "produced" AS DOUBLE / GET TOPIC "target" AS DOUBLE * 100)
```

## Related Patterns

- [PUBLISH MODEL Pattern](../Action-Models/PUBLISH-MODEL-Pattern.md) - Dynamic model publishing from actions
- [FROM Keyword Pattern](../Model-Inheritance/FROM-Keyword-Pattern.md) - Model inheritance

[Back to Models](../DEFINE%20MODEL.md)

