Here’s how you can **set up the Strapi MCP server on Windows** so it works with **Claude Desktop** step-by-step.

---

## **1. Install Node.js (if you don’t have it yet)**

The Strapi MCP server is distributed as an NPM package, so you need Node.js installed.

* Download and install Node.js LTS version:
  [https://nodejs.org/en/download](https://nodejs.org/en/download)
* During installation:

  * ✅ **Check the box** for *"Add to PATH"* so you can run `node` and `npm` in the terminal.

To verify installation:

```powershell
node -v
npm -v
```

---

## **2. Configure Claude Desktop**

Claude Desktop uses a `claude_desktop_config.json` file to define MCP servers.

> 📂 **Location of config file on Windows**
> Claude Desktop stores its configuration in:
>
> ```
> %USERPROFILE%\.claude\claude_desktop_config.json
> ```
>
> Example: `C:\Users\YourName\.claude\claude_desktop_config.json`

If the file doesn’t exist yet, create it manually.

---

### **Add Strapi MCP Server to config**

Open `claude_desktop_config.json` and add this:

```json
{
  "mcpServers": {
    "strapi": {
      "command": "npx",
      "args": ["-y", "@bschauer/strapi-mcp-server@2.6.0"]
    }
  }
}
```

---

## **3. Create Strapi MCP Configuration File**

The Strapi MCP server needs its own config file to connect to your Strapi instance.

Create a folder for MCP configs:

```powershell
mkdir %USERPROFILE%\.mcp
```

Now create a file called:

```
%USERPROFILE%\.mcp\strapi-mcp-server.config.json
```

Add the following JSON (edit with your Strapi details):

```json
{
  "myserver": {
    "api_url": "http://localhost:1337",
    "api_key": "your-jwt-token-from-strapi-admin",
    "version": "5.*"
  }
}
```

### **Getting your JWT Token**

1. Go to your Strapi Admin Panel → *Settings → API Tokens*
2. Create a new **Full Access Token** (or with specific permissions)
3. Copy the token into `api_key` above.

> ⚠️ **Security Tip:**
> Use a *read-only token* for testing if you don’t need to create or modify content.

---

## **4. Test Strapi MCP Server**

Before running Claude, test if the MCP server works directly.

Open **Command Prompt** or **PowerShell**, then run:

```powershell
npx -y @bschauer/strapi-mcp-server@2.6.0
```

If everything is correct, you should see it start up without errors.
If there’s a problem, check:

* Your `strapi-mcp-server.config.json` formatting
* Your Strapi API URL and token
* That your Strapi instance is running

---

## **5. Restart Claude Desktop**

Now restart Claude Desktop so it picks up the new MCP server.

Once restarted:

* Open Claude Desktop
* Type:

  ```
  /mcp list
  ```

  You should see **`strapi`** listed as a connected MCP server.

---

## **6. Test Commands in Claude**

Try some basic commands to confirm everything works:

### **List all configured Strapi servers**

```
/mcp strapi_list_servers
```

Expected response:

```json
{
  "servers": [
    {
      "name": "myserver",
      "version": "5.*"
    }
  ]
}
```

### **Get Content Types**

```
/mcp strapi_get_content_types { "server": "myserver" }
```

---

## **7. Troubleshooting**

If something isn’t working:

### **Check logs**

Run Claude Desktop from the terminal:

```powershell
"C:\Program Files\Claude Desktop\Claude.exe" --verbose
```

This will show connection logs.

### **Common Issues**

| Problem                       | Solution                                           |
| ----------------------------- | -------------------------------------------------- |
| `npx` command not found       | Make sure Node.js is installed and added to PATH   |
| 404 errors from Strapi        | Check plural/singular endpoint names and API URL   |
| Auth errors                   | Verify JWT token and permissions                   |
| Version mismatch errors       | Update `"version"` in config to `"4.*"` or `"5.*"` |
| Claude doesn’t see MCP server | Restart Claude after editing config                |

---

## **Final File Structure Recap**

Here’s how your files should be laid out on Windows:

```
C:\Users\<YourName>\
│
├─ .claude\
│   └─ claude_desktop_config.json     <-- Claude config
│
└─ .mcp\
    └─ strapi-mcp-server.config.json  <-- Strapi MCP config
```

---

Would you like me to generate a **ready-to-use config template ZIP** so you can just edit and drop it in place?
