# mcp-server

Quick Start
Installation
# Clone the repository
git clone https://github.com/nexus/mcp-for-nexus
cd mcp-for-nexus

# Install dependencies
npm install

# Configure environment
cp .env.example .env

# Start the server
npm run dev
Testing with the Client
Once your server is running, you can connect to it using the included test client:

# Using the test client (requires Redis for SSE transport)
node scripts/test-nexus-client.mjs http://localhost:3000
For local development, make sure Redis is installed and running:

# Install Redis (macOS)
brew install redis

# Start Redis
brew services start redis

# Verify Redis is running
redis-cli ping  # Should respond with "PONG"
Client Usage Example
// Example of calling blockchain tools from your application
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { SSEClientTransport } from "@modelcontextprotocol/sdk/client/sse.js";

const transport = new SSEClientTransport(new URL(`http://localhost:3000/sse`));

const client = new Client(
  { name: "my-app", version: "1.0.0" },
  { capabilities: { prompts: {}, resources: {}, tools: {} } }
);

await client.connect(transport);

// Get blockchain info
const networkInfo = await client.callTool({
  name: "getNetworkInfo",
  arguments: {}
});

// Query an account balance
const balance = await client.callTool({
  name: "getBalance",
  arguments: { address: "0x1234567890abcdef1234567890abcdef12345678" }
});

// Get block information
const blockData = await client.callTool({
  name: "getBlock", 
  arguments: {
    blockNumber: "latest",
    fullTransactions: true
  }
});
Integrating with AI Systems
Claude Integration Example
// Example of integrating with Claude
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { SSEClientTransport } from "@modelcontextprotocol/sdk/client/sse.js";
import Anthropic from "@anthropic-ai/sdk";

// Setup MCP client
const mcpClient = new Client(...);
await mcpClient.connect(transport);

// Get available tools
const tools = await mcpClient.listTools();
const toolDefinitions = tools.map(tool => ({
  name: tool.name,
  description: tool.description,
  input_schema: tool.inputSchema
}));

// Setup Claude client
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Create message with tools
const message = await anthropic.messages.create({
  model: "claude-3-5-sonnet-20241022",
  max_tokens: 1000,
  messages: [{ role: "user", content: "What's the latest block on Nexus?" }],
  tools: toolDefinitions
});

// Handle tool calls from Claude
for (const content of message.content) {
  if (content.type === 'tool_use') {
    const result = await mcpClient.callTool({
      name: content.name,
      arguments: content.input
    });
    
    // Send tool result back to Claude
    // ...
  }
}
Deployment Options
Local Development
The server can be run locally for development purposes, which is ideal for AI developers testing integration with the Nexus blockchain.

Production Deployment on Vercel
For production usage, the server can be deployed on Vercel:

Configure Redis (required for SSE transport):

Add Redis integration from the Vercel dashboard, or
Set REDIS_URL environment variable to your managed Redis instance
Enable Fluid Compute for longer-running operations
