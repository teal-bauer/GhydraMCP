[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/teal-bauer/GhydraMCP)](https://github.com/teal-bauer/GhydraMCP/releases)
[![GitHub stars](https://img.shields.io/github/stars/teal-bauer/GhydraMCP)](https://github.com/teal-bauer/GhydraMCP/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/teal-bauer/GhydraMCP)](https://github.com/teal-bauer/GhydraMCP/network/members)
[![GitHub contributors](https://img.shields.io/github/contributors/teal-bauer/GhydraMCP)](https://github.com/teal-bauer/GhydraMCP/graphs/contributors)
[![Build Status](https://github.com/teal-bauer/GhydraMCP/actions/workflows/build.yml/badge.svg)](https://github.com/teal-bauer/GhydraMCP/actions/workflows/build.yml)

# GhydraMCP

GhydraMCP is a bridge between [Ghidra](https://ghidra-sre.org/) and AI assistants that enables AI-assisted reverse engineering through the [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol/mcp). 

![GhydraMCP logo](https://github.com/user-attachments/assets/86b9b2de-767c-4ed5-b082-510b8109f00f)

## Overview

GhydraMCP consists of:

1. **Ghidra Plugin**: Exposes Ghidra's powerful reverse engineering capabilities through a REST API
2. **MCP Bridge**: A Python script that translates MCP requests into API calls
3. **Multi-instance Support**: Connect multiple Ghidra instances to analyze different binaries simultaneously

This allows AI assistants like Claude to directly:
- Decompile functions and analyze binary code
- Understand program structure, function relationships, and data types
- Perform binary analysis tasks (identify cross-references, data flow, etc.)
- Make meaningful changes to the analysis (rename functions, add comments, etc.)

GhydraMCP is based on [GhidraMCP by Laurie Wired](https://github.com/LaurieWired/GhidraMCP/) with added multi-instance support and numerous enhancements.

# Features

GhydraMCP combines a Ghidra plugin with an MCP server to provide a comprehensive set of reverse engineering capabilities to AI assistants:

## Program Analysis

- **Decompilation**: Convert binary functions to readable C code
- **Static Analysis**: 
  - Cross-reference analysis (find who calls what)
  - Data flow analysis
  - Type propagation and reconstruction
- **Symbol Management**:
  - View and analyze imports and exports
  - Identify library functions and dependencies

## Interactive Reverse Engineering

- **Code Understanding**:
  - Explore function code and relationships
  - Analyze data structures and types
- **Annotation**:
  - Rename functions, variables, and data 
  - Add comments and documentation
  - Create and modify data types

## Multi-instance Support

- Run multiple Ghidra instances simultaneously
- Analyze different binaries in parallel
- Connect to specific instances using port numbers
- Auto-discovery of running Ghidra instances
- Instance metadata with project and file information

## Program Navigation

- List and search functions, classes, and namespaces
- View memory segments and layout
- Search by name, pattern, or signature

# Installation

## Prerequisites
- Install [Ghidra](https://ghidra-sre.org)
- Python3
- MCP [SDK](https://github.com/modelcontextprotocol/python-sdk)

## Ghidra
First, download the latest [release](https://github.com/teal-bauer/GhydraMCP/releases) from this repository. The "Complete" artifact contains the zipped Ghidra plugin and the Python MCP bridge. Unpack the outer archive, then, add the plugin to Ghidra:

1. Run Ghidra
2. Select `File` -> `Install Extensions`
3. Click the `+` button
4. Select the `GhydraMCP-1.1.zip` (or your chosen version) from the downloaded release
5. Restart Ghidra
6. Make sure the GhydraMCPPlugin is enabled in `File` -> `Configure` -> `Developer`

> **Note:** By default, the first CodeBrowser opened in Ghidra gets port 8192, the second gets 8193, and so on. You can check which ports are being used by looking at the Console in the Ghidra main (project) window - click the computer icon in the bottom right to "Open Console". Look for log entries like:
> ```
> (HydraMCPPlugin) Plugin loaded on port 8193
> (HydraMCPPlugin) HydraMCP HTTP server started on port 8193
> ```
>
> GhydraMCP now includes auto-discovery of running Ghidra instances, so manually registering each instance is typically not necessary. The MCP bridge will automatically discover and register instances on startup and periodically check for new ones.

Video Installation Guide:

https://github.com/user-attachments/assets/75f0c176-6da1-48dc-ad96-c182eb4648c3

## MCP Clients

Theoretically, any MCP client should work with GhydraMCP. Two examples are given below.

## API Reference

### Available Tools

**Program Analysis**:
- `list_methods`: List all functions (params: offset, limit)
- `list_classes`: List all classes/namespaces (params: offset, limit)  
- `decompile_function`: Get decompiled C code (params: name)
- `rename_function`: Rename a function (params: old_name, new_name)
- `rename_data`: Rename data at address (params: address, new_name)
- `list_segments`: View memory segments (params: offset, limit)
- `list_imports`: List imported symbols (params: offset, limit)
- `list_exports`: List exported functions (params: offset, limit)
- `list_namespaces`: Show namespaces (params: offset, limit)
- `list_data_items`: View data labels (params: offset, limit)
- `search_functions_by_name`: Find functions (params: query, offset, limit)

**Instance Management**:
- `list_instances`: List active Ghidra instances (no params)
- `register_instance`: Register new instance (params: port, url)
- `unregister_instance`: Remove instance (params: port)
- `discover_instances`: Auto-discover running instances (params: host [optional])

**Example Usage**:
```python
# Program analysis
client.use_tool("ghydra", "decompile_function", {"name": "main"})

# Instance management  
client.use_tool("ghydra", "register_instance", {"port": 8192, "url": "http://localhost:8192/"})
client.use_tool("ghydra", "register_instance", {"port": 8193})

# Auto-discover instances
client.use_tool("ghydra", "discover_instances")  # Default host
client.use_tool("ghydra", "discover_instances", {"host": "192.168.1.10"})  # Custom host
```

## Client Setup

### Claude Desktop Configuration
```json
{
  "mcpServers": {
    "ghydra": {
      "command": "uv",
      "args": [
        "run",
        "/ABSOLUTE_PATH_TO/bridge_mcp_hydra.py"
      ],
      "env": {
        "GHIDRA_HYDRA_HOST": "localhost"  // Optional - defaults to localhost
      }
    }
  }
}
```

> **Note:** You can also use `python` instead of `uv run`, but then you'll need to manually install the requirements first with `pip install mcp requests`.

### 5ire Configuration
1. Tool Key: ghydra  
2. Name: GhydraMCP
3. Command: `uv run /ABSOLUTE_PATH_TO/bridge_mcp_hydra.py`

## Example Session

Below is an example of how an AI assistant might interact with multiple Ghidra instances using GhydraMCP:

**User:**
```
Check which Ghidra instances are currently running.
```

**Assistant:**
```
I'll check which Ghidra instances are currently running by discovering any active instances.

View result from discover_instances from ghydra (local)
{
  "found": 2,
  "instances": [
    {
      "port": 8192,
      "url": "http://localhost:8192",
      "result": "Registered instance on port 8192 at http://localhost:8192"
    },
    {
      "port": 8193,
      "url": "http://localhost:8193",
      "result": "Registered instance on port 8193 at http://localhost:8193"
    }
  ]
}

View result from list_instances from ghydra (local)
{
  "instances": [
    {
      "port": 8192,
      "url": "http://localhost:8192",
      "project": "TestProject",
      "file": "application.exe"
    },
    {
      "port": 8193,
      "url": "http://localhost:8193",
      "project": "FirmwareAnalysis",
      "file": "firmware.bin"
    }
  ]
}

I found two active Ghidra instances:
1. Port 8192: Analyzing "application.exe" in project "TestProject"
2. Port 8193: Analyzing "firmware.bin" in project "FirmwareAnalysis"

I'm ready to analyze these binaries. What would you like me to examine?
```

**User:**
```
List the functions through ghydra for both instances and identify any related functionality.
```

**Assistant:**
```
I'll list the functions from both instances and analyze them.

View result from list_functions from ghydra (local){
  `port`: 8192
}
[Function list for application.exe...]

View result from list_functions from ghydra (local){
  `port`: 8193
}
[Function list for firmware.bin...]

I notice some interesting functions in both binaries. Let me decompile a few to understand their behavior.

View result from get_function from ghydra (local){
  `port`: 8192,
  `name`: "process_command"
}
[Decompiled code for process_command...]

View result from get_function from ghydra (local){
  `port`: 8193,
  `name`: "handle_message"
}
[Decompiled code for handle_message...]

Based on this analysis, I can see these binaries communicate using a simple protocol where...
```

# JSON Communication

GhydraMCP uses structured JSON for all communication between the Python bridge and Java plugin. This ensures consistent and reliable data exchange.

## Response Format

All responses follow a standard format:

```json
{
  "success": true,
  "result": "...",
  "timestamp": 1712159482123,
  "port": 8192,
  "instanceType": "base"
}
```

Error responses include additional information:

```json
{
  "success": false,
  "error": "Error message",
  "status_code": 404,
  "timestamp": 1712159482123
}
```

This structured approach makes the communication more reliable and easier to debug.

# Testing

GhydraMCP includes comprehensive test suites for both the HTTP API and MCP bridge. See [TESTING.md](TESTING.md) for details on running the tests.

## HTTP API Tests

Tests the HTTP endpoints exposed by the Java plugin:
- Response format and structure
- JSON structure consistency
- Required fields in responses
- Error handling

## MCP Bridge Tests

Tests the MCP bridge functionality:
- MCP protocol communication
- Tool availability and structure
- Response format and structure
- JSON structure consistency

# Building from Source

You can build different artifacts with Maven:

## Build Everything (Default)
Build both the Ghidra plugin and the complete package:

```
mvn clean package
```

This creates:
- `target/GhydraMCP-[version].zip` - The Ghidra plugin only
- `target/GhydraMCP-Complete-[version].zip` - Complete package with plugin and bridge script

## Build Ghidra Plugin Only
If you only need the Ghidra plugin:

```
mvn clean package -P plugin-only
```

## Build Complete Package Only
If you only need the combined package:

```
mvn clean package -P complete-only
```

The Ghidra plugin includes these files required for Ghidra to recognize the extension:
- lib/GhydraMCP.jar
- extension.properties
- Module.manifest
