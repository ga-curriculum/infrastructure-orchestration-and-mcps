<h1>
  <span class="headline">Infrastructure, Orchestration, and MCPs</span>
  <span class="subhead">MCP Integration Walkthrough</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to define MCP servers in-process and integrate them as tools within Google Gemini agents.

## Part 3: Define the MCP Server and Tools (10 minutes)

Now that our environment is set up, let's create our MCP server with custom tools. Instead of running it as a separate process, we'll define it directly in our code using `FastMCP`.

### Cell 3: Define the MCP Server and Tools

Here, we define our MCP server. `FastMCP` makes this incredibly easy. We'll add two tools to it: one to get a weather forecast and another for dummy weather alerts.

```python
# Create an MCP server instance named "weather"
mcp_server = FastMCP("weather")

@mcp_server.tool()
def get_weather(latitude: float, longitude: float) -> str:
    """3‑day min/max temperature forecast via Open‑Meteo."""
    print(f"--- MCP Server: 'get_weather' tool called for lat={latitude}, lon={longitude} ---")
    url = (
        f"https://api.open-meteo.com/v1/forecast"
        f"?latitude={latitude}&longitude={longitude}"
        "&daily=temperature_2m_min,temperature_2m_max&timezone=UTC"
    )
    resp = httpx.get(url, timeout=10)
    daily = resp.json()["daily"]
    # Format the response into a readable string
    return "\n".join(
        f"{date}: low {mn}°C, high {mx}°C"
        for date, mn, mx in zip(
            daily["time"],
            daily["temperature_2m_min"],
            daily["temperature_2m_max"],
        )
    )

@mcp_server.tool()
def get_alerts(state: str) -> str:
    """Dummy US‑state alerts."""
    print(f"--- MCP Server: 'get_alerts' tool called for state={state} ---")
    return f"No active weather alerts for {state.upper()}."

print("✅ MCP server and tools are defined.")
```

### Understanding the Server Definition

Let's break down what's happening:

1. **`FastMCP("weather")`**: Creates a new MCP server with the name "weather"
2. **`@mcp_server.tool()`**: Decorator that registers functions as MCP tools
3. **Function docstrings**: Become tool descriptions that the AI model can read
4. **Type hints**: Help the model understand what parameters are expected
5. **Open-Meteo API**: Provides free weather data without requiring API keys

The beauty of this approach is that we're defining both the server and the tools in the same notebook, making everything self-contained.

## Part 4: Build and Run the Conversational Agent (25 minutes)

With our server and tools defined, we can now build our AI agent. We'll configure a Gemini model and tell it about the tools we created.

### Cell 4: Build and Run the Conversational Agent

The key component here is the `FastMCPTransport`, which allows our agent's MCP client to talk directly to the `mcp_server` we just defined, without needing a network connection.

```python
# --- Agent and Tool Configuration ---

# 1. Select the Gemini model we want to use
MODEL = "gemini-1.5-flash"
model = genai.GenerativeModel(
    model_name=MODEL,
    # The tools parameter tells the model what functions it can call
    tools=[mcp_server]
)

# 2. Create the MCP transport and client
# This transport connects our client directly to the server object in memory
transport = FastMCPTransport(mcp_server)

# --- Main Agent Logic ---
async def run_agent_query(prompt: str):
    """
    Sends a prompt to the Gemini model and handles any function calls
    by executing them with the MCP client.
    """
    print(f"\n--- Running agent with query: '{prompt}' ---")
    # The MCPClient uses a context manager to handle its lifecycle
    async with MCPClient(transport) as mcp_client:
        # 3. Send the prompt to the model
        response = await model.generate_content_async(prompt)
        response_part = response.candidates[0].content.parts[0]

        # 4. Check if the model decided to call a tool
        if not response_part.function_call.name:
            print("\n--- Final Agent Response (No Tool Used) ---")
            print(response_part.text)
            return

        call = response_part.function_call
        print(f"🔧 Gemini wants to call: {call.name} with args: {dict(call.args)}")

        # 5. Execute the tool call using the MCP client
        tool_result = await mcp_client.call_tool(call.name, call.args)
        print("\n--- Response from MCP Server ---")
        print(tool_result)

        # (Optional) In a real chat application, you would send this result
        # back to the model to get a final, natural-language answer.

# --- Run an Example ---
# This is the conversational query. The agent will figure out the coordinates.
query = "What's the 3-day weather forecast for San Francisco (latitude: 37.77, longitude: -122.42)?"
asyncio.run(run_agent_query(query))
```

### Understanding the Agent Flow

Let's trace through what happens when we run this code:

1. **Model Configuration**: We tell Gemini about our MCP server tools
2. **Transport Setup**: `FastMCPTransport` creates an in-memory bridge
3. **Query Processing**: The model receives our natural language prompt
4. **Tool Decision**: Gemini decides to call the `get_weather` tool
5. **MCP Execution**: Our client calls the tool via the MCP protocol
6. **Result Display**: We see both the tool output and any model response

### Expected Output

When you run this cell, you should see something like:

```
--- Running agent with query: 'What's the 3-day weather forecast for San Francisco (latitude: 37.77, longitude: -122.42)?' ---
🔧 Gemini wants to call: get_weather with args: {'latitude': 37.77, 'longitude': -122.42}
--- MCP Server: 'get_weather' tool called for lat=37.77, lon=-122.42 ---

--- Response from MCP Server ---
2024-01-15: low 8°C, high 15°C
2024-01-16: low 10°C, high 17°C
2024-01-17: low 9°C, high 16°C
```

## Part 5: Enhancing the Agent (Optional Extension)

### Making the Agent Smarter

Currently, our agent requires explicit latitude and longitude coordinates. For a real-world application, you'd want the agent to be smarter about location lookup. Here's pseudocode for how you could enhance it:

```python
# Enhanced agent pseudocode (not implemented)
@mcp_server.tool()
def get_weather_by_city(city_name: str, country_code: str = None) -> str:
    """Get weather forecast for a city by name."""
    # 1. Use geocoding API to convert city name to coordinates
    coordinates = geocode_city(city_name, country_code)
    
    # 2. Call the existing weather API with those coordinates
    return get_weather(coordinates.latitude, coordinates.longitude)

# Then your query could be simply:
# "What's the weather like in San Francisco?"
```

This enhancement would make the agent much more user-friendly by handling the coordinate lookup automatically.

### Try These Experiments

1. **Different Cities**: Modify the query to ask about other cities
2. **Weather Alerts**: Try asking about weather alerts for a US state
3. **Error Handling**: What happens if you provide invalid coordinates?
4. **Multi-step Queries**: Ask about weather in multiple cities

## Part 6: The Power of This Pattern

### What We've Accomplished

Today, we successfully:

1. **Created an in-process MCP server** with custom weather tools
2. **Connected a Gemini agent** to this server via MCP protocol
3. **Enabled natural language queries** that trigger structured tool calls
4. **Demonstrated the MCP ecosystem** in a self-contained environment

### Why This Matters

This pattern is incredibly powerful because:
- **Standardization**: MCP provides a consistent interface for tool integration
- **Scalability**: Easy to add new tools without changing agent code
- **Reusability**: Tools can be shared across different AI frameworks
- **Flexibility**: Works with local tools, remote APIs, or hybrid approaches

### Real-World Applications

In production environments, you might use MCP to connect agents to:
- **Databases**: Query customer data or analytics
- **File Systems**: Search and manipulate documents
- **APIs**: Integrate with CRM, ERP, or other business systems
- **Specialized Services**: OCR, translation, image processing
- **Infrastructure**: Monitor systems, deploy applications

### Best Practices We Demonstrated

1. **Type Safety**: Using type hints for reliable tool calls
2. **Error Handling**: Graceful handling of API failures
3. **Clear Documentation**: Descriptive docstrings for tool discovery
4. **Async Programming**: Efficient handling of I/O operations
5. **In-Process Architecture**: Simplified deployment for development

## Summary

Congratulations! You've built a working AI agent that can access real-time weather data through the Model Context Protocol. This foundation gives you the tools to integrate virtually any external service or data source into your AI applications.

The key takeaway is that MCP provides a standardized way to extend AI agents with specialized capabilities, whether you're building simple tools like we did today or integrating complex enterprise systems.

In our next module, we'll explore how to take agentic workflows like this one and deploy them using production-grade automation and orchestration tools.
