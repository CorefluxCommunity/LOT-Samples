---
title: PUBLISH MODEL Pattern
category: Model
version: 1.0.0
last_updated: 2025-10-25
description: Dynamic data creation from actions - event-driven structured data publishing with JSON extraction
related_tutorials: Tutorial/03-models/action-models.lotnb
---

# PUBLISH MODEL Pattern - Action Models

## Overview

**Action Models combine the power of event-driven actions with structured data models.** This pattern enables:
- Dynamic data creation based on events
- Structured responses to real-time conditions
- JSON data extraction and processing
- Consistent data formats across different triggers
- Event-driven database-like records

Unlike basic models that trigger automatically on topic changes, action models are explicitly published from within actions, giving you full control over when and how structured data is created.

## Pattern Syntax

### Step 1: Define COLLAPSED Model
```lot
DEFINE MODEL <ModelName> COLLAPSED
    ADD <FIELD_TYPE> "<field_name>"
    ADD <FIELD_TYPE> "<field_name>"
    ...
```

**Key difference:** `COLLAPSED` means no `WITH TOPIC` - the model is a template, not an automatic trigger

### Step 2: Publish Model from Action
```lot
DEFINE ACTION <ActionName>
ON TOPIC "<trigger/topic>" DO
    // Extract and process data
    SET "variable" WITH (processing logic)
    
    // Publish the model with runtime data
    PUBLISH MODEL <ModelName> TO "<output/topic>" WITH
        field_name = value_expression
        another_field = another_expression
        ...
```

## JSON Data Extraction

Action models commonly use `GET JSON` to extract fields from JSON payloads:

### GET JSON Syntax
```lot
GET JSON "<field_name>" IN PAYLOAD AS <TYPE>
```

### Supported Type Casting
- `AS STRING` - Extract as text
- `AS DOUBLE` - Extract as decimal number
- `AS INT` - Extract as integer
- `AS BOOL` - Extract as boolean

### Nested Field Access
```lot
GET JSON "parent.child" IN PAYLOAD AS STRING
GET JSON "data.sensors.temperature" IN PAYLOAD AS DOUBLE
```

## Pattern Examples

### Example 1: JSON Sensor Data Processing

```lot
DEFINE MODEL SensorDataRecord COLLAPSED
    ADD STRING "sensor_id"
    ADD STRING "sensor_type"
    ADD DOUBLE "temperature"
    ADD DOUBLE "pressure"
    ADD STRING "timestamp"
    ADD STRING "quality_status"
    ADD STRING "location"
```

```lot
DEFINE ACTION ProcessSensorJSON
ON TOPIC "sensors/+/json_data" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    
    // Extract data from JSON payload using GET JSON
    SET "temperature_val" WITH (GET JSON "temperature" IN PAYLOAD AS DOUBLE)
    SET "pressure_val" WITH (GET JSON "pressure" IN PAYLOAD AS DOUBLE)
    SET "sensor_type" WITH (GET JSON "type" IN PAYLOAD AS STRING)
    SET "quality" WITH (GET JSON "quality_status" IN PAYLOAD AS STRING)
    SET "location" WITH (GET JSON "location" IN PAYLOAD AS STRING)
    
    // Publish structured sensor record
    PUBLISH MODEL SensorDataRecord TO "sensors/processed/" + {sensor_id} WITH
        sensor_id = {sensor_id}
        sensor_type = {sensor_type}
        temperature = {temperature_val}
        pressure = {pressure_val}
        timestamp = TIMESTAMP "UTC"
        quality_status = {quality}
        location = {location}
```

**Test with:**
```
Publish to: sensors/temp001/json_data
Payload: 
{
  "temperature": 75.5,
  "pressure": 101.3,
  "type": "environmental",
  "quality_status": "GOOD",
  "location": "Factory Floor A"
}

Result in: sensors/processed/temp001
{
  "sensor_id": "temp001",
  "sensor_type": "environmental",
  "temperature": 75.5,
  "pressure": 101.3,
  "timestamp": "2025-10-25T14:30:15Z",
  "quality_status": "GOOD",
  "location": "Factory Floor A"
}
```

**Key Features:**
- Extracts JSON fields with type casting
- Combines extracted data with action variables
- Adds runtime timestamp
- Creates structured output from raw JSON

### Example 2: Dynamic Alarm Generation

```lot
DEFINE MODEL AlarmRecord COLLAPSED
    ADD STRING "alarm_id"
    ADD STRING "equipment_id"
    ADD STRING "alarm_type"
    ADD DOUBLE "trigger_value"
    ADD DOUBLE "threshold"
    ADD STRING "severity"
    ADD STRING "message"
    ADD STRING "timestamp"
    ADD BOOL "auto_generated"
```

```lot
DEFINE ACTION JSONAlarmProcessor
ON TOPIC "alarms/+/json_input" DO
    SET "equipment_id" WITH TOPIC POSITION 1
    
    // Extract alarm data from JSON payload
    SET "alarm_type" WITH (GET JSON "alarm_type" IN PAYLOAD AS STRING)
    SET "current_value" WITH (GET JSON "current_value" IN PAYLOAD AS DOUBLE)
    SET "threshold_value" WITH (GET JSON "threshold" IN PAYLOAD AS DOUBLE)
    SET "severity_level" WITH (GET JSON "severity" IN PAYLOAD AS STRING)
    SET "alarm_message" WITH (GET JSON "message" IN PAYLOAD AS STRING)
    
    // Generate structured alarm record
    PUBLISH MODEL AlarmRecord TO "alarms/structured/" + {equipment_id} WITH
        alarm_id = (RANDOM UUID)
        equipment_id = {equipment_id}
        alarm_type = {alarm_type}
        trigger_value = {current_value}
        threshold = {threshold_value}
        severity = {severity_level}
        message = {alarm_message}
        timestamp = TIMESTAMP "UTC"
        auto_generated = TRUE
    
    // Route based on severity
    IF {severity_level} EQUALS "CRITICAL" THEN
        PUBLISH TOPIC "alarms/critical/" + {equipment_id} WITH {alarm_message}
```

**Demonstrates:**
- Multiple JSON field extraction
- Unique ID generation with `RANDOM UUID`
- Conditional routing based on extracted data
- Comprehensive alarm record creation

### Example 3: Production Event Tracking

```lot
DEFINE MODEL ProductionEvent COLLAPSED
    ADD STRING "event_id"
    ADD STRING "line_id"
    ADD STRING "event_type"
    ADD INT "parts_count"
    ADD STRING "operator"
    ADD STRING "shift"
    ADD DOUBLE "cycle_time"
    ADD STRING "timestamp"
    ADD OBJECT "performance_metrics"
```

```lot
DEFINE ACTION ProductionEventProcessor
ON TOPIC "production/+/events/json" DO
    SET "line_id" WITH TOPIC POSITION 1
    
    // Extract production event data from JSON
    SET "event_type" WITH (GET JSON "event_type" IN PAYLOAD AS STRING)
    SET "parts_produced" WITH (GET JSON "parts_produced" IN PAYLOAD AS INT)
    SET "operator_name" WITH (GET JSON "operator" IN PAYLOAD AS STRING)
    SET "shift_name" WITH (GET JSON "shift" IN PAYLOAD AS STRING)
    SET "cycle_duration" WITH (GET JSON "cycle_time" IN PAYLOAD AS DOUBLE)
    SET "target_time" WITH (GET JSON "target_cycle_time" IN PAYLOAD AS DOUBLE)
    
    // Calculate performance metrics
    SET "efficiency" WITH ({target_time} / {cycle_duration} * 100)
    SET "variance" WITH ({cycle_duration} - {target_time})
    
    // Publish structured production event
    PUBLISH MODEL ProductionEvent TO "production/events/structured/" + {line_id} WITH
        event_id = (RANDOM UUID)
        line_id = {line_id}
        event_type = {event_type}
        parts_count = {parts_produced}
        operator = {operator_name}
        shift = {shift_name}
        cycle_time = {cycle_duration}
        timestamp = TIMESTAMP "UTC"
        performance_metrics = {
            "efficiency_percent": {efficiency},
            "target_cycle_time": {target_time},
            "variance_seconds": {variance},
            "performance_rating": IF {efficiency} > 100 THEN "EXCELLENT" ELSE IF {efficiency} > 90 THEN "GOOD" ELSE "NEEDS_IMPROVEMENT"
        }
```

**Demonstrates:**
- Extraction of multiple data types (STRING, INT, DOUBLE)
- Calculated metrics from extracted data
- Nested OBJECT with complex performance data
- Conditional logic within nested structures

### Example 4: Quality Report with Calculations

```lot
DEFINE MODEL QualityReport COLLAPSED
    ADD STRING "report_id"
    ADD STRING "station_id"
    ADD STRING "inspector"
    ADD INT "items_tested"
    ADD INT "items_passed"
    ADD INT "items_failed"
    ADD DOUBLE "pass_rate"
    ADD STRING "test_type"
    ADD STRING "report_timestamp"
```

```lot
DEFINE ACTION QualityDataProcessor
ON TOPIC "quality/+/test_results" DO
    SET "station_id" WITH TOPIC POSITION 1
    
    // Extract from JSON
    SET "inspector_name" WITH (GET JSON "inspector" IN PAYLOAD AS STRING)
    SET "total_tested" WITH (GET JSON "total_tested" IN PAYLOAD AS INT)
    SET "passed_count" WITH (GET JSON "passed" IN PAYLOAD AS INT)
    SET "failed_count" WITH (GET JSON "failed" IN PAYLOAD AS INT)
    SET "test_type_name" WITH (GET JSON "test_type" IN PAYLOAD AS STRING)
    
    // Calculate pass rate
    SET "calculated_pass_rate" WITH ({passed_count} AS DOUBLE / {total_tested} AS DOUBLE * 100)
    
    // Generate quality report
    PUBLISH MODEL QualityReport TO "quality/reports/" + {station_id} WITH
        report_id = (RANDOM UUID)
        station_id = {station_id}
        inspector = {inspector_name}
        items_tested = {total_tested}
        items_passed = {passed_count}
        items_failed = {failed_count}
        pass_rate = {calculated_pass_rate}
        test_type = {test_type_name}
        report_timestamp = TIMESTAMP "UTC"
    
    // Alert on low pass rate
    IF {calculated_pass_rate} < 90 THEN
        PUBLISH TOPIC "alerts/quality/low_pass_rate/" + {station_id} WITH "Pass rate: " + {calculated_pass_rate} + "%"
```

**Demonstrates:**
- Type casting in calculations (`AS DOUBLE`)
- Mathematical operations on extracted data
- Alert generation based on calculated values

## Field Population Methods

### 1. Direct Values
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    status = "ACTIVE"
    count = 100
    enabled = TRUE
```

### 2. Action Variables
```lot
SET "sensor_id" WITH TOPIC POSITION 1
PUBLISH MODEL MyModel TO "output/topic" WITH
    sensor_id = {sensor_id}
```

### 3. JSON Extraction
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    temperature = (GET JSON "temperature" IN PAYLOAD AS DOUBLE)
```

### 4. Topic Data
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    status = GET TOPIC "system/current/status"
```

### 5. Calculations
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    efficiency = ({produced} / {target} * 100)
```

### 6. Timestamps
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    timestamp = TIMESTAMP "UTC"
    unix_time = TIMESTAMP "UNIX"
```

### 7. Conditional Values
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    priority = IF {severity} EQUALS "HIGH" THEN 1 ELSE 2
```

### 8. Nested Objects
```lot
PUBLISH MODEL MyModel TO "output/topic" WITH
    metadata = {
        "source": {sensor_id},
        "quality": {quality_score},
        "validated": TRUE
    }
```

## Use Cases

### Event Logging
Create structured log entries when specific events occur:
- Equipment state changes
- Production milestones
- Alarm occurrences
- User actions

### Data Transformation
Convert raw data into standardized formats:
- JSON to structured records
- Multiple sources to single record
- Unit conversions and calculations
- Data enrichment

### Report Generation
Build reports when conditions are met:
- Shift summaries
- Quality reports
- Performance analyses
- Compliance records

### Alarm Management
Generate standardized alarms from various triggers:
- Threshold violations
- State changes
- Error conditions
- Maintenance alerts

## Best Practices

### 1. Always Use Type Casting with GET JSON
```lot
// Good - explicit type casting
SET "temp" WITH (GET JSON "temperature" IN PAYLOAD AS DOUBLE)

// Bad - no type casting
SET "temp" WITH (GET JSON "temperature" IN PAYLOAD)
```

### 2. Handle Missing JSON Fields
```lot
// Check if field exists or provide defaults
SET "optional_field" WITH (GET JSON "optional" IN PAYLOAD AS STRING)
IF {optional_field} EQUALS "" THEN
    SET "optional_field" WITH "DEFAULT_VALUE"
```

### 3. Validate Extracted Data
```lot
SET "value" WITH (GET JSON "value" IN PAYLOAD AS DOUBLE)
IF {value} < 0 OR {value} > 100 THEN
    PUBLISH TOPIC "errors/validation" WITH "Invalid value: " + {value}
ELSE
    PUBLISH MODEL ValidatedData TO "processed/data" WITH value = {value}
```

### 4. Use COLLAPSED for Action Models
```lot
// Correct - for action publishing
DEFINE MODEL MyModel COLLAPSED
    ADD STRING "field"

// Incorrect - WITH TOPIC not needed
DEFINE MODEL MyModel WITH TOPIC "some/topic" COLLAPSED
    ADD STRING "field"
```

## Python Integration

Combine action models with Python for advanced processing:

```python
# Script Name: DataProcessor
def validate_and_enrich(json_data):
    import json
    data = json.loads(json_data)
    
    # Validation logic
    if data.get('temperature', 0) < 0:
        return {"error": "Invalid temperature"}
    
    # Enrichment
    data['validated'] = True
    data['quality_score'] = calculate_quality(data)
    
    return json.dumps(data)
```

```lot
DEFINE ACTION PythonModelIntegration
ON TOPIC "sensors/+/raw" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    
    CALL PYTHON "DataProcessor.validate_and_enrich"
        WITH (PAYLOAD)
        RETURN AS {processed}
    
    PUBLISH MODEL SensorDataRecord TO "sensors/processed/" + {sensor_id} WITH
        sensor_id = {sensor_id}
        temperature = (GET JSON "temperature" IN {processed} AS DOUBLE)
        quality_score = (GET JSON "quality_score" IN {processed} AS DOUBLE)
        validated = (GET JSON "validated" IN {processed} AS BOOL)
        timestamp = TIMESTAMP "UTC"
```

## Related Patterns

- [DEFINE MODEL Pattern](../Basic-Models/DEFINE-MODEL-Pattern.md) - Basic model structure
- [FROM Keyword Pattern](../Model-Inheritance/FROM-Keyword-Pattern.md) - Model inheritance

[Back to Models](../DEFINE%20MODEL.md)

