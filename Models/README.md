
# KEYWORD: `DEFINE MODEL`

## 1. Overview
- **Description:**  
  A brief summary of what the `DEFINE MODEL` keyword does, including its purpose to create a data processing pipeline within the LOT language. Models are used to transform, aggregate, and compute values from input MQTT topics, publishing the results to new topics. They serve as recipes to generate model instances, which can either be static (singleton) or dynamic (supporting multiple instances based on topic wildcards).

## 2. Signature
- **Syntax:**  
  ```lot
  DEFINE MODEL <model_name> WITH TOPIC "<output_base_topic>"
      ADD "<property_name>" WITH <source> [AS TRIGGER]
      ...
  ```
  *Example:*  
  ```lot
  DEFINE MODEL LampEnergyCost WITH TOPIC "Coreflux/+/+/+/+/energy"
      ADD "total_energy" WITH TOPIC "shellies/+/+/+/+/device/energy" AS TRIGGER
      ADD "energy_price" WITH 3
      ADD "cost" WITH (total_energy * energy_price)
  ```

## 3. Able Keywords to Interact
List and describe each keyword that can be used in conjunction with `DEFINE MODEL`:
- **`ADD`** (*Keyword*):  
  Used to add properties to the model. A property can reference a topic, be assigned a constant, or be defined as an expression.
- **`WITH TOPIC`** (*Parameter*):  
  Specifies the base output topic for the model or defines the source topic for a property.
- **`AS TRIGGER`** (*Modifier*):  
  Indicates that changes in the input topic should trigger the model's computation.
- **`TIMESTAMP`** (*Keyword*):  
  Used within a property definition to include a timestamp in various formats (e.g., `"UTC"`, `"ISO"`, `"UNIX-MS"`, or `"UNIX"`).
- **`IF ... THEN ... ELSE`** (*Expression*):  
  Provides conditional logic for computing property values.

## 4. Return Value
- **Description:**  
  Models do not return a traditional value. Instead, they process input data and publish computed outputs to MQTT topics defined by the model's output namespace. The output is a set of derived topics containing transformed or aggregated data.

## 5. Usage Examples

### 5.1 Static Model Example
This example shows a model that processes energy data for a lamp:
```lot
DEFINE MODEL LampEnergyCost WITH TOPIC "Coreflux/+/+/+/+/energy"
    ADD "total_energy" WITH TOPIC "shellies/+/+/+/+/device/energy" AS TRIGGER
    ADD "energy_price" WITH 3
    ADD "cost" WITH (total_energy * energy_price)
```

### 5.2 Dynamic Singleton Model Example
A dynamic singleton model triggers on a specific topic. Only one instance is managed because the topic is explicit:
```lot
DEFINE MODEL MaterialContentData1 WITH TOPIC "Process/Machine/1/Material/MaterialContent_New"
```
If a JSON payload arrives at `Process/Machine/1/Material/MaterialContent_New`:
```json
{
   "MaterialID": 132323,
   "Name": "Peppers",
   "Quantity": 21
}
```
It will explode into:
- `Process/Machine/1/Material/MaterialContent_New/MaterialID` with value `132323`
- `Process/Machine/1/Material/MaterialContent_New/Name` with value `Peppers`
- `Process/Machine/1/Material/MaterialContent_New/Quantity` with value `21`

### 5.3 Dynamic Model Example
Dynamic models support multiple instances by using wildcards in the topic. For example:
```lot
DEFINE MODEL MaterialContentData WITH TOPIC "Process/+/+/Material/MaterialContent_New"
```
For a JSON payload arriving at `Process/Machine/33/Material/MaterialContent_New`:
```json
{
   "MaterialID": 132323,
   "Name": "Peppers",
   "Quantity": 21
}
```
The model will create the following topics:
- `Process/Machine/33/Material/MaterialContent_New/MaterialID` with value `132323`
- `Process/Machine/33/Material/MaterialContent_New/Name` with value `Peppers`
- `Process/Machine/33/Material/MaterialContent_New/Quantity` with value `21`

## 6. Notes & Additional Information
- **Dynamic Models:**  
  Dynamic models expect a JSON payload on the specified topic. They come in two forms:
  - **Dynamic Singleton Models:** Trigger only on a specific topic (e.g., `Process/Machine/1/Material/MaterialContent_New`), ensuring a single model instance.
  - **Dynamic Models:** Use wildcards (e.g., `Process/+/+/Material/MaterialContent_New`) to support multiple instances—ideal for scenarios with many similar devices.
- **Model Instance Identification:**  
  Each model instance is uniquely identified based on the topic structure and the time of creation, allowing LOT to manage even constant properties as distinct units over time.
- **Related Keywords:**  
  See also the documentation for `DEFINE RULE` and `ADD` for additional context on how models integrate with other LOT components.
