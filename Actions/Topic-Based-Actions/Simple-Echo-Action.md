---
title: Simple Echo Action
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Basic topic-triggered action that echoes messages from one topic to another
related_tutorials: Tutorial/01-basics/topic-actions.lotnb
---

# Simple Echo Action

## Overview

A simple echo action demonstrates the most basic form of topic-based event-driven programming in LOT. It listens for messages on one topic and republishes them to another topic, creating a simple message relay system.

## Pattern

```lot
DEFINE ACTION SimpleEcho
ON TOPIC "input/message" DO
    PUBLISH TOPIC "output/message" WITH PAYLOAD
```

## How It Works

1. **Trigger**: Action activates when any message arrives on `input/message`
2. **Processing**: Captures the incoming payload
3. **Output**: Republishes the same payload to `output/message`

## Expected MQTT Topics

### Input Topics
- `input/message` - Receives the original message

### Output Topics
- `output/message` - Publishes the echoed message

## Example Payloads

**Publish to:** `input/message`  
**Payload:** `"Hello LOT!"`

**Result in:** `output/message`  
**Payload:** `"Hello LOT!"`

## Use Cases

- **Message Routing**: Forward messages between different topic namespaces
- **Topic Bridging**: Connect different parts of your MQTT architecture
- **Testing**: Verify action execution and topic connectivity
- **Logging**: Create message copies for audit trails

## Variations

### Echo with Timestamp

```lot
DEFINE ACTION EchoWithTimestamp
ON TOPIC "input/message" DO
    PUBLISH TOPIC "output/message" WITH PAYLOAD
    PUBLISH TOPIC "output/message/timestamp" WITH TIMESTAMP "UTC"
```

### Echo with Transformation

```lot
DEFINE ACTION EchoUppercase
ON TOPIC "input/message" DO
    PUBLISH TOPIC "output/message" WITH REPLACE(PAYLOAD, "a-z", "A-Z")
```

## Related Patterns

- [Sensor Data Router](./Sensor-Data-Router.md) - Using wildcards for flexible routing
- [Alarm Processor](./Alarm-Processor.md) - Processing with context extraction
- [Command Processor](./Command-Processor.md) - Handling device commands

[Back to Actions](../DEFINE%20ACTION.md)

