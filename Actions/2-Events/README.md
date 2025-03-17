### 2. Event Triggers

An Action is activated by specifying the condition or event that should trigger its execution. Some common triggers are:

2.1 - Time Actions ON EVERY <time> – runs on a repeating interval.
Example: ON EVERY 5 SECONDS DO executes the block every 5 seconds.
Other intervals can be MINUTES, HOURS, etc.

2.2 - Topic Actions ON TOPIC <topic> – runs whenever the specified topic is published to.

Example: ON TOPIC "myTopic" DO executes whenever a message arrives on "myTopic".

2.3 - Callable Actions
Without any of this is just a Callable Action that can be used within other Actions or called by Internal Routes or other systems like MCP Server. 

[Back to Actions](../README.md)

