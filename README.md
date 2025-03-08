# MCPstarter

Here’s a rewritten version of the Medium article "Build an AI Trading Agent Using Anthropic’s MCP — Part I" by Zheng "Bruce" Li, published in *The Low End Disruptor*. I’ve rephrased the content in my own words while preserving the original meaning, structure, and intent. The goal is to make it clear, concise, and engaging for readers interested in AI and trading.

---

# Create an AI-Powered Forex Trading Agent with Anthropic’s MCP — Part I  
**By [Your Name] · Published in *The Low End Disruptor* · 10 min read · 3 days ago**

---

## Welcome to the Tutorial Series  
In this three-part guide, we’ll harness Anthropic’s **Model Context Protocol (MCP)** to craft an AI agent that trades foreign currencies using real-time market insights. Here’s the breakdown:  

- **Part I**: An introduction to MCP and building your first MCP server  
- **Part II**: Linking MCP servers for trading operations  
- **Part III**: Combining everything into a self-sufficient trading agent  

Through practical, step-by-step examples, you’ll discover how MCP equips AI agents with the flexibility and power to tackle complex, open-ended tasks by tapping into a range of tools and data sources.

---

## Our Goal: Forex Trading Between USD and EUR  
We’re building an AI agent with a specific mission:  
- Trade forex between **USD and EUR**.  
- Track **social media chatter** (especially on Truth Social) about US-Europe economic ties—like tariffs or trade agreements—that could sway exchange rates.  
- Begin with an initial investment and measure returns over a few days.  

### Tools We’ll Need  
- **Anthropic’s MCP**: The backbone of our project.  
- Real-time forex rate data.  
- Access to Truth Social posts via its API.  
- Extra tools for web searches, macroeconomic news, and Web3 insights.  

---

## MCP Explained: The Basics  
**Model Context Protocol (MCP)**, developed by Anthropic, is an open standard that simplifies how AI systems connect to diverse data and tools. Think of it as a universal adapter that lets large language models (LLMs) interact effortlessly with external resources.

### Why Use MCP?  
Building AI agents often involves linking them to data, APIs, and memory systems. MCP streamlines this process with a lightweight, standardized approach—no need for bulky frameworks. Here’s what it brings to the table:  

- **Easier Integration**: MCP cuts through the chaos of custom coding for every tool or data source, saving time and effort.  
- **Faster Results**: Direct access to data means quicker, more precise AI responses.  
- **Universal Compatibility**: It creates a seamless bridge between different AI models and data platforms.  

### How MCP Operates  
MCP follows a **client-server setup**:  
- **MCP Servers**: Small programs that share data or tools—like APIs or databases—using MCP’s standard format.  
- **MCP Clients**: AI apps that tap into these servers to use the available resources.  

You can whip up MCP servers with SDKs in **Python** or **TypeScript**, defining what data or functions the AI can access. Ready-made clients include Anthropic’s **Claude Desktop**, **Cursor**, and **LibreChat**.

### MCP in Action Today  
Since its debut in **November 2024**, MCP is picking up steam:  
- **Pioneers**: Companies like Block and Apollo, plus dev tools like Zed and Replit, are jumping on board.  
- **Open-Source Growth**: Over 1,100 pre-built MCP servers exist, linking to platforms like Google Drive and GitHub.  

### Why MCP Matters for Crypto and Web3  
MCP shines in connecting AI to **blockchain data** and **Web3 tools** (think wallets or decentralized apps). For trading, this means tapping into real-time crypto ecosystems—like **Solana’s Agent Kit** by SendAI, which uses MCP for over 100 blockchain actions (e.g., token swaps). Check out Anthropic’s [MCP workshop](https://youtu.be/kQmXtrmQ5Zg) for more.

---

## Hands-On: Build Your First MCP Server  
Let’s dive in by creating an MCP server to fetch forex rates from [Open Exchange Rates](https://openexchangerates.org/). You’ll need an API key—sign up there first.

### Step 1: Set Up the Server  
Clone Anthropic’s [quickstart repo](https://github.com/modelcontextprotocol/quickstart-resources), test it, then create a file named `trader.py` with this code:  

```python
from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

# Start the MCP server
mcp = FastMCP("trader")

# Forex API setup
APP_ID = "YOUR_API_KEY"
BASE_URL = "https://openexchangerates.org/api/latest.json"

@mcp.tool()
async def get_forex_rate(currencies: list[str]) -> str:
    """Fetch current USD exchange rates for given currencies.
    
    Args:
        currencies: List of currency codes (e.g., ['EUR', 'GBP'])
    """
    params = {
        "app_id": APP_ID,
        "symbols": ",".join(currencies),
        "prettyprint": "false",
        "show_alternative": "false"
    }
    headers = {"accept": "application/json"}

    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(BASE_URL, params=params, headers=headers)
            response.raise_for_status()
            data = response.json()
            rates = [f"Current USD Rates:"]
            for code, rate in data["rates"].items():
                rates.append(f"USD to {code}: {rate:.4f}")
            return "\n".join(rates)
        except Exception as e:
            return f"Error: {str(e)}"

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

This sets up a server with a tool to grab forex rates.

### Step 2: Connect to Claude Desktop  
Add this to your Claude Desktop config file (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):  

```json
{
    "mcpServers": {
        "trader": {
            "command": "uv",
            "args": [
                "--directory",
                "/YOUR/PROJECT/PATH/",
                "run",
                "trader.py"
            ]
        }
    }
}
```

Update the path, restart Claude Desktop, and ask: “What’s today’s EUR/USD rate?” It’ll fetch the data for you.

---

## Bonus: Fetch Truth Social Posts  
Let’s add a tool to pull posts from Truth Social using the [truthbrush API](https://github.com/stanfordio/truthbrush). You’ll need a Truth Social account. Add your credentials to a `.env` file:  

```
TRUTHSOCIAL_USERNAME=YOUR_USERNAME
TRUTHSOCIAL_PASSWORD=YOUR_PASSWORD
```

Then, append this to `trader.py`:  

```python
@mcp.tool()
async def truth_posts(handle: str = "realDonaldTrump", days: int = 1) -> str:
    """Grab recent Truth Social posts from a handle.
    
    Args:
        handle: User’s handle (e.g., realDonaldTrump)
        days: Days of posts to fetch
    """
    from truthbrush.api import Api
    import datetime
    
    try:
        api = Api()
        cutoff = datetime.datetime.now(datetime.timezone.utc) - datetime.timedelta(days=days)
        posts = []
        for post in api.pull_statuses(username=handle, created_after=cutoff, replies=False, pinned=False):
            posts.append(f"Time: {post['created_at']}\nPost: {post['content']}")
        return "\n---\n".join(posts) if posts else f"No posts from @{handle} in {days} day(s)"
    except Exception as e:
        return f"Error: {str(e)}"
```

Restart Claude Desktop—now you’ve got two tools!

**Note**: Run this locally where you’re logged into Truth Social due to API restrictions.

---

## Tap Into Remote MCP Servers  
You can also link to remote MCP servers like [Search1API](https://github.com/fatwang2/search1api-mcp) for web searches and news. Get a free API key, then update your config:  

```json
{
    "mcpServers": {
        "trader": {
            "command": "uv",
            "args": ["--directory", "/YOUR/PROJECT/PATH/", "run", "trader.py"]
        },
        "search1api": {
            "command": "npx",
            "args": ["-y", "search1api-mcp"],
            "env": {"SEARCH1API_KEY": "YOUR_KEY"}
        }
    }
}
```

Restart Claude Desktop, and you’ll have access to both local and remote tools.

---

## Test It All Together  
With forex rates, Truth Social posts, and Search1API tools ready, try this prompt in Claude Desktop:  

**Prompt**: “I’m a forex trader. Build a EUR/USD strategy for the next 7 days using current macro news, social media, and exchange rates.”  

Watch as it pulls data from all your tools—no extra coding required. That’s MCP’s magic at work.

---

## Wrapping Up  
In Part I, we:  
- Explored MCP and its value for AI agents.  
- Built a local MCP server for forex and Truth Social data.  
- Connected to a remote server for extra tools.  
- Showed how MCP ties everything together for smart trading decisions.  

Next up: **Part II**—using MCP for on-chain trading actions. Stay tuned!

---

**Resources**:  
- [MCP Quickstart](https://modelcontextprotocol.io/quickstart/server)  
- [Forex API Docs](https://docs.openexchangerates.org/reference/api-introduction)  
- [Truth Social API](https://github.com/stanfordio/truthbrush)  
- [Search1API](https://github.com/fatwang2/search1api-mcp)  

--- 

This version keeps the original’s spirit while delivering a fresh, reader-friendly take. Let me know if you’d like tweaks!
