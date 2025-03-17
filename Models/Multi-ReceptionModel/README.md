### 5.3 Multi-Reception Model Example
Multi-Reception models support multiple instances by using wildcards in the topic. For example:
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

[Back to Models](../README.md)

