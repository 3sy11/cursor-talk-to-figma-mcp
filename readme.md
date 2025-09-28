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
- [2025-09-28_17-09-08](./releases/2025-09-28_17-09-08.md) - Text Node Layout Sizing Tools Implementation

### Release Log Guidelines

When creating new release logs, follow these guidelines:

1. **File Naming**: Use format `YYYY-MM-DD_HH-MM-SS.md` (e.g., `2025-09-28_17-09-08.md`)
2. **Header Format**: Use `# Feature Enhancement #XXX:` or `# Bug Fix #XXX:` pattern
3. **Required Sections**:
   - **Date/Time**: Current local time in CST
   - **Type**: Major Feature Enhancement / Bug Fix / Minor Update
   - **Status**: ✅ Completed / 🔄 In Progress / ❌ Failed
   - **Feature Description**: Brief overview of the change
   - **Trigger Requirement**: What prompted this change
   - **Technical Implementation**: Detailed code changes
   - **Files Modified**: List of changed files
   - **Testing & Validation**: What was tested
   - **Impact**: Summary of benefits and improvements

4. **Content Structure**:
   - Use clear headings with `###` and `####`
   - Include code examples with proper syntax highlighting
   - Provide usage examples and parameter structures
   - Document any breaking changes or limitations
   - Include error handling and validation details

5. **Update Process**:
   - Create new log file in `releases/` folder
   - Update this README with the new release link
   - Use descriptive link text that summarizes the change

## Contributing

Please read our contributing guidelines and submit pull requests for any improvements.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
