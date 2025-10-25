# LOT Samples

![LOT Language of Things](https://cdn.prod.website-files.com/67444bdb9c312e239038aa41/678a888460016fcf4a0f13b7_Group%209133.webp)

## Project Overview

The **LOT Samples** repository is a comprehensive collection of demonstrations, tutorials, and sample code for the LOT language. LOT (Language of Things) is a human-readable, domain-specific language for IoT systems that uses near-English syntax to define rules, models, actions, and routes. This project is designed for both new and experienced users of LOT, providing practical examples and patterns for real-world IoT scenarios.

**Version: 1.0.0** | [View Changelog](CHANGELOG.md)

For more details on LOT, visit the [LOT Language of Things page](https://www.coreflux.org/lot-language-of-things).

## What is LOT (Language of Things)?

LOT is the SQL of real-time MQTT data. Just as SQL transforms and queries static database records, LOT transforms and processes live MQTT streams in real-time. It eliminates the need for complex middleware by embedding business logic directly into the MQTT broker.

### Core Components

- **[Actions](Actions/DEFINE%20ACTION.md)** - Event-driven logic triggered by time or topics
- **[Models](Models/DEFINE%20MODEL.md)** - Data structures that format and validate MQTT data
- **[Routes](Routes/DEFINE%20ROUTE.md)** - System integrations (bridges, databases, pipelines)
- **[Syntax](Syntax/LOT%20Syntax.md)** - Language keywords and functional operators
- **[Python Integration](Python/)** - Extend LOT with Python functions
- **[Examples](Examples/)** - Complete system implementations

## Quick Navigation

### 📚 Learn by Category

#### Actions - Event-Driven Logic

Execute logic in response to events:

- **[Topic-Based Actions](Actions/Topic-Based-Actions/)** - React to MQTT messages
  - [Simple Echo](Actions/Topic-Based-Actions/Simple-Echo-Action.md) - Basic message forwarding
  - [Sensor Data Router](Actions/Topic-Based-Actions/Sensor-Data-Router.md) - Wildcard routing
  - [Alarm Processor](Actions/Topic-Based-Actions/Alarm-Processor.md) - Multi-level processing

- **[Time-Based Actions](Actions/Time-Based-Actions/)** - Execute on schedules
  - [Simple Heartbeat](Actions/Time-Based-Actions/Simple-Heartbeat.md) - Periodic signals
  - [Counter Action](Actions/Time-Based-Actions/Counter-Action.md) - State management
  - [Production Simulator](Actions/Time-Based-Actions/Production-Simulator.md) - Realistic metrics

- **[Python Actions](Actions/Python-Actions/)** - Python-integrated logic
  - [Math Calculator](Actions/Python-Actions/Math-Calculator-Action.md) - Complex calculations

[View all Actions documentation →](Actions/DEFINE%20ACTION.md)

#### Models - Data Structures

Define and format MQTT data:

- **[Basic Models](Models/Basic-Models/)** - Core data schema pattern
  - [DEFINE MODEL Pattern](Models/Basic-Models/DEFINE-MODEL-Pattern.md) - Field types, triggers, calculations
  - JSON reception and explosion
  - Singleton and multi-instance patterns

- **[Action Models](Models/Action-Models/)** - Dynamic publishing
  - [PUBLISH MODEL Pattern](Models/Action-Models/PUBLISH-MODEL-Pattern.md) - Event-driven records
  - GET JSON extraction with type casting
  - Runtime data creation

- **[Model Inheritance](Models/Model-Inheritance/)** - Code reuse
  - [FROM Keyword Pattern](Models/Model-Inheritance/FROM-Keyword-Pattern.md) - Model families
  - Base and specialized models
  - Polymorphic data handling

[View all Models documentation →](Models/DEFINE%20MODEL.md)

#### Routes - System Integration

Connect LOT to external systems:

- **[DataPipeline Routes](Routes/DataPipeline/)**
  - [MQTT Bridge](Routes/DataPipeline/MQTT_BRIDGE.md) - Broker-to-broker connectivity
  - [OpenSearch](Routes/DataPipeline/OPENSEARCH.md) - Database storage
  - [Email](Routes/DataPipeline/EMAIL.md) - Email notifications
  - [File Storage](Routes/DataPipeline/FILE_STORAGE.md) - File operations

[View all Routes documentation →](Routes/DEFINE%20ROUTE.md)

#### Python Integration

Extend LOT with Python:

- **[Simple Python](Python/Simple-Python.md)** - Core integration patterns
  - CALL PYTHON syntax
  - Math calculations and statistics
  - Data validation
  - Text processing and unit conversions
  - JSON manipulation
  - Error handling strategies

[View Python documentation →](Python/Simple-Python.md)

### 🎯 Learn by Example

#### Complete System Examples

Real-world implementations combining multiple components:

- **[Sensor Data Processing System](Examples/Sensor-Data-Processing/Complete-Sensor-System.md)**
  - Topic actions for real-time processing
  - Python validation and analysis
  - Action models for structured records
  - MQTT bridge to cloud
  - OpenSearch database storage
  - Time-based monitoring and statistics

[View all Examples →](Examples/)

### 📖 Learn by Syntax

Explore language keywords and operators:

- **[Entities](Syntax/Entities/)** - Core LOT entities
  - [PAYLOAD](Syntax/Entities/PAYLOAD/) - Message data access
  - [TOPIC](Syntax/Entities/TOPIC/) - Topic structure operations
  - [TIMESTAMP](Syntax/Entities/TIMESTAMP/) - Time handling
  - [ON EVERY](Syntax/Entities/ON/) - Time-based triggers

- **[Functional Keywords](Syntax/Functional/)** - Data operations
  - [SET](Syntax/Functional/SET/) - Variable assignment
  - [GET TOPIC](Syntax/Functional/GET%20TOPIC/) - Topic data retrieval
  - [PUBLISH TOPIC](Syntax/Functional/PUBLISH%20TOPIC/) - Data publishing
  - [REPLACE](Syntax/Functional/REPLACE/) - Text replacement
  - [FILTER](Syntax/Functional/FILTER/) - Data filtering
  - [TRIM](Syntax/Functional/TRIM/) - String trimming

[View complete syntax guide →](Syntax/LOT%20Syntax.md)

## Installation & Dependencies

### Required Software

1. **Visual Studio Code** - Primary development environment
2. **LOT Notebook Extension** - LOT Language Support by Coreflux
   - Provides syntax highlighting
   - Enables LOT code execution
   - Notebook format support (.lotnb files)
3. **Coreflux MQTT Broker** - Runtime execution environment
   - Executes LOT rules and models
   - Provides MQTT infrastructure
   - Available from [coreflux.org](https://coreflux.org)

### Installation Steps

1. **Install Visual Studio Code**
   ```
   Download from: https://code.visualstudio.com/
   ```

2. **Install LOT Language Support Extension**
   - Open VS Code
   - Go to Extensions (Ctrl+Shift+X / Cmd+Shift+X)
   - Search for "LOT Language Support by Coreflux"
   - Click Install

3. **Set Up Coreflux MQTT Broker**
   - Download from [Coreflux website](https://coreflux.org)
   - Follow broker setup instructions
   - Start broker service

4. **Clone or Download LOT Samples**
   ```bash
   git clone https://github.com/CorefluxCommunity/LOT-Samples.git
   cd LOT-Samples
   ```

5. **Open in VS Code**
   ```bash
   code .
   ```

## How to Use

### Browsing Documentation

The repository is organized by LOT concepts and patterns:

```
LOT-Samples/
├── Actions/           # Event-driven logic
├── Models/            # Data structures
├── Routes/            # System integrations
├── Syntax/            # Language reference
├── Python/            # Python integration
└── Examples/          # Complete systems
```

Each folder contains:
- **Main documentation** (DEFINE [CONCEPT].md)
- **Pattern guides** (detailed explanations)
- **Examples** (working code samples)

### Running Examples

#### 1. Explore a Sample
Open any `.md` file or `.lotnb` file to view LOT code with inline comments.

#### 2. Execute LOT Code
To run a LOT sample:

1. Ensure your Coreflux MQTT broker is running
2. Open the sample file in VS Code
3. Right-click in the editor
4. Select **"Upload Model or Rule to MQTT Broker"**
5. Enter broker URL (e.g., `mqtt://127.0.0.1:1883`)
6. Provide credentials if required

#### 3. Test the Sample
After deployment:
- Use MQTT client to publish test messages
- Subscribe to output topics
- Monitor broker logs
- Verify expected behavior

### Example: Running Simple Heartbeat

```lot
DEFINE ACTION SimpleHeartbeat
ON EVERY 5 SECONDS DO
    PUBLISH TOPIC "system/heartbeat" WITH "alive"
```

1. Deploy this action to your broker
2. Subscribe to topic: `system/heartbeat`
3. Observe message "alive" every 5 seconds

## Documentation Structure

### Metadata Format

All documentation files include standardized metadata:

```markdown
---
title: Document Title
category: Action|Model|Route|Syntax|Python|Example
version: 1.0.0
last_updated: 2025-10-25
description: One-line summary
related_tutorials: Tutorial links
---
```

### Content Organization

Each documentation file follows a consistent structure:

1. **Overview** - What it does and why
2. **Pattern/Syntax** - How to use it
3. **Examples** - Working code
4. **Use Cases** - Real-world applications
5. **Variations** - Alternative approaches
6. **Related** - Links to related docs

## Learning Path

### For Beginners

1. Start with [Actions](Actions/DEFINE%20ACTION.md)
   - Begin with [Simple Heartbeat](Actions/Time-Based-Actions/Simple-Heartbeat.md)
   - Progress to [Simple Echo](Actions/Topic-Based-Actions/Simple-Echo-Action.md)

2. Learn [Models](Models/DEFINE%20MODEL.md)
   - Understand [DEFINE MODEL Pattern](Models/Basic-Models/DEFINE-MODEL-Pattern.md)
   - Explore JSON reception behavior

3. Explore [Routes](Routes/DEFINE%20ROUTE.md)
   - Start with [MQTT Bridge](Routes/DataPipeline/MQTT_BRIDGE.md)

4. Try [Complete Example](Examples/Sensor-Data-Processing/Complete-Sensor-System.md)

### For Intermediate Users

1. Master [Action Models](Models/Action-Models/)
2. Learn [Model Inheritance](Models/Model-Inheritance/)
3. Integrate [Python](Python/Simple-Python.md)
4. Build complete systems from [Examples](Examples/)

### For Advanced Users

1. Optimize performance patterns
2. Design scalable architectures
3. Create custom Python modules
4. Contribute new examples

## Resources

### Official Documentation
- **LOT Documentation**: [docs.coreflux.org/LOT/](https://docs.coreflux.org/LOT/)
- **Coreflux Platform**: [coreflux.org](https://coreflux.org)
- **LOT Language**: [coreflux.org/lot-language-of-things](https://www.coreflux.org/lot-language-of-things)

### Community
- **Discord Server**: [discord.com/invite/A3pPrptNMm](https://discord.com/invite/A3pPrptNMm)
- **GitHub Community**: [github.com/CorefluxCommunity](https://github.com/CorefluxCommunity)

### Related Projects
- **Tutorial Materials**: See `/Tutorial` folder for comprehensive training
- **Example Notebooks**: Practical LOT notebook examples

## Contribution

Contributions to **LOT Samples** are welcome!

### How to Contribute

1. **Fork the repository** on GitHub
2. **Create a feature branch** for your changes
3. **Follow the documentation standards**:
   - Include metadata front-matter
   - Use consistent formatting
   - Provide working examples
   - Add inline comments
4. **Test your examples** with Coreflux broker
5. **Submit a Pull Request** with detailed description

### Contribution Guidelines

- **New Examples**: Follow existing patterns and naming conventions
- **Documentation**: Match the established format and style
- **Code Quality**: Test all LOT code before submitting
- **Comments**: Explain logic and provide context
- **Metadata**: Include version and category information

### Reporting Issues

- Open issues for bugs, improvements, or suggestions
- Provide clear descriptions and reproduction steps
- Include LOT code snippets when relevant
- Suggest documentation improvements

## Version Information

**Current Version**: 1.0.0 (2025-10-25)

This release represents a major documentation refactoring:
- Reorganized structure based on Tutorial materials
- Improved naming conventions
- Added Python integration documentation
- Created comprehensive examples
- Standardized metadata across all files

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

## License

This project is licensed under the **Apache 2.0 License**. 

By using or contributing to this repository, you agree to the terms and conditions of the license. See the [LICENSE](LICENSE) file for complete details.

## Acknowledgments

This documentation project aligns with and extends the comprehensive Tutorial materials developed by the Coreflux team. Special thanks to all contributors and the Coreflux community for their valuable feedback and examples.

---

**Ready to start?** Begin with [Simple Heartbeat](Actions/Time-Based-Actions/Simple-Heartbeat.md) or explore the [Complete Sensor System Example](Examples/Sensor-Data-Processing/Complete-Sensor-System.md).

**Questions?** Join the [Discord Community](https://discord.com/invite/A3pPrptNMm) or visit [Coreflux Documentation](https://docs.coreflux.org).
