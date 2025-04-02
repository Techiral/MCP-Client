# Transforming Middleware to Model Context Protocol (MCP)

## Current State vs. MCP Specification

The code provided represents a middleware component that facilitates connections between applications and various services like Notion and Google Drive. While it shares some conceptual similarities with MCP, it's fundamentally different in architecture, purpose, and implementation.

### Core Differences Analysis

| Feature | Current Middleware | Model Context Protocol (MCP) |
|---------|-------------------|----------------------------|
| **Primary Purpose** | Integration orchestration between applications (Notion, Google Drive) | Connecting LLMs to external tools and resources |
| **Architecture** | Centralized adapter pattern with API calls | Client-server architecture with standardized protocol |
| **Client Focus** | Backend service integration | LLM-to-tool communication |
| **Capabilities** | Actions on applications via adapters | Resources, Tools, and Prompts |
| **Communication** | HTTP/REST API calls | Standardized protocol (likely using JSON-RPC) |
| **Authentication** | User-specific tokens from database | Simpler authentication model |
| **Error Handling** | Custom error management with database logging | Standardized error responses |

## Understanding the Current Middleware

The current code implements a Multi-Channel Platform (MCP) that serves as middleware between a main application and third-party services. It's designed to:

1. Route requests to specific service adapters (Notion, Google Drive)
2. Handle authentication via user-specific tokens
3. Log integration activities
4. Provide standardized request/response formats
5. Implement error handling and retries

This differs significantly from the Model Context Protocol, which is designed specifically for LLMs to interact with external tools and data sources.

## Key Components Requiring Transformation

### 1. Fundamental Architecture

```
Current Architecture:
[Main App] <-> [Middleware/Adapters] <-> [External Services]

MCP Architecture:
[LLM Client] <-> [MCP Protocol] <-> [Tool Servers]
```

The current middleware is structured as an adapter layer with centralized routing, whereas MCP uses a standardized protocol between clients and tool servers.

### 2. Tool Definition and Execution

The current middleware uses a dynamic adapter import pattern:

```python
# Current approach
if mcp_request.app == "notion":
    from app.mcp.adapters import notion_adapter as adapter_module
elif mcp_request.app == "google_drive":
    from app.mcp.adapters import google_adapter as adapter_module
```

MCP uses a decorator-based approach for defining tools:

```python
# MCP approach
@mcp.tool()
async def get_forecast(latitude: float, longitude: float) -> str:
    """Get weather forecast for a location.
    
    Args:
        latitude: Latitude of the location
        longitude: Longitude of the location
    """
    # Implementation
```

### 3. Communication Protocol

The current middleware uses HTTP requests and custom response objects, while MCP likely uses a standardized protocol like JSON-RPC with specific message types.

### 4. Authentication Model

The current code retrieves user-specific tokens from a database:

```python
token_data = await supabase.get_integration_token(user_id, mcp_request.app)
access_token = token_data["decrypted_access_token"]
```

MCP likely has a simpler authentication model focused on tool servers rather than individual service integrations.

## Transformation Roadmap

### Phase 1: Foundation Restructuring

1. **Adopt the FastMCP Framework**
   - Replace the current adapter pattern with the FastMCP server framework
   - Change from user-oriented to tool-oriented architecture

2. **Redefine Core Models**
   - Replace `McpRequest` and `McpResponse` with MCP protocol message types
   - Implement proper MCP message serialization/deserialization

### Phase 2: Tool Implementation

1. **Convert Adapters to Tools**
   - Transform each adapter function into a proper MCP tool using decorators
   - Add proper documentation through docstrings
   - Example transformation:

   ```python
   # From current adapter-based approach
   async def handle_action(action, parameters, token):
       if action == "create_page":
           return await create_page(parameters, token)
   
   # To MCP tool approach
   @mcp.tool()
   async def create_notion_page(title: str, content: str) -> str:
       """Create a new page in Notion.
       
       Args:
           title: Title of the page
           content: Content in markdown format
       """
       # Implementation
   ```

2. **Implement Resource Access**
   - Add resource capabilities that are missing in current implementation
   - Implement proper resource fetching and serving

### Phase 3: Protocol and Transport

1. **Implement Standard Transport**
   - Replace HTTP API with proper MCP transport (stdio or websockets)
   - Add `mcp.run(transport='stdio')` entry point

2. **Standardize Error Handling**
   - Replace custom error handling with MCP standard error formats
   - Remove database logging dependencies

## Implementation Guide

### 1. Core Server Setup

```python
from mcp.server.fastmcp import FastMCP

# Initialize FastMCP server with a name
mcp = FastMCP("integrations")
```

### 2. Converting Notion Adapter to MCP Tools

```python
@mcp.tool()
async def notion_create_page(title: str, content: str, parent_id: str = None) -> dict:
    """Create a new page in Notion.
    
    Args:
        title: Title of the new page
        content: Content in markdown format
        parent_id: Optional parent page ID
    """
    # Implementation using Notion API
    # Return results in a structured format
```

### 3. Converting Google Drive Adapter to MCP Tools

```python
@mcp.tool()
async def google_drive_upload_file(file_content: str, file_name: str, 
                                  folder_id: str = None) -> dict:
    """Upload a file to Google Drive.
    
    Args:
        file_content: Content of the file
        file_name: Name of the file
        folder_id: Optional folder ID to upload to
    """
    # Implementation using Google Drive API
    # Return results in a structured format
```

### 4. Server Execution

```python
if __name__ == "__main__":
    # Initialize and run the server
    mcp.run(transport='stdio')
```

## Authentication Considerations

MCP tools may need to handle authentication differently than the current middleware:

1. **Tool-Level Authentication**
   - Each tool can handle its own authentication
   - May use environment variables or configuration files

2. **Client-Provided Credentials**
   - Accept credentials as parameters to tools when needed
   - Implement secure credential handling

## Integration with Claude for Desktop

To integrate the new MCP server with Claude for Desktop:

1. Configure `claude_desktop_config.json`:
```json
{
    "mcpServers": {
        "integrations": {
            "command": "python",
            "args": [
                "/path/to/your/mcp_server.py"
            ]
        }
    }
}
```

2. Restart Claude for Desktop to pick up the new MCP server
3. Use the tools through the Claude interface

## Benefits of the Transformation

1. **Direct LLM Integration**: Enables Claude and other LLMs to directly use your tools
2. **Standardized Interface**: Consistent tool definition and execution model
3. **Simpler Architecture**: Removes complex middleware layers
4. **Better Documentation**: Automatic tool documentation from docstrings
5. **Broader Compatibility**: Works with any MCP client, not just your custom application

## Challenges and Considerations

1. **Authentication**: Moving from user-specific tokens to a more general model
2. **Error Handling**: Adapting to MCP's standardized error format
3. **State Management**: Handling stateful operations within stateless tool calls
4. **Resource Access**: Implementing proper resource capabilities

## Conclusion

Transforming the current middleware into a proper MCP implementation requires significant architectural changes, but offers substantial benefits in terms of LLM integration capabilities. By following the transformation roadmap and implementing the changes outlined above, you can create a powerful set of tools that Claude and other LLMs can leverage directly.
