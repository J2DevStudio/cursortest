# Understanding MCP Servers in Cursor

This guide explains how Cursor configures and manages Model Context Protocol (MCP) servers, with a specific focus on our current configuration and the browser-tools-mcp server setup.

## What is MCP?

The Model Context Protocol (MCP) is a standardized way for AI applications to interact with external tools and services. In Cursor, MCP servers provide additional capabilities to the AI assistant, such as:

- Accessing external APIs
- Interacting with your browser
- Managing GitHub repositories
- And more...

## MCP Configuration in Cursor

### Location and Purpose

The MCP configuration file is located at:

```
~/.cursor/mcp.json
```

This file defines which MCP servers Cursor should connect to and how to start them. It's crucial for extending Cursor's AI capabilities with additional tools and services.

### Current Configuration

Our current `mcp.json` contains two configured servers:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@smithery/cli@latest",
        "run",
        "@smithery-ai/github",
        "--client",
        "cursor",
        "--config",
        "{\"githubPersonalAccessToken\":\"your-token-here\"}"
      ]
    },
    "mcp-browser-tools": {
      "command": "npx",
      "args": [
        "-y",
        "@agentdeskai/browser-tools-mcp@1.2.0",
        "--client",
        "cursor"
      ]
    }
  }
}
```

## Browser Tools MCP Setup

The browser-tools-mcp server requires two components to function properly:

1. **MCP Server** (configured in mcp.json)

   - Handles communication with Cursor
   - Provides the interface for AI commands
   - Started automatically by Cursor

2. **Browser Tools Server** (must be run separately)
   - Interfaces with Chrome/Chromium
   - Handles browser events and data collection
   - Must be started manually

### Starting the Browser Tools Server

To get the browser tools working:

1. Install and start the Browser Tools Server:

   ```bash
   npx @agentdeskai/browser-tools-server
   ```

2. Install the Chrome extension from:
   [BrowserTools Chrome Extension](https://github.com/AgentDeskAI/browser-tools-mcp/releases/download/v1.2.0/BrowserTools-1.2.0-extension.zip)

3. Configure the extension:
   - Open Chrome DevTools
   - Go to the BrowserTools panel
   - Verify connection status shows "Connected"

### Verification

You can verify the setup is working by:

1. Checking the green dot in Cursor settings for the MCP server
2. Confirming the Browser Tools Server is running in your terminal
3. Verifying the Chrome extension shows "Connected"
4. Testing a browser tool command in Cursor

## To-Do: Automating Browser Tools Server

To automate the Browser Tools Server startup, consider these approaches:

1. **System Service**

   ```bash
   # Example systemd service file
   [Unit]
   Description=Browser Tools Server
   After=network.target

   [Service]
   ExecStart=npx @agentdeskai/browser-tools-server
   Restart=always
   User=your-username

   [Install]
   WantedBy=multi-user.target
   ```

2. **Startup Script**

   ```bash
   #!/bin/bash
   # browser-tools-startup.sh

   # Check if server is already running
   if ! pgrep -f "browser-tools-server" > /dev/null; then
       npx @agentdeskai/browser-tools-server &
       echo "Browser Tools Server started"
   else
       echo "Browser Tools Server is already running"
   fi
   ```

3. **Integration with Cursor**

   - Create a wrapper script that starts both Cursor and the Browser Tools Server
   - Add the script to your system's startup applications

4. **Process Manager**
   ```bash
   # Using PM2
   npm install -g pm2
   pm2 start "npx @agentdeskai/browser-tools-server" --name "browser-tools"
   pm2 save
   pm2 startup
   ```

Choose the method that best fits your workflow and operating system.

## Troubleshooting

If you encounter issues:

1. Verify both servers are running:

   ```bash
   ps aux | grep "browser-tools"
   ```

2. Check the Browser Tools Server port (default: 3025):

   ```bash
   lsof -i :3025
   ```

3. Review Chrome extension connection settings:

   - Host should be "localhost" or "127.0.0.1"
   - Port should match the Browser Tools Server (default: 3025)

4. Clear and restart:
   - Stop both servers
   - Clear Chrome extension data
   - Restart in order: Browser Tools Server → Chrome Extension → Cursor
