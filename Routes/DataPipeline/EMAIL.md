
# Route Type: `EMAIL`
> **Available in:**
> - Coreflux MQTT Broker >v1.6.2  
> - Linux ARM64  
> - Linux x64  
> - Windows x64  

## 1. Overview
- **Description:**  
  The `EMAIL` route allows you to send templated emails based on MQTT messages received on specific topics.  
  It's ideal for notifications, alerts, or reporting use cases. Email content is defined using HTML templates with placeholder support for dynamic payload rendering.

## 2. Signature
- **Syntax:**  
```lot
DEFINE ROUTE <route_name> WITH TYPE EMAIL
    ADD MQTT_CONFIG
        WITH BROKER_ADDRESS "<broker_address>"
        WITH BROKER_PORT '<port>'
        WITH CLIENT_ID "<client_id>"
        WITH USERNAME "<username>"
        WITH PASSWORD "<password>"
        WITH USE_TLS <true|false>
    ADD SMTP_CONFIG
        WITH HOST "<smtp_server>"
        WITH PORT '<smtp_port>'
        WITH USERNAME "<email_account>"
        WITH PASSWORD "<email_password>"
        WITH USE_TLS <true|false>
    ADD MAPPING <mapping_name>
        WITH SOURCE_TOPIC "<topic>"
        WITH TEMPLATE_PATH "<path_to_html_template>"
        WITH SUBJECT "<subject_template>"
        WITH RECIPIENT "<recipient_template>"
```

## 3. Configuration Parameters

### MQTT Configuration (`MQTT_CONFIG`)
- Same as in `MQTT_BRIDGE`:
  - `BROKER_ADDRESS`, `BROKER_PORT`, `CLIENT_ID`, `USERNAME`, `PASSWORD`, `USE_TLS`

### SMTP Configuration (`SMTP_CONFIG`)
- **HOST**: SMTP server address (e.g., `smtp.gmail.com`)
- **PORT**: SMTP server port (e.g., `587`)
- **USERNAME**: Email account to send from.
- **PASSWORD**: App password or actual password (for Gmail use an [App Password](https://support.google.com/accounts/answer/185833?hl=en)).
- **USE_TLS**: Enable secure email sending (recommended: `true`).

### Mapping Parameters
- **SOURCE_TOPIC**: MQTT topic that triggers the email.
- **TEMPLATE_PATH**: Path to the HTML file used as the email body.
- **SUBJECT**: Email subject. Can use placeholders (see below).
- **RECIPIENT**: Target email address. Can also use placeholders from the payload.

## 4. Placeholder Syntax

Email templates (both subject and body) support dynamic rendering from the MQTT message payload.

### Placeholders:
- `{value}`: Full JSON payload string.
- `{value.json.key}`: Inject a single value from the JSON.
- `{value.json.key.subkey}`: Support for nested values.

### Example Payload:
```json
{
  "device": "Sensor-A",
  "status": {
    "temp": 42,
    "humidity": 21
  }
}
```

### Example Usage:
- Subject: `Alert from {value.json.device}`
- Body: `<b>Temperature:</b> {value.json.status.temp}`

## 5. Usage Examples

### Example 1: Basic Email Route

```lot
DEFINE ROUTE TempAlert WITH TYPE EMAIL
    ADD MQTT_CONFIG
        WITH BROKER_ADDRESS "127.0.0.1"
        WITH BROKER_PORT '1883'
        WITH CLIENT_ID "EmailNotifier"
    ADD SMTP_CONFIG
        WITH HOST "smtp.gmail.com"
        WITH PORT '587'
        WITH USERNAME "alerts@mycompany.com"
        WITH PASSWORD "my-app-password"
        WITH USE_TLS true
    ADD MAPPING tempWarning
        WITH SOURCE_TOPIC "sensors/+/warning"
        WITH TEMPLATE_PATH "/templates/warning.html"
        WITH SUBJECT "Sensor Warning: {value.json.device}"
        WITH RECIPIENT "team@mycompany.com"
```

### Example 2: Dynamic Recipient and Subject

```lot
DEFINE ROUTE UserNotify WITH TYPE EMAIL
    ADD MQTT_CONFIG
        WITH BROKER SELF
    ADD SMTP_CONFIG
        WITH HOST "smtp.office365.com"
        WITH PORT '587'
        WITH USERNAME "noreply@org.com"
        WITH PASSWORD "secure-app-pass"
        WITH USE_TLS true
    ADD MAPPING notifyUser
        WITH SOURCE_TOPIC "users/+/notifications"
        WITH TEMPLATE_PATH "/templates/user_notify.html"
        WITH SUBJECT "Hello {value.json.username}, you have a new notification"
        WITH RECIPIENT "{value.json.email}"
```

## 6. Notes & Additional Info
- Supports wildcards in topics like `+` and `#`.
- Multiple mappings can be added to one route.
- Email body must be in HTML (plain text is not currently supported).
- Works well with dynamic payloads, sensors, and user events.
- For Google SMTP, ensure:
  - 2FA is enabled on the account.
  - App Passwords are used.

## Related Documentation
- [MQTT_BRIDGE](../System/MQTT_BRIDGE.md)
- [DEFINE ROUTE](../DEFINE%20ROUTE.md)

