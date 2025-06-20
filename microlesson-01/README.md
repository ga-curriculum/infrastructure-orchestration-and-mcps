<h1>
  <span class="headline">Infrastructure, Orchestration, and MCPs</span>
  <span class="subhead">MCP Environment Setup</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to set up an MCP environment in Google Colab and configure the necessary libraries for Gemini integration.

## Part 2: The Colab-Friendly MCP Environment

Let's set up our environment. We'll run each of the following cells one by one.

### Cell 1: Install Dependencies

This cell installs all the required Python packages with specific versions for stability. We're using `google-genai` for the Gemini model and `fastmcp` to easily create our tool server.

```python
# Run this cell to install all necessary libraries
print("--- Installing libraries... ---")
!pip install -q google-genai==0.5.4 mcp==1.9.4 fastmcp==2.8.1 httpx==0.28.1 nest_asyncio==1.6.0
print("Libraries installed successfully.")
```

### Understanding the Dependencies

Let's break down what each library does:

- **`google-genai`**: Official Google library for accessing Gemini models
- **`mcp`**: Core Model Context Protocol library for client-server communication
- **`fastmcp`**: Simplified library for creating MCP servers and tools
- **`httpx`**: Modern HTTP client for making API requests (used by our weather tool)
- **`nest_asyncio`**: Allows asyncio to work properly in Jupyter/Colab environments

### Cell 2: Set Up API Key and Imports

Next, we need to configure our Google Gemini API key. For security, we'll load it from Colab's secrets manager. Then, we'll import all the libraries we'll need for this session.

```python
import os
import getpass
import asyncio
import nest_asyncio
import httpx

from google.colab import userdata
from google import genai
from google.genai import types
from fastmcp import FastMCP
from fastmcp import Client as MCPClient
from fastmcp.client.transports import FastMCPTransport

# This allows asyncio to work correctly in a Colab/Jupyter environment
nest_asyncio.apply()

# --- API Key Setup ---
try:
    # Recommended: Use Colab's 'Secrets' feature to store your API key
    api_key = userdata.get('GEMINI_API_KEY')
    os.environ['GEMINI_API_KEY'] = api_key
    print("Gemini API key loaded from Colab Secrets.")
except Exception:
    # Fallback if secrets are not configured
    print("Could not find 'GEMINI_API_KEY' in Colab Secrets.")
    api_key = getpass.getpass("Please enter your GEMINI API key: ")
    os.environ['GEMINI_API_KEY'] = api_key
    print("API key entered manually.")

# Configure the genai library with the API key
genai.configure(api_key=os.environ['GEMINI_API_KEY'])

print("✅ Environment setup complete.")
```

### Setting Up Your Gemini API Key

To use Google's Gemini models, you'll need an API key:

1. **Get your API key**: Visit [Google AI Studio](https://aistudio.google.com/app/apikey) to create a free API key
2. **Store it securely in Colab**:
   - Click the key icon (🔑) in the left sidebar of Colab
   - Add a new secret named `GEMINI_API_KEY`
   - Paste your API key as the value
   - Toggle "Notebook access" on

### Understanding the Imports

Key components we're importing:

- **`FastMCP`**: Creates MCP servers with minimal code
- **`MCPClient`**: Connects to and communicates with MCP servers
- **`FastMCPTransport`**: Enables in-process communication between client and server
- **`nest_asyncio.apply()`**: Essential for running async code in Colab

### Common Setup Issues

**Issue**: `ModuleNotFoundError` when importing libraries
- **Solution**: Re-run the first cell to ensure all packages are installed

**Issue**: API key not found in secrets
- **Solution**: Follow the steps above to add your API key to Colab secrets

**Issue**: `RuntimeError: asyncio is already running`
- **Solution**: The `nest_asyncio.apply()` call should resolve this

## Summary

With this setup complete, our Colab environment is ready. We have:
- ✅ Installed all necessary libraries for MCP and Gemini
- ✅ Configured secure access to the Gemini API
- ✅ Set up async support for Colab
- ✅ Imported all required modules

In the next section, we'll define our MCP server and build the agent!
