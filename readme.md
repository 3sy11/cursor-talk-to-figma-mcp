# Cursor Talk to Figma MCP

This project enables Cursor AI to communicate with Figma through the Model Context Protocol (MCP), allowing AI-assisted design operations.

## Overview

The Cursor Talk to Figma MCP provides a bridge between Cursor AI and Figma, enabling automated design operations, variable management, and design system interactions through natural language commands.

## Features

- **Figma Integration**: Direct communication with Figma through WebSocket
- **Variable Management**: Create, modify, and bind design variables
- **Node Operations**: Manipulate Figma nodes, frames, and components
- **Design System Support**: Full support for Figma's design system features
- **AI-Powered**: Natural language interface for design operations

## Installation

1. Clone the repository
2. Install dependencies: `npm install`
3. Build the project: `npm run build`
4. Follow the setup instructions in the plugin

## Usage

1. Start the MCP server in Cursor
2. Connect the Figma plugin to the server
3. Use natural language commands to interact with Figma

## Release History

For detailed release notes and bug fixes, see the [releases](./releases/) folder:

- [2024-12-19](./releases/2024-12-19.md) - String Variable Setting Issue Fix
- [2025-09-12](./releases/2025-09-12.md) - Color Variable Serialization Fix & Tool Refactoring
- [2025-09-15](./releases/2025-09-15.md) - Universal Node Variable Binding Implementation
- [2025-09-18](./releases/2025-09-18.md) - Specialized Padding Variable Binding Tools

## Contributing

Please read our contributing guidelines and submit pull requests for any improvements.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
