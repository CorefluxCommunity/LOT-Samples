# Route Type: `FILE_STORAGE`
> **Available in:**
> - Coreflux MQTT Broker >v1.6.2  
> - Linux ARM64  
> - Linux x64  
> - Windows x64  

## 1. Overview
- **Description:**  
  The `FILE_STORAGE` route enables storing MQTT message data in files using CSV or JSON formats.  
  It's ideal for data logging, historical data storage, and creating data archives for later analysis.

## 2. Signature
- **Syntax:**  
```lot
DEFINE ROUTE <route_name> WITH TYPE FILE_STORAGE
    ADD MQTT_CONFIG
        WITH BROKER_ADDRESS "<broker_address>"
        WITH BROKER_PORT '<port>'
        WITH CLIENT_ID "<client_id>"
        WITH USERNAME "<username>"
        WITH PASSWORD "<password>"
        WITH USE_TLS <true|false>
    ADD FILE_STORAGE_CONFIG
        WITH STORAGE_DIR "<directory_path>"
        WITH FORMAT "<CSV|JSON>"
        WITH FILE_PATTERN "<file_name_pattern>"
        WITH APPEND <true|false>
        WITH DATE_FORMAT "<date_format>"
        WITH CSV_DELIMITER "<delimiter>"
        WITH INCLUDE_TIMESTAMP <true|false>
        WITH MAX_RECORDS <number>
```

## 3. Configuration Parameters

### MQTT Configuration (`MQTT_CONFIG`)
- Same as in `MQTT_BRIDGE`:
  - `BROKER_ADDRESS`, `BROKER_PORT`, `CLIENT_ID`, `USERNAME`, `PASSWORD`, `USE_TLS`

### File Storage Configuration (`FILE_STORAGE_CONFIG`)
- **STORAGE_DIR**: Directory where files will be stored.
- **FORMAT**: Output format (`CSV` or `JSON`).
- **FILE_PATTERN**: Pattern for file names, where `{0}` is replaced with the table name (e.g., `data_{0}.json`).
- **APPEND**: Whether to append to existing files (`true`) or create new ones (`false`).
- **DATE_FORMAT**: Format for timestamp fields (e.g., `yyyy-MM-dd HH:mm:ss`).
- **CSV_DELIMITER**: Delimiter character for CSV files (default: `,`).
- **INCLUDE_TIMESTAMP**: Include a timestamp with each record (`true|false`).
- **MAX_RECORDS**: Maximum records per file (for file rotation).

## 4. Using the Route with STORE Keyword

To use the FILE_STORAGE route, use the STORE keyword in other routes:

```lot
STORE <file_storage_route_name> WITH TABLE_NAME "<table_name>"
```

The `STORE` command automatically directs data to the specified FILE_STORAGE route, with:
- **<file_storage_route_name>**: Name of the previously defined FILE_STORAGE route.
- **TABLE_NAME**: Logical name for the data table/collection, used in file naming.

## 5. Data Handling

### JSON Format
- JSON data is stored as an array of objects.
- Each MQTT message is converted to a JSON object.
- Timestamps are added automatically if `INCLUDE_TIMESTAMP` is `true`.
- Files can be rotated when they reach `MAX_RECORDS` entries.

### CSV Format
- First row contains headers based on the JSON fields in the first record.
- Each subsequent row contains values from MQTT messages.
- Special characters are properly escaped for CSV format.
- Timestamps are added automatically if `INCLUDE_TIMESTAMP` is `true`.

## 6. Usage Examples

### Example 1: Basic JSON Storage

```lot
DEFINE ROUTE TemperatureLogger WITH TYPE FILE_STORAGE
    ADD MQTT_CONFIG
        WITH BROKER_ADDRESS "127.0.0.1"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "DataLogger"
    ADD FILE_STORAGE_CONFIG
        WITH STORAGE_DIR "/data/temperature_logs"
        WITH FORMAT "JSON"
        WITH FILE_PATTERN "temp_{0}.json"
        WITH APPEND true
        WITH INCLUDE_TIMESTAMP true
        WITH MAX_RECORDS 1000
```

### Example 2: CSV Storage with Custom Settings

```lot
DEFINE ROUTE DeviceMetrics WITH TYPE FILE_STORAGE
    ADD MQTT_CONFIG
        WITH BROKER SELF
    ADD FILE_STORAGE_CONFIG
        WITH STORAGE_DIR "/metrics/daily"
        WITH FORMAT "CSV"
        WITH FILE_PATTERN "metrics_{0}_%Y%m%d.csv"
        WITH APPEND true
        WITH DATE_FORMAT "yyyy-MM-dd'T'HH:mm:ss.fffZ"
        WITH CSV_DELIMITER ";"
        WITH INCLUDE_TIMESTAMP true
        WITH MAX_RECORDS 5000
```

### Example 3: Using with STORE in Another Route

```lot
DEFINE ROUTE ProcessSensorData WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER_ADDRESS "sensor-broker.local"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "SensorBridge"
    ADD DESTINATION_CONFIG
        WITH BROKER SELF
    ADD MAPPING sensorData
        WITH SOURCE_TOPIC "sensors/+/data"
        WITH DESTINATION_TOPIC "processed/sensors/+/data"
        WITH DIRECTION "in"
    STORE DeviceMetrics WITH TABLE_NAME "sensor_readings"
```

## 7. Notes & Additional Info
- JSON schema is automatically determined from the first message.
- The route will create the storage directory if it doesn't exist.
- For CSV files, new fields found in later messages are not added to existing files.
- Multiple STORE commands can use the same FILE_STORAGE route with different TABLE_NAME values.
- The STORE keyword is designed for use within other routes to automatically direct data to storage.

## Related Documentation
- [MQTT_BRIDGE](../DataPipeline/MQTT_BRIDGE.md)
- [DEFINE ROUTE](../DEFINE%20ROUTE.md) 