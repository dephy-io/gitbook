# How to Enable DePHY MCP Server

1\. Register an Account

Visit [https://mesh.dephy.io/](https://mesh.dephy.io/) and register an account

## 2. Get an API Key

Visit [API Keys page](https://mesh.dephy.io/dashboard/api-keys) to check if you have at least one API key.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

If there's no API keys available, click **Generate New API Key** to generate a new one.

## 3. Copy the MCP Endpoint URL(s)

Visit [Service page](https://mesh.dephy.io/dashboard/services) to get the endpoint URL(s).

Currently(20250509) we have 5 different MCP servers with 16 tools together. We highly recommend you copy the **Unified Service Endpoint** URL to use. It's an aggregator of all MCPs. If you have special needs you can still copy the separating service URLs.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## 4. Paste MCP Endpoint URL to Your AI Clients

For now our endpoint is SSE type MCP endpoint, please choose a SSE-MCP-supported clients to enable it.

We recommend serveral AI clients:

* [ChatWise](https://chatwise.app/?atp=nVf4iC) (macOS, Windows)
* [OpenCat](https://opencat.app/) (macOS, iOS, iPadOS)
* [CherryStudio](https://www.cherry-ai.com/) (macOS, Windows, Linux)

Paste the MCP endpoint to your AI client MCP setting page:

* Choose a name(or id) for this MCP
* Paste the link
* Choose the type SSE
* We highly recommend you check "without confirmation" or "no confirmation" checkbox to make it run smoothly.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Screenshot of ChatWise MCP setting</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>Screenshot of OpenCat MCP setting</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption><p>Screenshot of CherryStudio MCP setting</p></figcaption></figure>

For clients don't support SSE type(for example Claude), you can still use it by using a bridge tool. In this example we use `mcp-remote` in `claude_desktop_config.json` :&#x20;

{% code overflow="wrap" %}
```json
{
    "mcpServers": {
      "DePHY": {
        "command": "npx",
        "args" : [
          "mcp-remote",
          "https://mcp-demo.dephy.dev/mcp/{replace-your-api-key-here}/sse",
          "--transport",
          "sse-only"
        ]
      }
    }
}
```
{% endcode %}

## Check if MCP is Enabled

Once MCP server is enabled, you can see a tool toggle button in most AI chat apps:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption><p>ChatWise MCP Toggle Button</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption><p>OpenCat MCP Toggle Button</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption><p>CherryStudio MCP Toggle Button</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption><p>Claude MCP Toggle Button</p></figcaption></figure>

If DePHY MCP doesn't appear in your chat app, please recheck your settings.

{% hint style="info" %}
For some apps(like Claude), you need to quit and restart the app to enable the new-added MCP server.
{% endhint %}
