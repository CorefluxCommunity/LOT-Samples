---
title: Math Calculator Action
category: Action
version: 1.0.0
last_updated: 2025-10-25
description: Use Python functions for mathematical calculations beyond basic LOT operators
related_tutorials: Tutorial/05-python/simple-python.lotnb
---

# Math Calculator Action

## Overview

The Math Calculator Action demonstrates how to integrate Python functions with LOT actions for complex mathematical operations. Python provides access to advanced math libraries, error handling, and floating-point precision that extend LOT's built-in capabilities.

## Pattern

### Python Script

```python
# Script Name: MathCalculator
def add_numbers(a, b):
    """Add two numbers and return the result"""
    try:
        result = float(a) + float(b)
        return result
    except (ValueError, TypeError):
        return 0

def calculate_percentage(value, total):
    """Calculate percentage with error handling"""
    try:
        if float(total) == 0:
            return 0
        percentage = (float(value) / float(total)) * 100
        return round(percentage, 2)
    except (ValueError, TypeError, ZeroDivisionError):
        return 0

def format_temperature(celsius):
    """Convert Celsius to Fahrenheit and format nicely"""
    try:
        celsius_val = float(celsius)
        fahrenheit = (celsius_val * 9/5) + 32
        return f"{celsius_val}°C ({fahrenheit:.1f}°F)"
    except (ValueError, TypeError):
        return "Invalid temperature"
```

### LOT Action

```lot
DEFINE ACTION SimpleMathProcessor
ON TOPIC "math/+/calculate" DO
    SET "operation" WITH TOPIC POSITION 1
    
    IF {operation} EQUALS "add" THEN
        CALL PYTHON "MathCalculator.add_numbers"
            WITH (5, 3)
            RETURN AS {result}
        PUBLISH TOPIC "math/add/result" WITH {result}
    
    ELSE IF {operation} EQUALS "percentage" THEN
        CALL PYTHON "MathCalculator.calculate_percentage"
            WITH (GET TOPIC "math/current/value", GET TOPIC "math/current/total")
            RETURN AS {percentage}
        PUBLISH TOPIC "math/percentage/result" WITH {percentage}
    
    ELSE IF {operation} EQUALS "temperature" THEN
        CALL PYTHON "MathCalculator.format_temperature"
            WITH (PAYLOAD)
            RETURN AS {formatted_temp}
        PUBLISH TOPIC "math/temperature/formatted" WITH {formatted_temp}
```

## How It Works

1. **Topic Trigger**: Action activates on `math/{operation}/calculate`
2. **Operation Selection**: Extracts operation type from topic
3. **Python Function Call**: `CALL PYTHON` invokes the appropriate function
4. **Parameter Passing**: `WITH (...)` sends parameters to Python
5. **Return Handling**: `RETURN AS {variable}` captures the result
6. **Result Publishing**: Publishes Python function output to MQTT

## CALL PYTHON Syntax

```lot
CALL PYTHON "ScriptName.function_name"
    WITH (param1, param2, ...)
    RETURN AS {variable_name}
```

- **ScriptName**: Name of the Python script (without .py extension)
- **function_name**: Name of the Python function to call
- **WITH (...)**: Parameters passed to the function
- **RETURN AS**: Variable to store the return value

## Expected MQTT Topics

### Input Topics
- `math/add/calculate` - Trigger addition
- `math/percentage/calculate` - Trigger percentage calculation
- `math/temperature/calculate` - Trigger temperature conversion

### Output Topics
- `math/add/result` - Addition result
- `math/percentage/result` - Percentage calculation result
- `math/temperature/formatted` - Formatted temperature string

## Example Payloads

**Example 1: Temperature Conversion**

**Publish to:** `math/temperature/calculate`  
**Payload:** `25`

**Result in:** `math/temperature/formatted`  
**Payload:** `"25°C (77.0°F)"`

**Example 2: Percentage Calculation**

**Set topics:**
- `math/current/value`: `95`
- `math/current/total`: `100`

**Publish to:** `math/percentage/calculate`  
**Payload:** (any value)

**Result in:** `math/percentage/result`  
**Payload:** `95.0`

## Use Cases

- **Complex Calculations**: Operations beyond basic LOT math operators
- **Unit Conversions**: Temperature, pressure, distance conversions
- **Statistical Analysis**: Mean, median, standard deviation calculations
- **Data Formatting**: String formatting with Python's capabilities

## Python Benefits

### Error Handling
Python provides robust error handling:
```python
try:
    result = complex_calculation(input_value)
    return result
except (ValueError, TypeError, ZeroDivisionError):
    return default_value
```

### Math Libraries
Access to Python's math ecosystem:
```python
import math
import statistics
import numpy as np  # If available

def advanced_math(values):
    return math.sqrt(statistics.mean(values))
```

### Type Conversion
Explicit type handling:
```python
def safe_add(a, b):
    return float(a) + float(b)
```

## Advanced Variations

### Scientific Calculator

```python
# Script Name: ScientificCalculator
import math

def calculate_statistics(values_json):
    """Calculate mean, median, std deviation"""
    import json
    try:
        values = json.loads(values_json)
        if not values:
            return {"error": "No values provided"}
        
        mean = sum(values) / len(values)
        sorted_vals = sorted(values)
        n = len(sorted_vals)
        median = sorted_vals[n//2] if n % 2 else (sorted_vals[n//2-1] + sorted_vals[n//2]) / 2
        
        variance = sum((x - mean) ** 2 for x in values) / len(values)
        std_dev = math.sqrt(variance)
        
        return {
            "mean": round(mean, 2),
            "median": round(median, 2),
            "std_dev": round(std_dev, 2),
            "count": len(values)
        }
    except Exception as e:
        return {"error": str(e)}
```

```lot
DEFINE ACTION StatisticalProcessor
ON TOPIC "stats/calculate" DO
    CALL PYTHON "ScientificCalculator.calculate_statistics"
        WITH (PAYLOAD)
        RETURN AS {stats}
    
    PUBLISH TOPIC "stats/results" WITH {stats}
    PUBLISH TOPIC "stats/mean" WITH (GET JSON "mean" IN {stats} AS DOUBLE)
    PUBLISH TOPIC "stats/median" WITH (GET JSON "median" IN {stats} AS DOUBLE)
```

### Engineering Calculator

```python
# Script Name: EngineeringCalc
def convert_units(value, from_unit, to_unit):
    """Convert between engineering units"""
    val = float(value)
    
    # Pressure conversions
    if from_unit == "psi" and to_unit == "bar":
        return val * 0.0689476
    elif from_unit == "bar" and to_unit == "psi":
        return val * 14.5038
    
    # Flow conversions
    elif from_unit == "gpm" and to_unit == "lpm":
        return val * 3.78541
    elif from_unit == "lpm" and to_unit == "gpm":
        return val / 3.78541
    
    return val  # No conversion
```

## Related Patterns

- [Data Validation Action](./Data-Validation-Action.md) - Python-based validation
- [Text Processing Action](./Text-Processing-Action.md) - String manipulation with Python

[Back to Actions](../DEFINE%20ACTION.md)

