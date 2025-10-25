---
title: Simple Python Integration
category: Python
version: 1.0.0
last_updated: 2025-10-25
description: Integrate Python functions with LOT actions for complex calculations, data validation, and text processing
related_tutorials: Tutorial/05-python/simple-python.lotnb
---

# Simple Python Integration

## Overview

Python integration extends LOT's capabilities with full programming language features:
- Complex calculations and algorithms
- External library integration
- File operations and data processing
- Advanced string manipulation and parsing
- Data validation and transformation

## CALL PYTHON Syntax

```lot
CALL PYTHON "ScriptName.function_name"
    WITH (parameter1, parameter2, ...)
    RETURN AS {variable_name}
```

### Components
- **ScriptName**: Python script name (without .py extension)
- **function_name**: Function to call within the script
- **WITH (...)**: Parameters passed to the function
- **RETURN AS**: Variable to store the return value

## Python Script Structure

```python
# Script Name: MyScript
def my_function(param1, param2):
    """Function description"""
    try:
        # Your Python logic here
        result = process_data(param1, param2)
        return result
    except Exception as e:
        return {"error": str(e)}
```

### Best Practices for Python Functions
1. **Always include error handling** with try/except
2. **Return simple types** (str, int, float, bool) or JSON-serializable dicts/lists
3. **Document functions** with docstrings
4. **Validate inputs** before processing
5. **Handle None/null cases** gracefully

## Parameter Passing

### Single Parameter
```lot
CALL PYTHON "Processor.validate" WITH (PAYLOAD) RETURN AS {result}
```

### Multiple Parameters
```lot
CALL PYTHON "Calculator.add" WITH (5, 3) RETURN AS {sum}
CALL PYTHON "Validator.check_range" WITH (PAYLOAD, 0, 100) RETURN AS {valid}
```

### Mixed Types
```lot
CALL PYTHON "Formatter.format_data" 
    WITH (PAYLOAD, "celsius", TRUE, 2) 
    RETURN AS {formatted}
```

### Using Variables and Topics
```lot
SET "min_val" WITH GET TOPIC "config/min"
SET "max_val" WITH GET TOPIC "config/max"
CALL PYTHON "Validator.check_range" 
    WITH (PAYLOAD, {min_val}, {max_val}) 
    RETURN AS {result}
```

## Common Use Cases

### 1. Mathematical Calculations

```python
# Script Name: MathCalculator
import math

def calculate_statistics(values_json):
    """Calculate mean, median, std deviation from JSON array"""
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
            "min": min(values),
            "max": max(values),
            "count": len(values)
        }
    except Exception as e:
        return {"error": str(e)}

def convert_temperature(celsius):
    """Convert Celsius to Fahrenheit"""
    try:
        c = float(celsius)
        f = (c * 9/5) + 32
        return {"celsius": c, "fahrenheit": round(f, 1)}
    except (ValueError, TypeError):
        return {"error": "Invalid temperature"}
```

```lot
DEFINE ACTION StatisticalProcessor
ON TOPIC "data/statistics/calculate" DO
    CALL PYTHON "MathCalculator.calculate_statistics"
        WITH (PAYLOAD)
        RETURN AS {stats}
    
    PUBLISH TOPIC "data/statistics/results" WITH {stats}
    PUBLISH TOPIC "data/statistics/mean" WITH (GET JSON "mean" IN {stats} AS DOUBLE)
    PUBLISH TOPIC "data/statistics/median" WITH (GET JSON "median" IN {stats} AS DOUBLE)
```

### 2. Data Validation

```python
# Script Name: DataValidator
def validate_sensor_reading(value, min_val, max_val):
    """Validate sensor reading against acceptable range"""
    try:
        reading = float(value)
        min_limit = float(min_val)
        max_limit = float(max_val)
        
        if reading < min_limit:
            return {
                "valid": False,
                "reason": "Below minimum",
                "value": reading,
                "min_limit": min_limit,
                "max_limit": max_limit,
                "deviation": min_limit - reading
            }
        elif reading > max_limit:
            return {
                "valid": False,
                "reason": "Above maximum",
                "value": reading,
                "min_limit": min_limit,
                "max_limit": max_limit,
                "deviation": reading - max_limit
            }
        else:
            return {
                "valid": True,
                "reason": "Within range",
                "value": reading,
                "min_limit": min_limit,
                "max_limit": max_limit
            }
    except (ValueError, TypeError) as e:
        return {
            "valid": False,
            "reason": f"Invalid data: {str(e)}",
            "value": value
        }

def check_equipment_id_format(equipment_id):
    """Validate equipment ID format (e.g., PMP001)"""
    import re
    try:
        # Expected: 3 letters + 3 digits
        pattern = r'^[A-Z]{3}[0-9]{3}$'
        
        if re.match(pattern, str(equipment_id)):
            return {
                "valid": True,
                "format": "Standard",
                "type_code": equipment_id[:3],
                "number": equipment_id[3:]
            }
        else:
            return {
                "valid": False,
                "format": "Invalid",
                "expected": "3 letters + 3 digits (e.g., PMP001)"
            }
    except Exception as e:
        return {"valid": False, "error": str(e)}
```

```lot
DEFINE ACTION ValidationProcessor
ON TOPIC "validation/+/check" DO
    SET "validation_type" WITH TOPIC POSITION 1
    
    IF {validation_type} EQUALS "sensor" THEN
        CALL PYTHON "DataValidator.validate_sensor_reading"
            WITH (PAYLOAD, GET TOPIC "validation/sensor/min", GET TOPIC "validation/sensor/max")
            RETURN AS {validation_result}
        
        SET "is_valid" WITH (GET JSON "valid" IN {validation_result} AS BOOL)
        IF {is_valid} EQUALS TRUE THEN
            PUBLISH TOPIC "validation/sensor/passed" WITH {validation_result}
            PUBLISH TOPIC "sensors/validated/reading" WITH PAYLOAD
        ELSE
            PUBLISH TOPIC "validation/sensor/failed" WITH {validation_result}
            PUBLISH TOPIC "alarms/validation/sensor" WITH (GET JSON "reason" IN {validation_result} AS STRING)
```

### 3. Text Processing

```python
# Script Name: TextProcessor
def clean_sensor_data(raw_data):
    """Clean and validate sensor data string"""
    import re
    try:
        cleaned = str(raw_data).strip().upper()
        
        if len(cleaned) == 0:
            return "INVALID_DATA"
        
        # Remove special characters except numbers, letters, dots, underscores, hyphens
        cleaned = re.sub(r'[^A-Z0-9._-]', '', cleaned)
        
        return cleaned
    except Exception as e:
        return f"ERROR: {str(e)}"

def parse_equipment_code(equipment_string):
    """Parse equipment code into components"""
    try:
        # Expected format: "PUMP_001_LINE_A"
        parts = str(equipment_string).split('_')
        
        if len(parts) >= 4:
            return {
                "equipment_type": parts[0],
                "equipment_number": parts[1],
                "line_id": parts[2],
                "section": parts[3],
                "full_code": equipment_string,
                "valid": True
            }
        else:
            return {
                "error": "Invalid equipment code format",
                "expected": "TYPE_NUMBER_LINE_SECTION",
                "received": equipment_string,
                "valid": False
            }
    except Exception as e:
        return {"error": str(e), "valid": False}

def generate_summary(data_json):
    """Generate human-readable summary from JSON data"""
    import json
    try:
        data = json.loads(data_json) if isinstance(data_json, str) else data_json
        
        parts = []
        if 'production_count' in data:
            parts.append(f"Produced: {data['production_count']} units")
        if 'efficiency' in data:
            parts.append(f"Efficiency: {data['efficiency']}%")
        if 'quality_rate' in data:
            parts.append(f"Quality: {data['quality_rate']}%")
        if 'operator' in data:
            parts.append(f"Operator: {data['operator']}")
        
        return " | ".join(parts) if parts else "No data available"
    except Exception as e:
        return f"Summary error: {str(e)}"
```

### 4. Unit Conversions

```python
# Script Name: UnitConverter
def convert_units(value, from_unit, to_unit):
    """Convert between different engineering units"""
    try:
        val = float(value)
        
        # Temperature conversions
        if from_unit == "celsius" and to_unit == "fahrenheit":
            return round((val * 9/5) + 32, 2)
        elif from_unit == "fahrenheit" and to_unit == "celsius":
            return round((val - 32) * 5/9, 2)
        elif from_unit == "celsius" and to_unit == "kelvin":
            return round(val + 273.15, 2)
        
        # Pressure conversions
        elif from_unit == "psi" and to_unit == "bar":
            return round(val * 0.0689476, 4)
        elif from_unit == "bar" and to_unit == "psi":
            return round(val * 14.5038, 2)
        elif from_unit == "bar" and to_unit == "pascal":
            return round(val * 100000, 0)
        
        # Flow conversions
        elif from_unit == "gpm" and to_unit == "lpm":
            return round(val * 3.78541, 2)
        elif from_unit == "lpm" and to_unit == "gpm":
            return round(val / 3.78541, 2)
        
        # Length conversions
        elif from_unit == "mm" and to_unit == "inches":
            return round(val * 0.0393701, 3)
        elif from_unit == "inches" and to_unit == "mm":
            return round(val * 25.4, 2)
        
        else:
            return val  # No conversion needed or unknown units
    
    except (ValueError, TypeError):
        return 0
```

```lot
DEFINE ACTION UnitConversionProcessor
ON TOPIC "convert/+/request" DO
    SET "conversion_type" WITH TOPIC POSITION 1
    
    CALL PYTHON "UnitConverter.convert_units"
        WITH (PAYLOAD, GET TOPIC "convert/from_unit", GET TOPIC "convert/to_unit")
        RETURN AS {converted_value}
    
    PUBLISH TOPIC "convert/" + {conversion_type} + "/result" WITH {converted_value}
    PUBLISH TOPIC "convert/" + {conversion_type} + "/original" WITH PAYLOAD
```

### 5. JSON Processing

```python
# Script Name: JSONProcessor
def extract_nested_field(json_data, field_path):
    """Extract nested field from JSON using dot notation"""
    import json
    try:
        data = json.loads(json_data) if isinstance(json_data, str) else json_data
        
        # Split path like "sensor.readings.temperature"
        fields = field_path.split('.')
        
        result = data
        for field in fields:
            result = result[field]
        
        return result
    except (KeyError, TypeError, json.JSONDecodeError) as e:
        return None

def merge_json_objects(json1, json2):
    """Merge two JSON objects"""
    import json
    try:
        obj1 = json.loads(json1) if isinstance(json1, str) else json1
        obj2 = json.loads(json2) if isinstance(json2, str) else json2
        
        merged = {**obj1, **obj2}
        return json.dumps(merged)
    except Exception as e:
        return json.dumps({"error": str(e)})

def calculate_averages(readings_json):
    """Calculate statistics from array of readings"""
    import json
    try:
        readings = json.loads(readings_json) if isinstance(readings_json, str) else readings_json
        
        if not isinstance(readings, list) or len(readings) == 0:
            return {"error": "No readings provided"}
        
        total = sum(readings)
        count = len(readings)
        average = total / count
        
        return {
            "average": round(average, 2),
            "total": total,
            "count": count,
            "min_value": min(readings),
            "max_value": max(readings),
            "range": max(readings) - min(readings)
        }
    except Exception as e:
        return {"error": str(e)}
```

## Return Value Handling

### Simple Values
```python
def simple_calculation(a, b):
    return a + b  # Returns: 8
```

```lot
CALL PYTHON "Calculator.simple_calculation" WITH (5, 3) RETURN AS {result}
PUBLISH TOPIC "result" WITH {result}  // Publishes: 8
```

### JSON Objects
```python
def complex_result(value):
    return {
        "original": value,
        "doubled": value * 2,
        "squared": value ** 2
    }
```

```lot
CALL PYTHON "Calculator.complex_result" WITH (5) RETURN AS {result}
PUBLISH TOPIC "result" WITH {result}  // Publishes entire JSON object
PUBLISH TOPIC "result/doubled" WITH (GET JSON "doubled" IN {result} AS INT)  // Extracts field
```

## Error Handling

### Python Side
```python
def safe_division(a, b):
    """Division with error handling"""
    try:
        result = float(a) / float(b)
        return {"success": True, "result": result}
    except ZeroDivisionError:
        return {"success": False, "error": "Division by zero"}
    except (ValueError, TypeError):
        return {"success": False, "error": "Invalid input types"}
```

### LOT Side
```lot
CALL PYTHON "Calculator.safe_division" WITH (10, 2) RETURN AS {result}
SET "success" WITH (GET JSON "success" IN {result} AS BOOL)
IF {success} EQUALS TRUE THEN
    PUBLISH TOPIC "calculation/result" WITH (GET JSON "result" IN {result} AS DOUBLE)
ELSE
    PUBLISH TOPIC "calculation/error" WITH (GET JSON "error" IN {result} AS STRING)
```

## Performance Considerations

### 1. Keep Functions Fast
- Python execution adds latency
- Target <100ms for most functions
- Use caching for expensive operations

### 2. Minimize External Dependencies
- Fewer imports = faster startup
- Use built-in libraries when possible
- Consider loading large libraries globally

### 3. Batch Processing
```python
def process_batch(values_json):
    """Process multiple values in one call"""
    import json
    values = json.loads(values_json)
    return [process_single(v) for v in values]
```

### 4. Avoid I/O Operations
- No file operations in time-critical paths
- No network calls from Python functions
- Use LOT routes for I/O instead

## When to Use Python

| Use Python For | Use LOT For |
|----------------|-------------|
| Complex calculations | Simple math (+, -, *, /) |
| Statistical analysis | Basic conditionals |
| String regex/parsing | String concatenation |
| Data validation | Topic routing |
| External libraries | MQTT operations |
| JSON manipulation | Simple data flow |

## Related Documentation

- [Advanced Python Integration](./Advanced-Python.md) - External libraries, complex processing
- [Integration Patterns](./Integration-Patterns.md) - Best practices and patterns
- [Math Calculator Action](../Actions/Python-Actions/Math-Calculator-Action.md) - Example integration

[Back to Main](../README.md)

