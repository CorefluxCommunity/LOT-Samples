# KEYWORD: `DEFINE ROUTE`

## 1. Overview

- **Description:**
  Defines configuration routes within the Coreflux MQTT Broker, enabling broker-to-broker communication, data storage integrations, and pipeline management. Routes allow the broker to dynamically manage data flow, internal broker architecture, and external system integrations.

## 2. Signature

- **Syntax:**
  ```lot
  DEFINE ROUTE <route_name> WITH TYPE <route_type>
      ADD <CONFIGURATION_TYPE>
          WITH <configuration_parameters>
      ADD MAPPING <mapping_name>
          WITH SOURCE_TOPIC <source_topic>
          WITH DESTINATION_TOPIC <destination_topic>
          WITH DIRECTION <direction>
  ```

## 3. Supported Route Types

# LOT Routes by Type

## 🔧 **System Routes**

| Route Name      | Description                                    | Documentation                                                   |
|-----------------|------------------------------------------------|-----------------------------------------------------------------|
| `MQTT_CLUSTER`  | Enables clustering and broker scalability.     | [View Docs](../Routes/System/MQTT_CLUSTER.md)                   |

## 📚 **Data Storage Routes**

| Route Name        | Description                                  | Documentation                                  |
|-------------------|----------------------------------------------|------------------------------------------------|
| `CrateDB`         | Stores data into a CrateDB database.         | [View Docs](../Routes/DataStorage/CrateDB.md)  |
| `OpenSearch`      | Stores data into OpenSearch database.        | [View Docs](../Routes/DataStorage/OpenSearch.md)|
| `Elastic`         | Stores data into Elasticsearch database.     | [View Docs](../Routes/DataStorage/Elastic.md)  |
| `Snowflake`       | Stores data into Snowflake database. *(Coming Soon)*| Coming Soon                             |
| `MySQL`           | Stores data into MySQL database. *(Coming Soon)*| Route unavailable - use Connector in broker (Assets) |
| `SQL Server`      | Stores data into SQL Server database. *(Coming Soon)*| Route unavailable - use Connector in broker (Assets) |
| `PostgreSQL`      | Stores data into PostgreSQL database. *(Coming Soon)*| Route unavailable - use Connector in broker (Assets) |
| `MariaDB`         | Stores data into MariaDB database. *(Coming Soon)*| Route unavailable - use Connector in broker (Assets) |
| `MongoDB`         | Stores data into MongoDB database. *(Coming Soon)*| Route unavailable - use Connector in broker (Assets) |
| `Firebase`        | Stores data into Firebase. *(Coming Soon)*   | Route unavailable - use Connector in broker (Assets) |

## 🚀 **Data Pipeline Routes**

| Route Name    | Description                                      | Documentation                                               |
|---------------|--------------------------------------------------|-------------------------------------------------------------|
| `MQTT_BRIDGE` | Enables data flow between MQTT brokers.          | [View Docs](../Routes/DataPipeline/MQTT_BRIDGE.md)          |
| `Kafka`       | Enables integration with Kafka pipeline. *(Coming Soon)* | Coming Soon                                     |







## 4. Route Usage Examples

### 4.1 MQTT Cluster Route (System Route)

Defines a route for MQTT clustering:

```lot
DEFINE ROUTE MqttClusterRoute WITH TYPE MQTT_CLUSTER
    ADD CLUSTER_CONFIG
        WITH BROKER_ADDRESS "127.0.0.1"
        WITH BROKER_PORT '1884'
        WITH CLIENT_ID "ClusterNode1"
        WITH USERNAME "root"
        WITH PASSWORD "coreflux"
        WITH MESSAGE_VALIDITY_SEC '30'
```

### 5.2 Opensearch Route (Data Storage Route)

Integrates MQTT broker with a database like OpenSearch:

```lot
DEFINE ROUTE OpenSearch_Connection WITH TYPE OPENSEARCH
    ADD OPENSEARCH_CONFIG
        WITH BASE_URL "address"
        WITH USERNAME "user"
        WITH PASSWORD "pass"
        WITH USE_SSL "true"
        WITH IGNORE_CERT_ERRORS "false"
```

### 5.3 MQTT Bridge Route (Data Pipeline Route)

Configures MQTT bridge communication:

```lot
DEFINE ROUTE MqttBridgeRoute WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER SELF
    ADD DESTINATION_CONFIG
        WITH BROKER_ADDRESS "iot.coreflux.cloud"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "RemoteClient"
    ADD MAPPING temperature
        WITH SOURCE_TOPIC "room/temp"
        WITH DESTINATION_TOPIC "source/room/temp"
        WITH DIRECTION "out"
```

##



## Related Documents

- [MQTT Bridge Route Implementation](../Routes/DataPipeline/MQTT_BRIDGE.md)
- [MQTT Cluster Route Implementation](../Routes/System/MQTT_CLUSTER.md)

[Back to Main Documentation](../README.md)

