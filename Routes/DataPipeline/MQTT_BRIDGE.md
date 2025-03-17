# Route Type: `MQTT_BRIDGE`
> **Available in:**
> - Coreflux MQTT Broker >v1.5.6  
> - Linux ARM64 
> - Linux x64 
> - Windows x64 
## 1. Overview
- **Description:**  
  The `MQTT_BRIDGE` route in Coreflux enables seamless data transfer between two MQTT brokers. This route is ideal for connecting local and remote MQTT brokers securely and efficiently.

## 2. Signature
- **Syntax:**  
  ```lot
  DEFINE ROUTE <route_name> WITH TYPE MQTT_BRIDGE
      ADD SOURCE_CONFIG
          WITH BROKER_ADDRESS "<source_broker_address>"
          WITH BROKER_PORT '<source_broker_port>'
          WITH CLIENT_ID "<source_client_id>"
          WITH USERNAME "<username>"
          WITH PASSWORD "<password>"
          WITH USE_TLS <true|false>
          WITH ALLOW_UNTRUSTED_CERTS <true|false>
          WITH SERVER_CA_CERT_PATH "<path>"
          WITH CLIENT_CERT_PATH "<path>"
          WITH CLIENT_CERT_PASS "<password>"
      ADD DESTINATION_CONFIG
          WITH BROKER_ADDRESS "<destination_broker_address>"
          WITH BROKER_PORT '<destination_broker_port>'
          WITH CLIENT_ID "<client_id>"
          WITH USERNAME "<username>"
          WITH PASSWORD "<password>"
      ADD MAPPING <mapping_name>
          WITH SOURCE_TOPIC "<source_topic>"
          WITH DESTINATION_TOPIC "<destination_topic>"
          WITH DIRECTION "<out|in|both>"
  ```

## 3. Configuration Parameters

### General Configuration
- **BROKER_ADDRESS**: Address of the MQTT broker (source or destination).
- **BROKER_PORT**: MQTT broker port.
- **CLIENT_ID**: Unique identifier for the MQTT client.
- **USERNAME** and **PASSWORD**: Credentials for authentication (optional).
- **DIRECTION**: Defines message flow (`out`, `in`, or `both`).

### Security (TLS) Configuration
- **USE_TLS**: Enables secure communication via TLS.
- **ALLOW_UNTRUSTED_CERTS**: Allows accepting untrusted certificates for development environments.
- **SERVER_CA_CERT_PATH**: Path to CA certificate for server validation.
- **CLIENT_CERT_PATH**: Path to client certificate (for mutual TLS).
- **CLIENT_CERT_PASS**: Password to access the client certificate.

### Wildcard Usage in Topic Mappings
- `+`: Single-level wildcard.
- `#`: Multi-level wildcard.

**Example:**
- Source topic: `devices/+/temperature`
- Destination topic: `bridge/devices/+/temp`
- Message published to `devices/sensor1/temperature` routed to `results/sensor1/temp`

## 4. Usage Examples

### Example 1: Basic MQTT Bridge

```lot
DEFINE ROUTE BasicMQTTBridge WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER_ADDRESS "192.168.1.10"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "LocalBridgeClient"
    ADD DESTINATION_CONFIG
        WITH BROKER_ADDRESS "iot.example.com"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "RemoteClient"
    ADD MAPPING sensorMapping
        WITH SOURCE_TOPIC "sensor/value"
        WITH DESTINATION_TOPIC "remote/sensor/value"
        WITH DIRECTION "out"
```

### Example 2: Secure MQTT Bridge (TLS)

```lot
DEFINE ROUTE SecureMQTTBridge WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER_ADDRESS "192.168.1.10"
        WITH BROKER_PORT '8883'
        WITH CLIENT_ID "SecureLocalClient"
        WITH USE_TLS true
        WITH ALLOW_UNTRUSTED_CERTS false
        WITH SERVER_CA_CERT_PATH "/path/to/ca.pem"
        WITH CLIENT_CERT_PATH "/path/to/client.pem"
        WITH CLIENT_CERT_PASS "certificate_password"
    ADD DESTINATION_CONFIG
        WITH BROKER_ADDRESS "secure.cloudbroker.com"
        WITH BROKER_PORT '8883'
        WITH CLIENT_ID "CloudClient"
        WITH USERNAME "user"
        WITH PASSWORD "secure_password"
        WITH USE_TLS true
        WITH SERVER_CA_CERT_PATH "/path/to/remote/ca.pem"
    ADD MAPPING secureSensorMapping
        WITH SOURCE_TOPIC "secure/sensor/+"
        WITH DESTINATION_TOPIC "cloud/sensor/+"
        WITH DIRECTION "both"
```

### Example 3: MQTT Bridge using Wildcards

```lot
DEFINE ROUTE WildcardMQTTBridge WITH TYPE MQTT_BRIDGE
    ADD SOURCE_CONFIG
        WITH BROKER SELF
    ADD DESTINATION_CONFIG
        WITH BROKER_ADDRESS "broker.remote.com"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "WildcardClient"
    ADD MAPPING wildcardMapping
        WITH SOURCE_TOPIC "room/+/status"
        WITH DESTINATION_TOPIC "remote/room/+/status"
        WITH DIRECTION "out"
```

## 5. Notes & Additional Information
- **TLS Security**: Ensure proper certificate management for production-grade security.
- **Wildcard usage**: Enables efficient management of multiple topic hierarchies.
- **Direction settings**:
  - `out`: Messages from SOURCE to DESTINATION.
  - `in`: Messages from DESTINATION broker received by SOURCE broker.
  - `both`: Bi-directional communication.

## Related Documentation
- [MQTT_CLUSTER](../System/MQTT_CLUSTER.md)

[Back to Routes](../DEFINE%20ROUTE.md)

