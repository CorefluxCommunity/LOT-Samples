# Entity Keywords: `SECOND`, `SECONDS`, `MINUTE`, `MINUTES`, `HOUR`, `HOURS`, `DAY`, `DAYS`, `WEEK`, `WEEKS`, `MONTH`, `MONTHS`, `YEAR`, `YEARS`

## 1. Overview
- **Description:**  
  These entities define specific time intervals used to trigger actions periodically within the LOT language. They specify how frequently timed actions should be executed.

## 2. Signature
- **Syntax:**  
  ```lot
  ON EVERY <time_interval> <SECOND|SECONDS|MINUTE|MINUTES|HOUR|HOURS|DAY|DAYS|WEEK|WEEKS|MONTH|MONTHS|YEAR|YEARS>
  ```

## 3. Compatible Keywords
List and description of each KEYWORD that can be used in combination:
- **`ON EVERY`**: Specifies the repetition interval for actions.
- **`DEFINE ACTION`**: Defines logic blocks that are executed at specified intervals.

## 4. Return Value
- **Description:**
  No explicit return value. These keywords control the timing and periodicity of action execution.

## 5. Usage Examples

### Seconds Example
Increments a topic's value every second:

```lot
DEFINE ACTION IncrementActionSecond
ON EVERY 1 SECOND DO
    IF GET TOPIC "second/topic" == EMPTY THEN
        PUBLISH TOPIC "second/topic" WITH 0
    ELSE
        PUBLISH TOPIC "second/topic" WITH (GET TOPIC "second/topic" + 1)
```

### Minutes Example
Updates a counter every minute:

```lot
DEFINE ACTION IncrementActionMinute
ON EVERY 1 MINUTE DO
    IF GET TOPIC "minute/topic" == EMPTY THEN
        PUBLISH TOPIC "minute/topic" WITH 0
    ELSE
        PUBLISH TOPIC "minute/topic" WITH (GET TOPIC "minute/topic" + 1)
```

### Hours Example
Hourly execution for logging or counting:

```lot
DEFINE ACTION IncrementActionHour
ON EVERY 1 HOUR DO
    IF GET TOPIC "hour/topic" == EMPTY THEN
        PUBLISH TOPIC "hour/topic" WITH 0
    ELSE
        PUBLISH TOPIC "hour/topic" WITH (GET TOPIC "hour/topic" + 1)
```

### Days Example
Performs daily increments:

```lot
DEFINE ACTION IncrementActionDay
ON EVERY 1 DAY DO
    IF GET TOPIC "day/topic" == EMPTY THEN
        PUBLISH TOPIC "day/topic" WITH 0
    ELSE
        PUBLISH TOPIC "day/topic" WITH (GET TOPIC "day/topic" + 1)
```

### Weeks Example
Executes actions weekly:

```lot
DEFINE ACTION IncrementActionWeek
ON EVERY 1 WEEK DO
    IF GET TOPIC "week/topic" == EMPTY THEN
        PUBLISH TOPIC "week/topic" WITH 0
    ELSE
        PUBLISH TOPIC "week/topic" WITH (GET TOPIC "week/topic" + 1)
```

### Months Example
Monthly triggers:

```lot
DEFINE ACTION IncrementActionMonth
ON EVERY 1 MONTH DO
    IF GET TOPIC "month/topic" == EMPTY THEN
        PUBLISH TOPIC "month/topic" WITH 0
    ELSE
        PUBLISH TOPIC "month/topic" WITH (GET TOPIC "month/topic" + 1)
```

### Years Example
Yearly executions for long-term tracking:

```lot
DEFINE ACTION IncrementActionYear
ON EVERY 1 YEAR DO
    IF GET TOPIC "year/topic" == EMPTY THEN
        PUBLISH TOPIC "year/topic" WITH 0
    ELSE
        PUBLISH TOPIC "year/topic" WITH (GET TOPIC "year/topic" + 1)
```

## 6. Notes & Additional Information
- **Granularity:**  
  The choice of interval (SECONDS, MINUTES, HOURS, etc.) determines the granularity of your actions.

- **Performance:**  
  Shorter intervals (e.g., seconds) may consume more resources compared to longer intervals (e.g., hours, days).

- **Related Entities:**

  - [TIMESTAMP](../TIMESTAMP/TIMESTAMP.md)
  - [PAYLOAD](../PAYLOAD/PAYLOAD.md)

[Back to Entities](../Entities.md)

