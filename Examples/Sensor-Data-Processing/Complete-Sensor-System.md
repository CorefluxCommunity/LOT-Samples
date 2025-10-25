---
title: Complete Sensor Data Processing System
category: Example
version: 1.0.0
last_updated: 2025-10-25
description: Comprehensive example combining Actions, Models, Routes, and Python for industrial sensor monitoring
related_tutorials: Tutorial/01-basics/, Tutorial/03-models/, Tutorial/04-routes/, Tutorial/05-python/
---

# Complete Sensor Data Processing System

## Overview

This comprehensive example demonstrates a complete industrial sensor monitoring system that combines:
- **Topic-Based Actions** for real-time data processing
- **Time-Based Actions** for periodic monitoring
- **Basic Models** for data structuring
- **Action Models** for dynamic record creation
- **Python Integration** for statistical analysis
- **MQTT Bridge Route** for cloud connectivity
- **Database Route** for data persistence

## System Architecture

```
Raw Sensor Data (MQTT)
    ↓
Topic Actions (Data Processing)
    ↓
Python Validation & Analysis
    ↓
Action Models (Structured Records)
    ↓
├── Local Storage (OpenSearch)
├── Cloud Bridge (MQTT)
└── Alarm Generation (Conditional)
    ↓
Time-Based Monitoring (Health Checks)
```

## Component 1: Python Validation Module

```python
# Script Name: SensorValidator
import json
import math

def validate_sensor_reading(sensor_data_json, sensor_id):
    """Validate and enrich sensor data"""
    try:
        data = json.loads(sensor_data_json)
        
        # Extract values
        temperature = float(data.get('temperature', 0))
        pressure = float(data.get('pressure', 0))
        humidity = float(data.get('humidity', 0))
        
        # Validation ranges
        validations = {
            "temperature_valid": -50 <= temperature <= 150,
            "pressure_valid": 0 <= pressure <= 200,
            "humidity_valid": 0 <= humidity <= 100
        }
        
        # Calculate overall quality score
        valid_count = sum(validations.values())
        quality_score = (valid_count / len(validations)) * 100
        
        # Determine quality status
        if quality_score == 100:
            quality_status = "EXCELLENT"
        elif quality_score >= 66:
            quality_status = "GOOD"
        elif quality_score >= 33:
            quality_status = "FAIR"
        else:
            quality_status = "POOR"
        
        return {
            "sensor_id": sensor_id,
            "temperature": temperature,
            "pressure": pressure,
            "humidity": humidity,
            "validations": validations,
            "quality_score": quality_score,
            "quality_status": quality_status,
            "all_valid": quality_score == 100,
            "validated_at": None  # Will be added by LOT
        }
    
    except Exception as e:
        return {
            "sensor_id": sensor_id,
            "error": str(e),
            "quality_status": "ERROR",
            "all_valid": False
        }

def calculate_sensor_statistics(readings_json):
    """Calculate statistical metrics for sensor readings"""
    try:
        readings = json.loads(readings_json)
        
        if not readings or len(readings) == 0:
            return {"error": "No readings provided"}
        
        mean = sum(readings) / len(readings)
        sorted_readings = sorted(readings)
        n = len(sorted_readings)
        median = sorted_readings[n//2] if n % 2 else (sorted_readings[n//2-1] + sorted_readings[n//2]) / 2
        
        variance = sum((x - mean) ** 2 for x in readings) / len(readings)
        std_dev = math.sqrt(variance)
        
        return {
            "mean": round(mean, 2),
            "median": round(median, 2),
            "std_dev": round(std_dev, 2),
            "min": min(readings),
            "max": max(readings),
            "range": max(readings) - min(readings),
            "count": len(readings),
            "variance": round(variance, 2)
        }
    
    except Exception as e:
        return {"error": str(e)}
```

## Component 2: Data Models

### Validated Sensor Record Model
```lot
DEFINE MODEL ValidatedSensorRecord COLLAPSED
    ADD STRING "sensor_id"
    ADD STRING "sensor_type"
    ADD STRING "location"
    ADD DOUBLE "temperature"
    ADD DOUBLE "pressure"
    ADD DOUBLE "humidity"
    ADD DOUBLE "quality_score"
    ADD STRING "quality_status"
    ADD BOOL "all_valid"
    ADD STRING "timestamp"
    ADD OBJECT "validations"
```

### Sensor Alarm Model
```lot
DEFINE MODEL SensorAlarm COLLAPSED
    ADD STRING "alarm_id"
    ADD STRING "sensor_id"
    ADD STRING "alarm_type"
    ADD STRING "severity"
    ADD STRING "message"
    ADD DOUBLE "trigger_value"
    ADD STRING "timestamp"
    ADD BOOL "acknowledged"
```

### Sensor Statistics Model
```lot
DEFINE MODEL SensorStatistics COLLAPSED
    ADD STRING "sensor_id"
    ADD STRING "statistic_period"
    ADD DOUBLE "mean"
    ADD DOUBLE "median"
    ADD DOUBLE "std_dev"
    ADD DOUBLE "min"
    ADD DOUBLE "max"
    ADD DOUBLE "range"
    ADD INT "sample_count"
    ADD STRING "timestamp"
```

## Component 3: Real-Time Processing Actions

### Main Sensor Data Processor
```lot
DEFINE ACTION ProcessSensorData
ON TOPIC "sensors/+/+/raw_data" DO
    SET "sensor_type" WITH TOPIC POSITION 1
    SET "sensor_id" WITH TOPIC POSITION 2
    
    // Call Python validation
    CALL PYTHON "SensorValidator.validate_sensor_reading"
        WITH (PAYLOAD, {sensor_id})
        RETURN AS {validated_data}
    
    // Extract validated fields
    SET "temperature" WITH (GET JSON "temperature" IN {validated_data} AS DOUBLE)
    SET "pressure" WITH (GET JSON "pressure" IN {validated_data} AS DOUBLE)
    SET "humidity" WITH (GET JSON "humidity" IN {validated_data} AS DOUBLE)
    SET "quality_score" WITH (GET JSON "quality_score" IN {validated_data} AS DOUBLE)
    SET "quality_status" WITH (GET JSON "quality_status" IN {validated_data} AS STRING)
    SET "all_valid" WITH (GET JSON "all_valid" IN {validated_data} AS BOOL)
    
    // Get sensor metadata
    SET "location" WITH GET TOPIC "sensors/" + {sensor_type} + "/" + {sensor_id} + "/metadata/location"
    
    // Publish validated sensor record
    PUBLISH MODEL ValidatedSensorRecord TO "sensors/validated/" + {sensor_type} + "/" + {sensor_id} WITH
        sensor_id = {sensor_id}
        sensor_type = {sensor_type}
        location = {location}
        temperature = {temperature}
        pressure = {pressure}
        humidity = {humidity}
        quality_score = {quality_score}
        quality_status = {quality_status}
        all_valid = {all_valid}
        timestamp = TIMESTAMP "UTC"
        validations = (GET JSON "validations" IN {validated_data})
    
    // Store raw reading for statistics
    SET "reading_count" WITH (GET TOPIC "sensors/" + {sensor_id} + "/stats/count" + 1)
    PUBLISH TOPIC "sensors/" + {sensor_id} + "/stats/count" WITH {reading_count}
    PUBLISH TOPIC "sensors/" + {sensor_id} + "/stats/readings/" + {reading_count} WITH {temperature}
    
    // Generate alarms for invalid readings
    IF {all_valid} EQUALS FALSE THEN
        IF {temperature} < -50 OR {temperature} > 150 THEN
            PUBLISH MODEL SensorAlarm TO "alarms/sensors/" + {sensor_id} + "/temperature" WITH
                alarm_id = (RANDOM UUID)
                sensor_id = {sensor_id}
                alarm_type = "TEMPERATURE_OUT_OF_RANGE"
                severity = "HIGH"
                message = "Temperature " + {temperature} + "°C outside valid range (-50 to 150°C)"
                trigger_value = {temperature}
                timestamp = TIMESTAMP "UTC"
                acknowledged = FALSE
        
        IF {pressure} < 0 OR {pressure} > 200 THEN
            PUBLISH MODEL SensorAlarm TO "alarms/sensors/" + {sensor_id} + "/pressure" WITH
                alarm_id = (RANDOM UUID)
                sensor_id = {sensor_id}
                alarm_type = "PRESSURE_OUT_OF_RANGE"
                severity = "CRITICAL"
                message = "Pressure " + {pressure} + " PSI outside valid range (0 to 200 PSI)"
                trigger_value = {pressure}
                timestamp = TIMESTAMP "UTC"
                acknowledged = FALSE
    
    // Publish to cloud bridge topic
    PUBLISH TOPIC "cloud/sensors/" + {sensor_type} + "/" + {sensor_id} + "/validated" WITH {validated_data}
```

## Component 4: Periodic Statistics Actions

### Hourly Statistics Calculator
```lot
DEFINE ACTION CalculateHourlyStatistics
ON EVERY 1 HOUR DO
    // Process each sensor type
    SET "sensor_types" WITH ["temperature", "pressure", "flow"]
    
    // For each active sensor (simplified - would use actual sensor list)
    SET "sensor_id" WITH "SENSOR001"
    
    // Collect last hour's readings
    SET "readings" WITH []
    SET "i" WITH 1
    
    // Collect up to 60 readings (one per minute)
    WHILE {i} <= 60 DO
        SET "reading" WITH GET TOPIC "sensors/" + {sensor_id} + "/stats/readings/" + {i}
        IF {reading} > 0 THEN
            SET "readings" WITH {readings} + [{reading}]
        SET "i" WITH ({i} + 1)
    
    // Calculate statistics using Python
    CALL PYTHON "SensorValidator.calculate_sensor_statistics"
        WITH (JSON.stringify({readings}))
        RETURN AS {stats}
    
    // Publish statistics model
    PUBLISH MODEL SensorStatistics TO "sensors/statistics/" + {sensor_id} + "/hourly" WITH
        sensor_id = {sensor_id}
        statistic_period = "HOURLY"
        mean = (GET JSON "mean" IN {stats} AS DOUBLE)
        median = (GET JSON "median" IN {stats} AS DOUBLE)
        std_dev = (GET JSON "std_dev" IN {stats} AS DOUBLE)
        min = (GET JSON "min" IN {stats} AS DOUBLE)
        max = (GET JSON "max" IN {stats} AS DOUBLE)
        range = (GET JSON "range" IN {stats} AS DOUBLE)
        sample_count = (GET JSON "count" IN {stats} AS INT)
        timestamp = TIMESTAMP "UTC"
    
    // Reset hourly counters
    PUBLISH TOPIC "sensors/" + {sensor_id} + "/stats/count" WITH 0
```

### System Health Monitor
```lot
DEFINE ACTION SystemHealthMonitor
ON EVERY 5 MINUTES DO
    // Check sensor activity in last 5 minutes
    SET "active_sensors" WITH 0
    SET "inactive_sensors" WITH 0
    
    // Check each sensor's last update time
    // (Simplified - would iterate through all sensors)
    SET "sensor_id" WITH "SENSOR001"
    SET "last_update" WITH GET TOPIC "sensors/validated/temperature/" + {sensor_id} + "/timestamp"
    
    // Calculate time since last update
    SET "current_time" WITH TIMESTAMP "UNIX"
    SET "time_diff" WITH ({current_time} - {last_update})
    
    IF {time_diff} < 300 THEN  // Less than 5 minutes
        SET "active_sensors" WITH ({active_sensors} + 1)
    ELSE
        SET "inactive_sensors" WITH ({inactive_sensors} + 1)
        PUBLISH TOPIC "alarms/system/sensor_offline" WITH "Sensor " + {sensor_id} + " offline for " + {time_diff} + " seconds"
    
    // Publish system health summary
    PUBLISH TOPIC "system/health/active_sensors" WITH {active_sensors}
    PUBLISH TOPIC "system/health/inactive_sensors" WITH {inactive_sensors}
    PUBLISH TOPIC "system/health/last_check" WITH TIMESTAMP "UTC"
```

## Component 5: MQTT Bridge to Cloud

```lot
DEFINE ROUTE CloudBridge WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER_ADDRESS "edge-broker.local"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "EdgeSensorClient"
        WITH USERNAME "edge_user"
        WITH PASSWORD "edge_pass"
    ADD DESTINATION_CONFIG
        WITH BROKER_ADDRESS "cloud.iot.company.com"
        WITH BROKER_PORT '8883'
        WITH CLIENT_ID "CloudSensorClient"
        WITH USERNAME "cloud_user"
        WITH PASSWORD "cloud_pass"
        WITH USE_TLS TRUE
        WITH RECONNECTION_RETRIES 10
    ADD MAPPING validated_sensors
        WITH SOURCE_TOPIC "cloud/sensors/#"
        WITH DESTINATION_TOPIC "iot/edge001/sensors/#"
        WITH DIRECTION "out"
    ADD MAPPING statistics
        WITH SOURCE_TOPIC "sensors/statistics/#"
        WITH DESTINATION_TOPIC "iot/edge001/statistics/#"
        WITH DIRECTION "out"
    ADD MAPPING alarms
        WITH SOURCE_TOPIC "alarms/sensors/#"
        WITH DESTINATION_TOPIC "iot/edge001/alarms/#"
        WITH DIRECTION "out"
```

## Component 6: Database Storage

```lot
DEFINE ROUTE SensorDataStorage WITH TYPE OPENSEARCH
    ADD OPENSEARCH_CONFIG
        WITH BASE_URL "http://opensearch:9200"
        WITH USERNAME "admin"
        WITH PASSWORD "admin"
        WITH USE_SSL "false"
        WITH IGNORE_CERT_ERRORS "true"
    ADD MAPPING sensor_validated_data
        WITH TOPIC "sensors/validated/#"
        WITH INDEX "sensor-validated-data"
    ADD MAPPING sensor_statistics
        WITH TOPIC "sensors/statistics/#"
        WITH INDEX "sensor-statistics"
    ADD MAPPING sensor_alarms
        WITH TOPIC "alarms/sensors/#"
        WITH INDEX "sensor-alarms"
```

## Testing the System

### 1. Publish Raw Sensor Data
```
Topic: sensors/temperature/SENSOR001/raw_data
Payload:
{
  "temperature": 25.5,
  "pressure": 101.3,
  "humidity": 45.2
}
```

### 2. Expected Data Flow

**Validated Record** → `sensors/validated/temperature/SENSOR001`:
```json
{
  "sensor_id": "SENSOR001",
  "sensor_type": "temperature",
  "location": "Factory Floor A",
  "temperature": 25.5,
  "pressure": 101.3,
  "humidity": 45.2,
  "quality_score": 100.0,
  "quality_status": "EXCELLENT",
  "all_valid": true,
  "timestamp": "2025-10-25T14:30:15Z",
  "validations": {
    "temperature_valid": true,
    "pressure_valid": true,
    "humidity_valid": true
  }
}
```

**Cloud Bridge** → `iot/edge001/sensors/temperature/SENSOR001/validated`

**Database** → OpenSearch index `sensor-validated-data`

### 3. Test Invalid Data
```
Topic: sensors/temperature/SENSOR002/raw_data
Payload:
{
  "temperature": 200.0,
  "pressure": -10.0,
  "humidity": 150.0
}
```

**Expected Alarms:**
- `alarms/sensors/SENSOR002/temperature` - Temperature out of range
- `alarms/sensors/SENSOR002/pressure` - Pressure out of range

## System Benefits

### Real-Time Processing
- Immediate data validation
- Instant alarm generation
- Sub-second response times

### Data Quality
- Python-based validation
- Comprehensive quality scoring
- Automatic error detection

### Scalability
- Wildcard topic patterns handle unlimited sensors
- Independent processing per sensor
- Horizontal scaling ready

### Cloud Integration
- Secure TLS connection to cloud
- Automatic data replication
- Bidirectional communication

### Data Persistence
- Historical data in OpenSearch
- Queryable sensor records
- Long-term trend analysis

### Monitoring
- Health checks every 5 minutes
- Hourly statistics
- Sensor activity tracking

## Extension Ideas

### 1. Add Predictive Maintenance
- Analyze sensor trends
- Predict equipment failures
- Schedule maintenance proactively

### 2. Machine Learning Integration
- Train anomaly detection models
- Real-time outlier identification
- Adaptive thresholds

### 3. Dashboard Integration
- Real-time sensor visualization
- Alarm management interface
- Historical trend charts

### 4. Multi-Site Deployment
- Replicate across multiple facilities
- Centralized monitoring
- Site-specific customization

## Related Documentation

- [Topic-Based Actions](../../Actions/Topic-Based-Actions/)
- [Time-Based Actions](../../Actions/Time-Based-Actions/)
- [Action Models](../../Models/Action-Models/)
- [Python Integration](../../Python/Simple-Python.md)
- [MQTT Bridge](../../Routes/DataPipeline/MQTT_BRIDGE.md)

[Back to Examples](../)

