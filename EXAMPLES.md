# MCP Server Installation - Examples & Configurations

**Real-world configuration examples, troubleshooting scenarios, and platform-specific setups**

---

## Table of Contents

1. [npm-based Servers](#npm-based-servers)
2. [Python-based Servers](#python-based-servers)
3. [Authentication Examples](#authentication-examples)
4. [Troubleshooting Scenarios](#troubleshooting-scenarios)
5. [Platform-Specific Configs](#platform-specific-configs)

---

## npm-based Servers

### Example 1: Filesystem Server (No Auth)

**Use Case:** Local file system access with directory restrictions

**Installation:**
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**Claude Desktop Configuration:**
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "@modelcontextprotocol/server-filesystem",
      "args": ["/Users/jordan/Documents", "/Users/jordan/Projects"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error"
      }
    }
  }
}
```

**Key Points:**
- Multiple allowed directories in args array
- No authentication required
- Paths must be absolute

---

### Example 2: Brave Search (API Key Required)

**Use Case:** Web search integration

**Installation:**
```bash
npm install -g @modelcontextprotocol/server-brave-search
```

**Get API Key:**
1. Visit https://brave.com/search/api/
2. Sign up for free tier (2,000 queries/month)
3. Generate API key

**Claude Desktop Configuration:**
```json
{
  "mcpServers": {
    "brave-search": {
      "command": "@modelcontextprotocol/server-brave-search",
      "env": {
        "BRAVE_API_KEY": "your-api-key-here",
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

---

### Example 3: Fetch Server (HTTP Requests)

**Use Case:** General web content fetching

**Installation:**
```bash
npm install -g @modelcontextprotocol/server-fetch
```

**Configuration with User Agent:**
```json
{
  "mcpServers": {
    "fetch": {
      "command": "@modelcontextprotocol/server-fetch",
      "env": {
        "MCP_MODE": "stdio",
        "USER_AGENT": "MyApp/1.0",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

---

## Python-based Servers

### Example 4: Custom Python Server (System Install)

**Use Case:** Simple Python-based server without conflicts

**Installation:**
```bash
pip install mcp-server-example --break-system-packages
```

**Claude Desktop Configuration:**
```json
{
  "mcpServers": {
    "python-server": {
      "command": "mcp-server-example",
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error"
      }
    }
  }
}
```

---

### Example 5: Python Server (Virtual Environment)

**Use Case:** Python server with dependency conflicts

**Setup:**
```bash
# Create virtual environment
python -m venv ~/mcp-servers/myserver
source ~/mcp-servers/myserver/bin/activate  # Unix/Mac
# or: ~/mcp-servers/myserver/Scripts/activate  # Windows

# Install server
pip install mcp-server-example

# Get Python path
which python  # Unix/Mac
where python  # Windows
```

**Claude Desktop Configuration (Unix/Mac):**
```json
{
  "mcpServers": {
    "python-venv-server": {
      "command": "/Users/jordan/mcp-servers/myserver/bin/python",
      "args": ["-m", "mcp_server_example"],
      "env": {
        "MCP_MODE": "stdio"
      }
    }
  }
}
```

**Claude Desktop Configuration (Windows):**
```json
{
  "mcpServers": {
    "python-venv-server": {
      "command": "~/mcp-servers\\myserver\\Scripts\\python.exe",
      "args": ["-m", "mcp_server_example"],
      "env": {
        "MCP_MODE": "stdio"
      }
    }
  }
}
```

---

## Authentication Examples

### OAuth-based Server

**Use Case:** Services requiring OAuth flow (Gmail, Drive, etc.)

**Installation:**
```bash
npm install -g @modelcontextprotocol/server-gmail
```

**Configuration:**
```json
{
  "mcpServers": {
    "gmail": {
      "command": "@modelcontextprotocol/server-gmail",
      "env": {
        "MCP_MODE": "stdio"
      }
    }
  }
}
```

**First Run:**
- Server will prompt for OAuth consent
- Browser opens for authentication
- Token stored locally for future use
- No API key needed in config

---

### API Token Server

**Use Case:** Services with token-based authentication

**Installation:**
```bash
npm install -g @modelcontextprotocol/server-github
```

**Get GitHub Token:**
1. Go to GitHub Settings → Developer Settings → Personal Access Tokens
2. Generate new token (classic)
3. Select scopes: repo, read:org
4. Copy token

**Configuration:**
```json
{
  "mcpServers": {
    "github": {
      "command": "@modelcontextprotocol/server-github",
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here",
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error"
      }
    }
  }
}
```

---

### Database Connection String

**Use Case:** PostgreSQL/MySQL database access

**Installation:**
```bash
npm install -g @modelcontextprotocol/server-postgres
```

**Configuration:**
```json
{
  "mcpServers": {
    "postgres": {
      "command": "@modelcontextprotocol/server-postgres",
      "args": ["postgresql://user:password@localhost:5432/dbname"],
      "env": {
        "MCP_MODE": "stdio",
        "PGSSLMODE": "require"
      }
    }
  }
}
```

**Security Note:** Connection string contains password. Consider:
- Using environment variables
- Restricting config file permissions
- Using connection pooling
- Enabling SSL/TLS

---

## Troubleshooting Scenarios

### Scenario 1: JSON Parse Error

**Problem:**
```
Error: Unexpected token 'n' in JSON at position 0
```

**Investigation:**
```bash
# Test if package outputs clean JSON
echo '{"test": true}' | @modelcontextprotocol/server-name

# If you see npm messages:
npm WARN deprecated package@1.0.0: Use newer version
{"result": "data"}  # ← This is the problem
```

**Solution:**
```bash
# Pre-install globally
npm install -g @modelcontextprotocol/server-name

# Update config to use direct command
{
  "command": "@modelcontextprotocol/server-name",  # Not npx!
  "env": {
    "DISABLE_CONSOLE_OUTPUT": "true"
  }
}
```

**Verification:**
```bash
# Should return ONLY JSON, no npm messages
@modelcontextprotocol/server-name --version
```

---

### Scenario 2: Server Loads But Tools Don't Work

**Problem:**
- Server appears in tools list
- No error messages
- Functions return empty results

**Investigation Checklist:**

```json
{
  "mcpServers": {
    "debug-mode": {
      "command": "server-name",
      "args": ["--verbose"],
      "env": {
        "LOG_LEVEL": "debug",
        "DEBUG": "*",
        "NODE_ENV": "development"
      }
    }
  }
}
```

**Common Causes & Solutions:**

| Cause | How to Identify | Solution |
|-------|----------------|----------|
| Missing API key | Logs show "401 Unauthorized" | Add API_KEY to env |
| Invalid paths | Logs show "ENOENT" | Use absolute paths |
| Wrong permissions | Logs show "EACCES" | Check file permissions |
| Firewall blocking | Logs show "ETIMEDOUT" | Configure firewall |

---

### Scenario 3: Slow Performance

**Problem:**
- First request takes 30+ seconds
- Subsequent requests fast

**Diagnosis:**
```bash
# Time the command
time server-name --version

# If >5 seconds, it's downloading on-demand
```

**Solution:**
```bash
# Pre-install the package
npm install -g server-name

# Verify it's fast now
time server-name --version  # Should be <1s
```

**Performance Benchmarks:**

| Operation | Target | After Pre-Install |
|-----------|--------|-------------------|
| Server start | <1s | ✅ 0.2s |
| Simple query | <500ms | ✅ 100ms |
| Complex query | <5s | ✅ 2s |

---

### Scenario 4: Works on Mac, Fails on Windows

**Problem:**
- Configuration works on macOS
- Same config fails on Windows

**Common Issues:**

**Path Separators:**
```json
// ❌ Won't work on Windows
{
  "args": ["/Users/jordan/files"]
}

// ✅ Works on both
{
  "args": ["~/files"]  // Forward slashes work!
}

// ✅ Windows-specific
{
  "args": ["~/files"]  // Escaped backslashes
}
```

**Command Resolution:**
```json
// ❌ Might not work cross-platform
{
  "command": "python"  // Could be python3 on Mac
}

// ✅ Platform-specific configs
// macOS:
{
  "command": "/usr/local/bin/python3"
}

// Windows:
{
  "command": "C:\\Python312\\python.exe"
}
```

---

## Platform-Specific Configs

### macOS Configuration (Apple Silicon)

**Homebrew Prefix:**
```bash
# Check Homebrew location
brew --prefix
# Apple Silicon: /opt/homebrew
# Intel: /usr/local
```

**Example with Homebrew Node:**
```json
{
  "mcpServers": {
    "server-name": {
      "command": "/opt/homebrew/bin/server-name",
      "env": {
        "PATH": "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin"
      }
    }
  }
}
```

---

### Windows Configuration (PowerShell)

**Execution Policy Issues:**
```powershell
# Check current policy
Get-ExecutionPolicy

# If Restricted, allow scripts
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**Path with Spaces:**
```json
{
  "mcpServers": {
    "server": {
      "command": "C:\\Program Files\\nodejs\\server.exe",
      // Or use 8.3 name:
      // "command": "C:\\PROGRA~1\\nodejs\\server.exe"
    }
  }
}
```

---

### Linux Configuration (System-wide)

**User vs System Install:**
```bash
# User install (recommended)
pip install --user mcp-server-name
# Location: ~/.local/bin

# System install (requires sudo, not recommended)
sudo pip install mcp-server-name
# Location: /usr/local/bin
```

**Configuration:**
```json
{
  "mcpServers": {
    "python-server": {
      "command": "/home/jordan/.local/bin/mcp-server-name",
      "env": {
        "PYTHONPATH": "/home/jordan/.local/lib/python3.11/site-packages"
      }
    }
  }
}
```

---

## Complete Multi-Server Example

**Real-world configuration with multiple servers:**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "@modelcontextprotocol/server-filesystem",
      "args": ["/Users/jordan/Documents", "/Users/jordan/Projects"],
      "env": {
        "MCP_MODE": "stdio"
      }
    },
    "brave-search": {
      "command": "@modelcontextprotocol/server-brave-search",
      "env": {
        "BRAVE_API_KEY": "BSA_xxx",
        "MCP_MODE": "stdio",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    },
    "github": {
      "command": "@modelcontextprotocol/server-github",
      "env": {
        "GITHUB_TOKEN": "ghp_xxx",
        "MCP_MODE": "stdio"
      }
    },
    "postgres": {
      "command": "@modelcontextprotocol/server-postgres",
      "args": ["postgresql://user:pass@localhost:5432/db"],
      "env": {
        "MCP_MODE": "stdio",
        "PGSSLMODE": "require"
      }
    },
    "custom-python": {
      "command": "/Users/jordan/mcp-servers/custom/bin/python",
      "args": ["-m", "custom_server"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error"
      }
    }
  }
}
```

**Testing Sequence:**
```
1. "What MCP servers are available?"
2. "List files in my Documents folder"
3. "Search the web for MCP protocol documentation"
4. "Show my GitHub repositories"
5. "Query the database for user statistics"
6. "Use the custom server to [specific operation]"
```

---

## Quick Reference

### Installation Commands

```bash
# npm global install
npm install -g package-name

# Python system install
pip install package-name --break-system-packages

# Python virtual environment
python -m venv ~/mcp-servers/venv-name
source ~/mcp-servers/venv-name/bin/activate
pip install package-name

# Verify installation
which package-name  # Unix/Mac
where package-name  # Windows
npm list -g package-name  # npm
pip show package-name  # Python
```

### Configuration Checklist

- [ ] Package pre-installed globally/in venv
- [ ] Command field uses correct executable name
- [ ] Args use absolute paths
- [ ] Required environment variables set
- [ ] MCP_MODE set to "stdio"
- [ ] LOG_LEVEL set appropriately
- [ ] DISABLE_CONSOLE_OUTPUT for npm packages
- [ ] API keys/tokens configured
- [ ] Configuration JSON is valid

### Testing Checklist

- [ ] Host application restarted
- [ ] Server loads without errors
- [ ] Documentation tool works
- [ ] Simple operation succeeds
- [ ] Error handling works
- [ ] Performance acceptable (<1s typical)
- [ ] No warnings in logs

---

## Related Documentation

- [README.md](README.md) - Project overview
- [ARCHITECTURE.md](ARCHITECTURE.md) - Detailed protocol and decision trees
- [IMPLEMENTATION.md](IMPLEMENTATION.md) - n8n MCP case study

---

**Author:** Jordan Waxman  
**Last Updated:** November 2025  
**Examples Version:** 1.0
