<h1>
  <span class="headline">Infrastructure, Orchestration, and MCPs</span>
  <span class="subhead">Instructor Guide</span>
</h1>

## Module Overview

This 60-minute module is a guided code walkthrough demonstrating how to create in-process MCP (Model Context Protocol) servers in Google Colab and integrate them with Google Gemini agents. The focus is on practical implementation in a self-contained environment rather than complex external setups.

## Session Structure

### Opening (5 minutes) - Bridge from Previous Module
**Start with Context:** Connect this module to the previous one on task decomposition.

> "In our last session, we learned about orchestrator and worker agents. Today, we'll see how to extend our agents with custom tools using the Model Context Protocol—a standard that's becoming the backbone of agent infrastructure. We'll do this entirely in Google Colab for simplicity."

### Concepts (10 minutes)
- **Focus:** MCP as infrastructure for agents, but simplified for Colab
- **Key Message:** We can create powerful tool integrations without complex setups
- **Teaching Tip:** Use the simplified architecture diagram to show in-process MCP

### Microlesson 1: Colab Environment Setup (15 minutes)
**Critical Steps:**
1. **Install Commands**: Have students run Cell 1, checking for successful installation
2. **API Key Setup**: Guide students through Colab secrets for Gemini API key
3. **Import Verification**: Ensure all imports work correctly in Cell 2

**Common Issues:**
- Missing Gemini API key (provide link to get free one)
- Import errors (re-run installation cell)
- Async issues (nest_asyncio should handle this)

### Microlesson 2: Code Walkthrough (35 minutes)
**Part 1: Server Definition (10 minutes)**
- Run Cell 3 to define the MCP server
- Explain the `@mcp_server.tool()` decorator
- Show how docstrings become tool descriptions

**Part 2: Agent Integration (20 minutes)**
- Build Cell 4 step by step if time allows
- Emphasize the `FastMCPTransport` for in-process communication
- Show the agent making tool calls and getting results

**Part 3: Enhancement Discussion (5 minutes)**
- Discuss the pseudocode for city name lookup
- Encourage students to experiment with different queries

## Teaching Tips

### For Live Coding in Colab
- **Share Your Notebook**: Use Colab's share feature so students can follow along
- **Run Cells Sequentially**: Emphasize running cells in order
- **Check Output**: After each cell, verify students see expected output
- **Handle Errors Together**: Use common errors as teaching moments

### For Environment Issues
- **Gemini API Key**: Have backup instructions for manual entry
- **Library Installation**: The `-q` flag keeps output minimal but functional
- **Async Support**: `nest_asyncio` should handle most Colab async issues

### For Explanations
- **In-Process Advantage**: Emphasize how this approach avoids network complications
- **MCP Benefits**: Focus on standardization and tool reusability
- **Real-world Context**: Mention how this scales to production environments

## Common Student Questions

**Q: "Why use Google Gemini instead of OpenAI?"**
A: Gemini has excellent function calling capabilities and generous free tier. The MCP pattern works with any LLM that supports function calling.

**Q: "How is this different from regular function calling?"**
A: MCP provides a standard protocol that works across different AI frameworks. Your tools can be reused with different agents and applications.

**Q: "Can I use this approach outside of Colab?"**
A: Absolutely! FastMCP works in any Python environment. Colab just makes it easy to get started without local setup.

**Q: "What if I want to deploy this in production?"**
A: You'd typically run MCP servers as separate processes or microservices. This in-process approach is perfect for development and prototyping.

## Timing Management

**If Running Behind:**
- Focus on getting Cells 1-4 working successfully
- Skip detailed explanation of the pseudocode enhancement
- Demonstrate the final working agent quickly

**If Running Ahead:**
- Have students modify the query to try different cities
- Show what happens with invalid coordinates
- Discuss how to add more sophisticated tools

## Assessment Opportunities

### Formative Assessment
- Check that students can run all 4 cells successfully
- Verify they understand the MCP client-server communication
- Observe if students can explain what each decorator does

### Extension Challenges
- Modify the agent to handle multiple weather queries
- Add error handling for invalid coordinates
- Create a new tool that uses a different API

## Technical Prerequisites Check

Before starting, ensure students have:
- Google account with Colab access
- Google Gemini API key (guide them to get free one)
- Basic understanding of Python functions
- Familiarity with Jupyter/Colab cell execution

## Troubleshooting Guide

### Installation Failures
```python
# If pip install fails, try:
!pip install --upgrade pip
!pip install -q google-genai==0.5.4 mcp==1.9.4 fastmcp==2.8.1 httpx==0.28.1 nest_asyncio==1.6.0
```

### API Key Issues
- Guide students to [Google AI Studio](https://aistudio.google.com/app/apikey)
- Show how to add secrets in Colab (key icon in sidebar)
- Verify `userdata.get('GEMINI_API_KEY')` returns a value

### MCP Communication Errors
- Ensure cells are run in order
- Check that `mcp_server` is defined before use
- Verify `FastMCPTransport` is properly initialized

### Async Errors
- Make sure `nest_asyncio.apply()` is called early
- Use `asyncio.run()` for main execution
- Don't mix sync and async code patterns

## Additional Resources for Instructors

- [Google AI Studio](https://aistudio.google.com/) for API keys
- [FastMCP Documentation](https://github.com/jlowin/fastmcp)
- [MCP Protocol Specification](https://github.com/modelcontextprotocol/spec)
- [Google Colab Tips](https://colab.research.google.com/notebooks/basic_features_overview.ipynb)

---

🏗️ **Pro Tips for Success**

1. **Start Simple**: Get the basic flow working before explaining advanced concepts
2. **Use Colab Features**: Leverage secrets, text cells for explanations, and easy sharing
3. **Embrace the Magic**: Students love seeing the agent make live API calls
4. **Connect to Reality**: Mention how this pattern scales to enterprise applications
5. **Keep It Interactive**: Have students modify queries and see different results

**Remember:** This is about demonstrating the power of MCP in the simplest possible way. The goal is for students to leave with a working understanding of how to extend AI agents with custom tools using industry-standard protocols.

**The "Wow Moment":** When the agent successfully calls the weather tool and returns real data - that's when students understand the power of this approach!
