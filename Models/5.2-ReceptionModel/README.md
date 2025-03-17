### 5.2 Reception Singleton Model Example
A Reception singleton model triggers on a specific topic. Only one instance is managed because the topic is explicit:
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

[Back to Models](../README.md)

