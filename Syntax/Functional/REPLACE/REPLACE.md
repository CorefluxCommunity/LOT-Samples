# KEYWORD: `REPLACE`
> **Available in:**
> - Coreflux MQTT Broker >v1.5.6  
> - Linux ARM64 
> - Linux x64 
> - Windows x64 
## 1. Overview
- **Description:**  
  Replaces specific data within payloads or topic names with alternative data. Useful for dynamic adjustments of payloads or topic addresses based on variable information.

## 2. Signature
- **Syntax:**  
  ```lot
  <SET | PUBLISH TOPIC | KEEP TOPIC> USING REPLACE "find" WITH "replace" IN <GET TOPIC | PAYLOAD | "string">
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used together:
- **`SET`**: Assigns dynamically modified content to variables for later use.
- **`PUBLISH TOPIC`**: Publishes content with dynamically replaced data.
- **`KEEP TOPIC`**: Internally stores content with dynamically replaced data.
- **`GET TOPIC`**: Sources original data for replacement operations.
- **`PAYLOAD`**: Uses incoming data payloads as a source for replacements.

## 4. Return Value
- **Description:**
  Returns the content specified in the `IN` section with the applied replacements.

## 5. Usage Examples

### Simple Example
When receiving a machine number in the topic `Selected/Machine`, the action replaces the placeholder `xx` with the payload's content. This updated string is then published to the `Message` topic.

```lot
DEFINE ACTION TestReplace
ON TOPIC "Selected/Machine"
DO
    SET "Message" WITH REPLACE "xx" WITH PAYLOAD IN "Machine xx was Selected!"
    PUBLISH TOPIC "Message" WITH {Message}
```
- **Example Scenario:**
  - Suppose `Selected/Machine` payload contains `12`.
  - Result: Topic `Message` will broadcast "Machine 12 was Selected!".

### Direct Usage with `PUBLISH` and `KEEP`
The keywords `PUBLISH TOPIC` and `KEEP TOPIC` can directly accept the `REPLACE` operation:

```lot
DEFINE ACTION TestReplaceDirect
ON TOPIC "Selected/+/Machine"
DO
    PUBLISH TOPIC "MessageExternally" WITH REPLACE "xx" WITH TOPIC POSITION 2 IN "Machine xx was Selected!"
```
- **Example Scenario:**
  - Suppose the TOPIC received is `Selected/07/Machine` the payload can be anything.
  - Result:
    - `MessageExternally` broadcasts "Machine 07 was Selected!" to subscribers.

## 6. Notes & Additional Information
- **Related Methods:**

  - [SET](../SET/SET.md)
  - [TRIM](../TRIM/TRIM.md)
  - [FILTER](../FILTER/FILTER.md)

[Back to Functions](../Functions.md)

