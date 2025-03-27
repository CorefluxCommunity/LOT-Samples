# Entity Keyword: `TIMESTAMP`

> **Available in:**
> - Coreflux MQTT Broker >v1.5.1  
> - Linux ARM64
> - Linux x64
> - Windows x64

## 1. Overview
- **Description:**  
  Provides the current date and time in various predefined or custom formats. This entity can dynamically generate timestamps for use within actions or model definitions in LOT.

## 2. Signature
- **Syntax:**  
  ```lot
  TIMESTAMP WITH "<format>"
  ```

## 3. Supported Formats

### Built-in Formats

| Format    | Description                                | Example Output                     |
|-----------|--------------------------------------------|------------------------------------|
| `ISO`     | ISO 8601 standard format                   | `2025-03-17T14:25:30.123Z`         |
| `UTC`     | UTC date and time                          | `2025-03-17 14:25:30`              |
| `UNIX`    | UNIX timestamp (seconds since Unix epoch)  | `1742193930`                       |
| `UNIX-MS` | UNIX timestamp (milliseconds since epoch)  | `1742193930123`                    |

### Custom Format Strings
You can define a custom date-time format using special characters to specify date and time components.

| Symbol | Description                  | Example |
|--------|------------------------------|---------|
| `yyyy` | 4-digit year                 | `2025`  |
| `MM`   | 2-digit month (01-12)        | `03`    |
| `dd`   | 2-digit day (01-31)          | `17`    |
| `HH`   | 24-hour clock hour (00-23)   | `14`    |
| `mm`   | Minute (00-59)               | `25`    |
| `ss`   | Second (00-59)               | `30`    |
| `fff`  | Milliseconds (000-999)       | `123`   |
| `tt`   | AM/PM designator             | `PM`    |
| `zzz`  | Timezone offset              | `+00:00`|
| `dddd` | Full weekday name            | `Monday`|
| `MMM`  | Abbreviated month name       | `Mar`   |
| `MMMM` | Full month name              | `March` |

**Example Custom Formats:**
- `"yyyy/MM/dd HH:mm:ss"` → `2025/03/17 14:25:30`
- `"dd-MM-yyyy"` → `17-03-2025`
- `"HH:mm:ss.fff"` → `14:25:30.123`
- `"MM-dd-yyyy HH:mm:ss tt"` → `03-17-2025 02:25:30 PM`
- `"dddd, dd MMMM yyyy"` → `Monday, 17 March 2025`
- `"ddd, dd MMM yyyy"` → `Mon, 17 Mar 2025`

## 4. Compatible Keywords
- **`SET`**: Stores the generated timestamp into a variable.
- **`PUBLISH TOPIC`**: Publishes timestamp data externally.
- **`KEEP TOPIC`**: Stores timestamp data internally.
- **`REPLACE`**: Inserts dynamically generated timestamps into strings.

## 5. Return Value
- **Description:**
  Returns a formatted string representing the current timestamp based on the specified format.

## 6. Usage Examples

### Basic Example
Publishes current timestamp in ISO format every 10 seconds:

```lot
DEFINE ACTION PublishTimestampISO
ON EVERY 10 SECONDS DO
    PUBLISH TOPIC "time/iso" WITH TIMESTAMP WITH "ISO"
```
- **Output Example:**
  - Publishes: `2025-03-17T14:25:30.123Z`

### Intermediate Example
Stores a UNIX timestamp internally for device event logging:

```lot
DEFINE ACTION LogEvent
ON TOPIC "device/event" DO
    KEEP TOPIC "event/log" WITH TIMESTAMP WITH "UNIX"
```
- **Output Example:**
  - Stores internally: `1742193930`

### Advanced Example with Custom Format
Publishes a detailed timestamp for precise monitoring:

```lot
DEFINE ACTION DetailedMonitoringTimestamp
ON EVERY 1 MINUTE DO
    PUBLISH TOPIC "monitoring/timestamp" WITH TIMESTAMP WITH "yyyy-MM-dd HH:mm:ss.fff"
```
- **Output Example:**
  - Publishes: `2025-03-17 14:25:30.123`

### Usage with `REPLACE`
Inserting dynamic timestamps into message strings:

```lot
DEFINE ACTION DynamicTimestampMessage
ON TOPIC "alerts/generate" DO
    SET "AlertMessage" WITH REPLACE "<timestamp>" WITH TIMESTAMP WITH "UTC" IN "Alert triggered at <timestamp>"
    PUBLISH TOPIC "alerts/messages" WITH {AlertMessage}
```
- **Output Example:**
  - Publishes: `Alert triggered at 2025-03-17 14:25:30`

### Weekday and Month Name Example
Publishes a readable date including weekday and month names:

```lot
DEFINE ACTION ReadableDate
ON EVERY 1 DAY DO
    PUBLISH TOPIC "daily/date" WITH TIMESTAMP WITH "dddd, dd MMMM yyyy"
```
- **Output Example:**
  - Publishes: `Monday, 17 March 2025`

## 7. Notes & Additional Information
- **Timezone**: All built-in formats use Coordinated Universal Time (UTC).
- **Custom Formats**: Ensure your custom string follows correct formatting symbols as indicated above.
- Incorrect custom formats will result in an empty string output.

## 8. Related Entities
- [PAYLOAD](../PAYLOAD/PAYLOAD.md)
- [TOPIC](../TOPIC/TOPIC.md)

[Back to Entities](../Entities.md)