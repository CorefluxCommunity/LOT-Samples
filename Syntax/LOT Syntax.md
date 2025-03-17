# LOT Language Syntax Guide

## Overview
This document explains the syntax structure used in the LOT (Language of Things) language, clearly distinguishing between **Entities** (subjects or data) and **Functional Keywords** (imperative actions performed by the MQTT broker). The language aims to closely resemble natural English, making it intuitive and easy to read.

## Language Components

### 📌 **Entities (Subjects/Data)**
Entities represent the subjects or objects within the system, such as payloads, topics, timestamps, and structured data types. They are always described in an affirmative form.

**Example Entities:**
- `PAYLOAD`: Data received from an MQTT topic.
- `TOPIC`: An MQTT topic address.
- `TIMESTAMP`: Represents a point in time.
- `EMPTY`: Represents an empty or null value.

> [View All Entities](Entities/Entities.md)

### ⚙️ **Functional Keywords (Imperative Actions)**
Functional keywords are commands that instruct the broker to perform specific operations. They use an imperative form because the broker itself is the actor performing these actions.

**Example Functional Keywords:**
- `PUBLISH TOPIC`: Commands the broker to broadcast data.
- `KEEP TOPIC`: Commands the broker to store data internally.
- `SET`: Commands the broker to define or modify variables.
- `GET TOPIC`: Commands the broker to retrieve data from a specified topic.

> [View All Functional Keywords](Functional/Functional.md)

## Imperative vs. Affirmative Syntax

LOT clearly differentiates syntax based on who or what performs the action:

- **Imperative (Broker Actions):**
  - Example: `PUBLISH TOPIC "device/status" WITH PAYLOAD`
  - Actor: The MQTT Broker (performs publishing)

- **Affirmative (Data-driven descriptions):**
  - Example: `DEVICE "lamp/1" NEEDS TO PUBLISH "status/on"`
  - Actor: The defined entity or "THING" (describes the intent or condition of the data/entity)

## Syntax Examples

### Example 1: Imperative
Broker actively publishes data:
```lot
DEFINE ACTION CheckTemperature
ON TOPIC "sensor/temp"
DO
    PUBLISH TOPIC "alerts/high_temp" WITH PAYLOAD
```
- **Explanation:** The MQTT broker acts upon receiving data from "sensor/temp" and actively publishes the payload to "alerts/high_temp".

### Example 2: Affirmative
Entity defined with conditions or intents (future extension with DEFINE THING):
```lot
DEFINE THING DEVICE "lamp/1"
    NEEDS TO PUBLISH "status/on" WHEN CONDITION MET
```
- **Explanation:** The DEVICE entity "lamp/1" is described affirmatively. It has a requirement (intent) to publish to "status/on" based on certain conditions.

## Compatible Keywords
Entities and Functional Keywords are used together to construct meaningful LOT instructions:
- **Entities** represent data or objects handled.
- **Functional Keywords** are commands executed by the broker.

Example combination:
- `PUBLISH TOPIC` (command) WITH `PAYLOAD` (data entity)



[Back to Main ](../README.md)

