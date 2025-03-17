# Entity Keyword: `REPLACE`

## 1. Overview
- **Description:**  
 When an action is triggered by a topic, this Entity keeps the data that triggered the event, in this case the payload itself. 

## 2. Signature
- **Syntax:**  
  ```lot
  PAYLOAD
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used together:
- **`SET`**: Assigns PAYLOAD to a variable.
- **`PUBLISH TOPIC`**: Publishes content with the PAYLOAD and make a copy of the topic. 
- **`KEEP TOPIC`**: Internally stores content with the PAYLOAD and make a copy of the topic. 
- **`GET TOPIC`**: Get another topic and interact with the PAYLOAD. 
- **`IF`**: Make conditional code using the payload , by checking the status.

## 4. Return Value
- **Description:**
  Returns the content of the topic that triggered a specific ACTION. 

## 5. Usage Examples

### Basic Example
```lot
DEFINE ACTION IncrementActionTriggered
ON TOPIC "input/topic" DO
     PUBLISH TOPIC "output/topic" WITH (PAYLOAD+1)
```


### Intermediate Example 
```lot
DEFINE ACTION CountTrigger
ON TOPIC "Plant/Equipmemnts/+/di1" DO
    IF GET TOPIC "Plant/Machines/+/Production/Counter" EQUAL EMPTY THEN
        PUBLISH TOPIC "Plant/Machines/+/Production/Counter" WITH 0
	IF PAYLOAD == "True" THEN
		PUBLISH TOPIC "Plant/Machines/+/Production/Counter" WITH (GET TOPIC "Plant/Machines/+/Production/Counter"+1)
```

### Advanced Example
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
ON TOPIC "Selected/Machine"
DO
    PUBLISH TOPIC "MessageExternally" WITH REPLACE "xx" WITH PAYLOAD IN "Machine xx was Selected!"
    KEEP TOPIC "MessageInternally" WITH REPLACE "xx" WITH PAYLOAD IN "Machine xx was Selected!"
```
- **Example Scenario:**
  - Suppose the payload received is `07`.
  - Result:
    - `MessageExternally` broadcasts "Machine 07 was Selected!" to subscribers.
    - `MessageInternally` stores "Machine 07 was Selected!" internally.

## 6. Notes & Additional Information
- **Related Entities:**

  - [TIMESTAMP](../TIMESTAMP/TIMESTAMP.md)
  - [TOPIC](../TOPIC/TOPIC.md)


[Back to Entities](../Entities.md)

