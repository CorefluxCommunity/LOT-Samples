# Route Type: `OPENSEARCH`
> **Available in:**
> - Coreflux MQTT Broker >v1.6.2  
> - Linux ARM64  
> - Linux x64  
> - Windows x64  

## 1. Overview
- **Description:**  
  The `OPENSEARCH` route enables direct indexing of MQTT message data into OpenSearch instances.  
  It's ideal for real-time data analytics, log aggregation, and creating searchable data archives.

## 2. Signature
- **Syntax:**  
```lot
DEFINE ROUTE <route_name> WITH TYPE OPENSEARCH
    ADD MQTT_CONFIG
        WITH BROKER_ADDRESS "<broker_address>"
        WITH BROKER_PORT '<port>'
        WITH CLIENT_ID "<client_id>"
        WITH USERNAME "<username>"
        WITH PASSWORD "<password>"
        WITH USE_TLS <true|false>
    ADD OPENSEARCH_CONFIG
        WITH BASE_URL "<opensearch_url>"
        WITH USERNAME "<opensearch_username>"
        WITH PASSWORD "<opensearch_password>"
        WITH USE_SSL <true|false>
        WITH IGNORE_CERT_ERRORS <true|false>
```

## 3. Configuration Parameters

### MQTT Configuration (`MQTT_CONFIG`)
- Same as in `MQTT_BRIDGE`:
  - `BROKER_ADDRESS`, `BROKER_PORT`, `CLIENT_ID`, `USERNAME`, `PASSWORD`, `USE_TLS`

### OpenSearch Configuration (`OPENSEARCH_CONFIG`)
- **BASE_URL**: URL of the OpenSearch instance (e.g., `https://opensearch-host:9200`).
- **USERNAME**: OpenSearch authentication username.
- **PASSWORD**: OpenSearch authentication password.
- **USE_SSL**: Enable secure connection to OpenSearch (`true|false`).
- **IGNORE_CERT_ERRORS**: Bypass certificate validation for development environments (`true|false`).

## 4. Using the Route with STORE Keyword

To use the OPENSEARCH route, use the STORE keyword in other routes:

```lot
STORE <opensearch_route_name> WITH INDEX_NAME "<index_name>"
```

The `STORE` command automatically directs data to the specified OPENSEARCH route, with:
- **<opensearch_route_name>**: Name of the previously defined OPENSEARCH route.
- **INDEX_NAME**: Name of the OpenSearch index to store data.

## 5. Data Handling

### Document Indexing
- Each MQTT message is indexed as a separate document in OpenSearch.
- Messages are expected to be in JSON format.
- Documents are indexed at the endpoint `/{index_name}/_doc`.
- No automatic ID generation is provided; OpenSearch assigns default IDs.

## 6. Usage Examples

### Example 1: Basic OpenSearch Indexing

```lot
DEFINE ROUTE SensorDataIndexer WITH TYPE OPENSEARCH
    ADD MQTT_CONFIG
        WITH BROKER_ADDRESS "127.0.0.1"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "OpenSearchIndexer"
    ADD OPENSEARCH_CONFIG
        WITH BASE_URL "https://opensearch.example.com:9200"
        WITH USERNAME "admin"
        WITH PASSWORD "secure-password"
        WITH USE_SSL true
```

### Example 2: Local Development Setup

```lot
DEFINE ROUTE DeviceLogsIndexer WITH TYPE OPENSEARCH
    ADD MQTT_CONFIG
        WITH BROKER SELF
    ADD OPENSEARCH_CONFIG
        WITH BASE_URL "http://localhost:9200"
        WITH USE_SSL false
        WITH IGNORE_CERT_ERRORS true
```

### Example 3: Using with STORE in Another Route

```lot
DEFINE ROUTE ProcessSensorAlerts WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER_ADDRESS "alert-broker.local"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "AlertBridge"
    ADD DESTINATION_CONFIG
        WITH BROKER SELF
    ADD MAPPING alertData
        WITH SOURCE_TOPIC "alerts/+/critical"
        WITH DESTINATION_TOPIC "processed/alerts/+/critical"
        WITH DIRECTION "in"
    STORE SensorDataIndexer WITH INDEX_NAME "critical_alerts"
```

## 7. Notes & Additional Info
- Multiple STORE commands can use the same OPENSEARCH route with different INDEX_NAME values.
- For production environments, set `IGNORE_CERT_ERRORS` to `false`.
- Authentication is optional for unsecured OpenSearch instances.
- Error handling is provided for failed indexing operations.
- Compatible with both OpenSearch and Elasticsearch API endpoints.
- The STORE keyword is designed for use within other routes to automatically direct data to storage.

## Related Documentation
- [MQTT_BRIDGE](../DataPipeline/MQTT_BRIDGE.md)
- [DEFINE ROUTE](../DEFINE%20ROUTE.md) 