---
title: DEFINE MODEL
category: Model
version: 1.0.0
last_updated: 2025-10-25
description: Define data structures that transform, aggregate, and format MQTT data with automatic JSON handling
related_tutorials: Tutorial/03-models/basic-models.lotnb, Tutorial/03-models/action-models.lotnb, Tutorial/03-models/model-inheritance.lotnb
---

# KEYWORD: `DEFINE MODEL`

## 1. Overview

**Models in LOT are like database schemas for MQTT data.** They:
- Define the structure of your data
- Ensure consistency across your system
- Provide automatic JSON formatting
- Enable data validation and defaults
- Create reusable data templates

Models serve as recipes that can generate model instances, transforming and aggregating values from input MQTT topics and publishing results to new topics. They create structured, consistent data formats for industrial IoT applications.

## 2. Model Types

LOT supports three primary model patterns:

### 2.1 Basic Models
**Models as data schemas** - Define structure, trigger on topic data, automatically handle JSON

```lot
DEFINE MODEL SensorReading WITH TOPIC "sensors/formatted/temperature"
    ADD STRING "sensor_id" WITH "TEMP001"
    ADD DOUBLE "value" WITH TOPIC "sensors/raw/temperature" AS TRIGGER
    ADD STRING "unit" WITH "celsius"
    ADD STRING "timestamp" WITH TIMESTAMP "UTC"
```

[Learn more about Basic Models →](./Basic-Models/)

### 2.2 Action Models
**Dynamic data creation based on events** - Publish models from actions with runtime data

```lot
DEFINE MODEL SensorDataRecord COLLAPSED
    ADD STRING "sensor_id"
    ADD DOUBLE "temperature"
    ADD STRING "timestamp"

DEFINE ACTION ProcessSensorData
ON TOPIC "sensors/+/data" DO
    PUBLISH MODEL SensorDataRecord TO "sensors/processed/" + {sensor_id} WITH
        sensor_id = {sensor_id}
        temperature = (GET JSON "temperature" IN PAYLOAD AS DOUBLE)
        timestamp = TIMESTAMP "UTC"
```

[Learn more about Action Models →](./Action-Models/)

### 2.3 Model Inheritance
**Create model families with shared characteristics** - Reuse common fields

```lot
DEFINE MODEL BaseAlert COLLAPSED
    ADD STRING "alert_id"
    ADD STRING "timestamp"
    ADD STRING "severity"

DEFINE MODEL TemperatureAlert FROM BaseAlert
    ADD DOUBLE "temperature_value"
    ADD DOUBLE "threshold_exceeded"
```

[Learn more about Model Inheritance →](./Model-Inheritance/)

## 3. Signature

### Basic Model Syntax
```lot
DEFINE MODEL <model_name> WITH TOPIC "<output_topic>"
    ADD <FIELD_TYPE> "<field_name>" WITH <source> [AS TRIGGER]
    ...
```

### Collapsed Model Syntax (for Action Models)
```lot
DEFINE MODEL <model_name> COLLAPSED
    ADD <FIELD_TYPE> "<field_name>"
    ...
```

### Inheritance Syntax
```lot
DEFINE MODEL <child_model> FROM <parent_model>
    ADD <FIELD_TYPE> "<field_name>"
    ...
```

## 4. Field Types

- **`STRING`** - Text data
- **`INT`** - Integer numbers
- **`DOUBLE`** - Decimal numbers (floating-point)
- **`BOOL`** - Boolean values (true/false)
- **`OBJECT`** - JSON objects (nested structures)
- **`ARRAY`** - JSON arrays

## 5. Data Sources

### Static Values
```lot
ADD STRING "location" WITH "Factory Floor A"
ADD INT "default_threshold" WITH 100
```

### Topic Data
```lot
ADD DOUBLE "temperature" WITH TOPIC "sensors/temp/current"
ADD STRING "status" WITH TOPIC "equipment/status" AS TRIGGER
```

### Timestamps
```lot
ADD STRING "timestamp" WITH TIMESTAMP "UTC"
ADD INT "unix_time" WITH TIMESTAMP "UNIX"
```

### Calculations
```lot
ADD DOUBLE "efficiency" WITH (GET TOPIC "produced" AS DOUBLE / GET TOPIC "target" AS DOUBLE * 100)
```

### Conditional Values
```lot
ADD STRING "status" WITH IF (GET TOPIC "value" > 80) THEN "HIGH" ELSE "NORMAL"
```

## 6. Normal Model Behavior: JSON Reception

**All models automatically handle JSON payload reception.** When a JSON payload arrives at the model's topic, LOT explodes it into individual topics:

### Singleton Reception (Specific Topic)
```lot
DEFINE MODEL MaterialContentData WITH TOPIC "Process/Machine/1/Material/Content"
```

**JSON Payload arrives at:** `Process/Machine/1/Material/Content`
```json
{
   "MaterialID": 132323,
   "Name": "Peppers",
   "Quantity": 21
}
```

**Explodes into individual topics:**
- `Process/Machine/1/Material/Content/MaterialID` → `132323`
- `Process/Machine/1/Material/Content/Name` → `Peppers`
- `Process/Machine/1/Material/Content/Quantity` → `21`

### Multi-Instance Reception (Wildcard Topics)
```lot
DEFINE MODEL MaterialContentData WITH TOPIC "Process/+/+/Material/Content"
```

**JSON Payload arrives at:** `Process/Machine/33/Material/Content`
```json
{
   "MaterialID": 132323,
   "Name": "Peppers",
   "Quantity": 21
}
```

**Explodes into individual topics:**
- `Process/Machine/33/Material/Content/MaterialID` → `132323`
- `Process/Machine/33/Material/Content/Name` → `Peppers`
- `Process/Machine/33/Material/Content/Quantity` → `21`

This JSON explosion behavior:
- Creates a separate topic for each JSON field
- Supports nested objects (creates hierarchical topics)
- Enables individual field subscriptions
- Works with both singleton and wildcard topic patterns

## 7. Trigger Mechanism

The `AS TRIGGER` keyword specifies which field change causes the model to publish:

```lot
DEFINE MODEL EquipmentStatus WITH TOPIC "equipment/status/formatted"
    ADD STRING "equipment_id" WITH TOPIC "equipment/current/id"
    ADD STRING "status" WITH TOPIC "equipment/current/status" AS TRIGGER
    ADD INT "runtime_hours" WITH TOPIC "equipment/current/runtime"
```

**Behavior:**
- Model publishes only when `equipment/current/status` changes
- Other field changes don't trigger publication
- Ensures efficient, event-driven model updates

## 8. Usage Examples

### Example 1: Equipment Status Model

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

### Example 2: Production Record with Calculations

```lot
DEFINE MODEL ProductionRecord WITH TOPIC "production/records/completed"
    ADD STRING "batch_id" WITH TOPIC "production/current/batch_id"
    ADD STRING "product_code" WITH TOPIC "production/current/product_code"
    ADD INT "quantity_produced" WITH TOPIC "production/current/quantity" AS TRIGGER
    ADD INT "quantity_target" WITH TOPIC "production/current/target"
    ADD DOUBLE "efficiency_percent" WITH (GET TOPIC "production/current/quantity" AS DOUBLE / GET TOPIC "production/current/target" AS DOUBLE * 100)
    ADD STRING "operator" WITH TOPIC "production/current/operator"
    ADD STRING "completion_time" WITH TIMESTAMP "UTC"
```

**Demonstrates:**
- Multiple data sources
- Calculated fields (efficiency_percent)
- Trigger on quantity update
- Timestamp inclusion

### Example 3: Dynamic Model Publishing from Action

```lot
DEFINE MODEL AlarmRecord COLLAPSED
    ADD STRING "alarm_id"
    ADD STRING "equipment_id"
    ADD STRING "alarm_type"
    ADD DOUBLE "trigger_value"
    ADD STRING "severity"
    ADD STRING "timestamp"

DEFINE ACTION ProcessAlarm
ON TOPIC "alarms/+/json_input" DO
    SET "equipment_id" WITH TOPIC POSITION 1
    
    PUBLISH MODEL AlarmRecord TO "alarms/structured/" + {equipment_id} WITH
        alarm_id = (RANDOM UUID)
        equipment_id = {equipment_id}
        alarm_type = (GET JSON "alarm_type" IN PAYLOAD AS STRING)
        trigger_value = (GET JSON "current_value" IN PAYLOAD AS DOUBLE)
        severity = (GET JSON "severity" IN PAYLOAD AS STRING)
        timestamp = TIMESTAMP "UTC"
```

**Demonstrates:**
- COLLAPSED model definition (no WITH TOPIC)
- Dynamic publishing with PUBLISH MODEL
- JSON data extraction with GET JSON
- Runtime field population

## 9. Compatible Keywords

- **`ADD`**: Add fields to the model
- **`WITH TOPIC`**: Specify source topic for field or model output
- **`AS TRIGGER`**: Mark field that triggers model publication
- **`TIMESTAMP`**: Include timestamps in various formats
- **`IF ... THEN ... ELSE`**: Conditional field values
- **`GET TOPIC`**: Retrieve stored topic values
- **`GET JSON`**: Extract fields from JSON payloads with type casting
- **`COLLAPSED`**: Define model without output topic (for action publishing)
- **`FROM`**: Inherit fields from parent model

## 10. Python Integration

Models can be combined with Python for complex data processing:

```lot
DEFINE ACTION PythonModelProcessor
ON TOPIC "sensors/+/raw_data" DO
    SET "sensor_id" WITH TOPIC POSITION 1
    
    CALL PYTHON "DataProcessor.validate_and_transform"
        WITH (PAYLOAD)
        RETURN AS {processed_data}
    
    PUBLISH MODEL SensorDataRecord TO "sensors/processed/" + {sensor_id} WITH
        sensor_id = {sensor_id}
        temperature = (GET JSON "temperature" IN {processed_data} AS DOUBLE)
        quality = (GET JSON "quality_score" IN {processed_data} AS STRING)
        timestamp = TIMESTAMP "UTC"
```

<!-- Future Integration: Python data validation example -->
```python
# Script Name: DataProcessor
def validate_and_transform(raw_json):
    import json
    data = json.loads(raw_json)
    # Validation and transformation logic
    return processed_data
```

## 11. Model Design Principles

### Consistency
Use standard field names across similar models:
- Timestamps: Always use "timestamp" or "last_update"
- IDs: Use consistent naming like "sensor_id", "equipment_id"
- Status fields: Standardize on "status", "state", or "condition"

### Completeness
Include all necessary context:
- Identification fields (IDs, names)
- Measurement values
- Units of measurement
- Timestamps
- Status or quality indicators

### Efficiency
Only include fields that add value:
- Avoid redundant calculated fields
- Don't duplicate data unnecessarily
- Use conditional fields sparingly

### Flexibility
Use conditional logic for dynamic behavior:
- Adapt field values based on other fields
- Handle multiple scenarios in one model
- Provide defaults for missing data

## 12. Future Extensions

<!-- Future Integration: Advanced model features -->
```lot
// Planned: Model validation rules
DEFINE MODEL ValidatedSensorData WITH TOPIC "sensors/validated/+"
    ADD DOUBLE "value" WITH TOPIC "sensors/raw/+/value"
    VALIDATE value BETWEEN 0 AND 100
    ON VALIDATION_FAILURE PUBLISH TOPIC "errors/validation"
```

### Planned Features
- Built-in data validation rules
- Model versioning and migration
- Schema evolution support
- Advanced type constraints
- Model composition patterns

## 13. Notes & Additional Information

### Model Instance Identification
Each model instance is uniquely identified based on:
- The topic structure (wildcards create multiple instances)
- The time of creation
- The triggering event

This allows LOT to manage multiple concurrent model instances, even for constant properties.

### Performance Considerations
- Models trigger only on AS TRIGGER field changes
- Wildcard models create instances per unique topic match
- Calculations execute on each model publication
- Consider trigger frequency for high-volume topics

### Static vs Dynamic Models
- **Static Models**: Define WITH TOPIC, trigger on field changes
- **Dynamic Models**: Define COLLAPSED, publish from actions
- Choose based on whether you need automatic triggering or manual control

## 14. Related Keywords & Entities

- [DEFINE ACTION](../Actions/DEFINE%20ACTION.md)
- [DEFINE ROUTE](../Routes/DEFINE%20ROUTE.md)
- [SET](../Syntax/Functional/SET/SET.md)
- [GET TOPIC](../Syntax/Functional/GET%20TOPIC/GET%20TOPIC.md)
- [GET JSON](../Syntax/Functional/GET%20JSON/) *(if exists)*
- [PUBLISH TOPIC](../Syntax/Functional/PUBLISH%20TOPIC/PUBLISH%20TOPIC.md)
- [PAYLOAD](../Syntax/Entities/PAYLOAD/PAYLOAD.md)
- [TIMESTAMP](../Syntax/Entities/TIMESTAMP/TIMESTAMP.md)

[Back to Main](../README.md)
