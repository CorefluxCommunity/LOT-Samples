# LOT Samples

![LOT Language of Things](https://cdn.prod.website-files.com/67444bdb9c312e239038aa41/678a888460016fcf4a0f13b7_Group%209133.webp)

## Project Overview
The **LOT Samples** repository is a collection of demonstrations, tutorials, and sample code for the LOT language. LOT (Language of Things) is a human-readable, domain-specific language for IoT systems that uses near-English syntax to define rules, models, actions, and more. This project is designed for new users of LOT, providing an easy way to learn each LOT keyword through practical examples. By exploring the examples in this repo, users can quickly understand how to use LOT keywords in real-world IoT scenarios.

For more details on LOT, visit the [LOT Language of Things page](https://www.coreflux.org/lot-language-of-things).


## What is Languague of Things
- [Actions](../Actions/DEFINE ACTION.md)
- [Models](../Models/DEFINE MODEL.md)
- [Routes](../Routes/README.md)
- [Syntax](../Syntax/README.md)

## Installation & Dependencies
Setting up the LOT Samples project is straightforward. The only dependencies needed are:
- **Visual Studio Code** with the **LOT Notebook extension** (LOT Language Support by Coreflux) for syntax highlighting and running LOT code.
- The **Coreflux MQTT broker**, which acts as the backend to execute LOT rules and models.

**Installation Steps:**
1. Install **Visual Studio Code** if you haven't already.
2. In VS Code, install the *LOT Language Support* extension (search for "LOT Language Support by Coreflux" in the Extensions marketplace).
3. Obtain the **Coreflux MQTT Broker** from the [Coreflux website](https://coreflux.org) and follow its setup instructions (or use any Coreflux-compatible MQTT broker instance).
4. Clone this repository or download the ZIP of **LOT Samples**, and open the folder in VS Code.

## How to Use
Once you have the dependencies set up and the repository open in VS Code, you can start exploring and running the samples:

1. **Browse the Samples:**  
   The repository is organized by LOT keywords or features. Each file demonstrates a specific keyword (e.g., examples for `DEFINE`, `MODEL`, `RULE`, `ACTION`, `ROUTE`, `GET`, `PUBLISH`, etc.). Open any sample file to view the LOT code along with inline comments explaining its functionality.

2. **Run a Sample:**  
   To execute a LOT sample:
   - Ensure your Coreflux MQTT broker is running.
   - Open the sample file in VS Code and use the LOT extension to run it:
     - Right-click within the editor and select **"Upload Model or Rule to MQTT Broker"** (for files containing `DEFINE MODEL` or `DEFINE RULE` blocks).
     - Enter your broker URL when prompted (for a local broker, you might use `mqtt://127.0.0.1:1883`), along with any necessary credentials.
   - The extension will deploy the LOT rule/model to the broker. Test the deployed sample by publishing messages to the relevant topics or checking the broker’s logs/output.

3. **Experiment:**  
   Modify the samples or combine different LOT keywords to experiment with the language. The samples are designed as a starting point—adjust parameters and conditions to see how the behavior changes. LOT's readability allows you to quickly iterate and understand the effects of your changes.

## Resources
For more in-depth information and further learning, refer to the following resources:
- **Official LOT Documentation:**  
  Find comprehensive guides and references at the [Coreflux LOT Docs](https://docs.coreflux.org/LOT/).
- **Coreflux MQTT Broker:**  
  Learn more about the Coreflux IoT platform and access the MQTT broker at the [Coreflux website](https://coreflux.org).
- **LOT Language of Things:**  
  Explore additional information and updates on LOT at the [LOT Language of Things page](https://www.coreflux.org/lot-language-of-things).

## Contribution
Contributions to **LOT Samples** are welcome! If you have additional examples, improvements, or bug fixes, please feel free to contribute:
- **Fork the repository** on GitHub and create a new branch for your changes.
- **Add or update samples:** Follow the existing format and naming conventions when adding new examples. Provide clear comments or documentation notes with your sample.
- **Submit a Pull Request:** Open a pull request with a detailed description of your changes. Maintainers will review and merge contributions that align with the project’s guidelines.
- Open issues to report problems or suggest new sample ideas. Your feedback helps improve the repository for everyone.

## License
This project is licensed under the **Apache 2.0 License**. By using or contributing to this repository, you agree to the terms and conditions of the license. Please see the [LICENSE](LICENSE) file for details.

