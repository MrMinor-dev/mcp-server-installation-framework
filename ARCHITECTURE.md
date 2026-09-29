# MCP Server Installation - Architecture & Protocol

**Complete installation methodology, decision frameworks, and troubleshooting guides**

---

## Table of Contents

1. [Installation Protocol](#installation-protocol)
2. [Decision Trees](#decision-trees)
3. [Troubleshooting Guide](#troubleshooting-guide)
4. [Platform-Specific Notes](#platform-specific-notes)

---

## Installation Protocol

### Four-Phase Methodology

```
Phase 1: Research → Phase 2: Pre-Install → Phase 3: Configure → Phase 4: Test
   (5 min)            (2 min)              (3 min)            (2 min)
```

**Total Time:** 12 minutes per server (vs 45-60 min traditional approach)

---

### Phase 1: Research & Selection (5 minutes)

**Objective:** Validate server quality before investing time

**Evaluation Checklist:**

| Criterion | Green Flag ✅ | Red Flag ❌ |
|-----------|--------------|-------------|
| GitHub Activity | Commits in last 3 months | No activity >6 months |
| Documentation | Clear setup instructions | Missing or unclear |
| Issues | <10 open, maintainer responsive | 50+ open, no responses |
| Dependencies | Minimal, well-maintained | Complex, outdated |
| Community | Active discussions, examples | No community activity |

**Decision Matrix:**

```
Score each criterion (0-2 points):
├─ 8-10 points → PROCEED (high confidence)
├─ 5-7 points  → PROCEED WITH CAUTION (test thoroughly)
└─ 0-4 points  → SKIP (find alternative)
```

---

### Phase 2: Pre-Installation (2 minutes) ⚡ CRITICAL

**Why Pre-Install?**

| Metric | npx/on-demand | Pre-installed | Improvement |
|--------|---------------|---------------|-------------|
| Setup Time | 60s | 5s | **12x faster** |
| Parse failure on launch | Most launches | Not seen since | Eliminated the stdout parse failure |
| Startup Speed | 30-60s | <1s | **30x faster** |
| Maintenance | Per-invocation | One-time | **Simpler** |

**Installation Commands:**

**npm-based servers:**
```bash
# Global installation (recommended)
npm install -g @modelcontextprotocol/server-name

# Verify
npm list -g @modelcontextprotocol/server-name
which server-name  # Unix/Mac
where server-name  # Windows
```

**Python-based servers:**
```bash
# System-wide (recommended)
pip install mcp-server-name --break-system-packages

# OR virtual environment (more isolated)
python -m venv ~/mcp-servers/server-name
source ~/mcp-servers/server-name/bin/activate
pip install mcp-server-name

# Verify
pip show mcp-server-name
```

**Critical Success Factors:**
- ✅ Package in PATH (test with `which`/`where`)
- ✅ Version confirmed (test with `--version` if supported)
- ✅ No error messages during install
- ✅ Can execute standalone (before MCP config)

---

### Phase 3: Configuration (3 minutes)

**Configuration Template:**

```json
{
  "mcpServers": {
    "descriptive-name": {
      "command": "actual-command-name",
      "args": ["optional", "arguments"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

**Key Configuration Rules:**

1. **command field:**
   - npm global: Use bare package name (e.g., `"server-name"`)
   - Python system: Use bare package name (e.g., `"mcp-server-name"`)
   - Python venv: Use absolute path to Python (e.g., `"/home/user/venv/bin/python"`)
   - Never use `npx` (causes stdout pollution)

2. **args field:**
   - Path arguments: Always use absolute paths
   - API configuration: Use args or env vars per docs
   - Keep minimal initially (add as needed)

3. **env field:**
   - Always include `MCP_MODE: "stdio"`
   - Always include `LOG_LEVEL: "error"` (production)
   - npm packages: Add `DISABLE_CONSOLE_OUTPUT: "true"`
   - API keys: Add as documented by server

**Configuration Decision Tree:**

```
Configuration Strategy
│
├─ Authentication Required?
│  ├─ YES → Add API_KEY/TOKEN to env
│  └─ NO → Skip auth config
│
├─ File System Access?
│  ├─ YES → Add allowed paths to args
│  └─ NO → Skip path config
│
├─ Network Access?
│  ├─ YES → Verify firewall rules
│  └─ NO → Skip network config
│
└─ Custom Settings?
   ├─ YES → Review docs, add to env
   └─ NO → Use defaults
```

**Platform-Specific Path Handling:**

**Windows:**
```json
{
  "command": "C:\\Users\\Username\\venv\\Scripts\\python.exe",
  "args": ["C:\\path\\to\\allowed\\dir"]
}
```

**Unix/Mac:**
```json
{
  "command": "/home/username/venv/bin/python",
  "args": ["/path/to/allowed/dir"]
}
```

---

### Phase 4: Testing & Validation (2 minutes)

**Testing Protocol:**

```
1. Restart host application (Claude Desktop, etc.)
2. Open new conversation
3. Run test sequence:
   ├─ Documentation check
   ├─ Simple operation
   └─ Error handling
4. Verify performance
5. Check logs for warnings
```

**Test Commands:**

**Documentation Test:**
```
"What tools does the [server-name] provide?"
"Show me available functions"
```
**Expected:** List of tools, <1 second response

**Functionality Test:**
```
"Use [server] to [simple safe operation]"
```
**Expected:** Successful execution, reasonable response time

**Error Handling Test:**
```
"Use [server] with [invalid input]"
```
**Expected:** Clear error message, graceful failure

**Performance Benchmarks:**

| Response Type | Target | Acceptable | Investigate |
|--------------|--------|------------|-------------|
| Documentation | <200ms | <500ms | >1s |
| Simple ops | <500ms | <2s | >5s |
| Complex ops | <2s | <10s | >30s |

**Success Criteria:**
- ✅ All tools discoverable
- ✅ Functions execute correctly
- ✅ Errors handled gracefully
- ✅ Performance within targets
- ✅ No warnings in logs

---

## Decision Trees

### Installation Method Selection

```
Which Installation Method?

Package Type?
├─ npm Package
│  └─ Frequency of Use?
│     ├─ Regular (weekly+) → Global install (npm install -g)
│     ├─ Occasional → Global install (consistency > space)
│     └─ Testing only → npx acceptable (but slower)
│
└─ Python Package
   └─ Isolation Required?
      ├─ YES (conflicts expected) → Virtual environment
      ├─ NO (standalone server) → System install
      └─ UNSURE → Virtual environment (safer)
```

### Troubleshooting Decision Tree

```
Server Not Working?

Step 1: Does it load?
├─ NO → Installation Issue
│  ├─ Package installed? → npm list -g / pip show
│  ├─ Command in PATH? → which / where
│  ├─ Config syntax valid? → JSON validator
│  └─ Permissions correct? → Check file permissions
│
└─ YES → Functionality Issue
   ├─ Tools visible? → Documentation tool test
   │  ├─ NO → Protocol error (check logs)
   │  └─ YES → Continue to Step 2
   │
   └─ Step 2: Do tools work?
      ├─ NO → Configuration Issue
      │  ├─ Missing env vars? → Review docs
      │  ├─ Wrong paths? → Verify absolute paths
      │  ├─ Auth failing? → Check API keys
      │  └─ Permissions? → File system access
      │
      └─ YES BUT SLOW → Performance Issue
         ├─ Using npx? → Switch to pre-install
         ├─ Network delays? → Check connectivity
         ├─ Large payloads? → Optimize requests
         └─ Server issue? → Review server logs
```

---

## Troubleshooting Guide

### Common Issue #1: JSON Parsing Errors

**Symptoms:**
```
Error: Unexpected token 'n' in JSON at position 0
Error: Invalid JSON response from server
```

**Root Cause:** npm downloads polluting stdout during package installation

**Diagnosis:**
```bash
# Test if package produces clean JSON
echo '{"jsonrpc":"2.0","method":"initialize","id":1}' | package-name

# Should return valid JSON only
# If you see npm messages, that's the problem
```

**Solution Priority:**

1. **Pre-install globally (BEST):**
   ```bash
   npm install -g package-name
   # Update config to use package-name directly
   ```

2. **Environment variable:**
   ```json
   "env": {
     "DISABLE_CONSOLE_OUTPUT": "true",
     "NODE_ENV": "production"
   }
   ```

3. **Alternative package manager:**
   ```bash
   yarn global add package-name
   pnpm install -g package-name
   ```

**Verification:**
```bash
# Clean output test
package-name --version  # Should be clean, no npm messages
```

---

### Common Issue #2: Command Not Found

**Symptoms:**
```
Error: spawn package-name ENOENT
Command 'package-name' not found
```

**Diagnosis Steps:**

**For npm:**
```bash
# 1. Is it installed?
npm list -g package-name

# 2. What's the install location?
npm config get prefix

# 3. Is that location in PATH?
echo $PATH  # Unix/Mac
echo %PATH%  # Windows

# 4. Can I find the executable?
which package-name  # Unix/Mac
where package-name  # Windows
```

**For Python:**
```bash
# 1. Is it installed?
pip show package-name

# 2. Where are executables installed?
python -m site --user-base

# 3. Is that location in PATH?
echo $PATH

# 4. Try full path
$(python -m site --user-base)/bin/package-name
```

**Solutions:**

**npm - Add to PATH (Windows):**
```cmd
# Find npm global location
npm config get prefix
# Usually: the shared drive/Users\[USERNAME]\AppData\Roaming\npm

# Add to PATH
setx PATH "%PATH%;the shared drive/Users\[USERNAME]\AppData\Roaming\npm"

# Restart terminal and test
where package-name
```

**Python - Add to PATH (Unix/Mac):**
```bash
# Add to ~/.bashrc or ~/.zshrc
export PATH="$PATH:$(python -m site --user-base)/bin"

# Reload shell
source ~/.bashrc

# Test
which package-name
```

---

### Common Issue #3: Slow Startup / Timeouts

**Symptoms:**
- Server takes >10 seconds to respond
- First request times out
- Inconsistent performance

**Common Causes:**

| Cause | How to Identify | Solution |
|-------|----------------|----------|
| Using npx | Config has `"command": "npx"` | Pre-install globally |
| First-time downloads | Logs show "downloading..." | Pre-install dependencies |
| Network issues | Ping fails to API endpoint | Check firewall, proxy |
| Server initialization | Logs show long startup | Increase timeout, optimize server |

**Performance Analysis:**
```bash
# Time the startup
time package-name --version

# Should be <1 second for pre-installed
# If >5 seconds, investigate
```

---

### Common Issue #4: Silent Failures

**Symptoms:**
- Server loads, no errors
- Tools appear but don't execute
- Empty or generic error responses

**Diagnosis Checklist:**

- [ ] Environment variables set? (check config env section)
- [ ] API keys valid? (test with curl/postman)
- [ ] Permissions granted? (file system, network access)
- [ ] Paths accessible? (test cd to paths in args)
- [ ] Firewall blocking? (check network traffic)

**Debug Configuration:**
```json
{
  "mcpServers": {
    "debug-server": {
      "command": "package-name",
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

**Log Analysis:**
1. Restart host with debug config
2. Attempt failing operation
3. Review logs for specific errors
4. Search GitHub issues for error messages

---

## Platform-Specific Notes

### Windows

**Path Handling:**
- Use double backslashes: `C:\\path\\to\\file`
- Or forward slashes: `C:/path/to/file` (works in JSON)
- Avoid spaces in paths (use 8.3 names if necessary)

**Common Locations:**
```
npm global: the shared drive/Users\[USER]\AppData\Roaming\npm
Python user: the shared drive/Users\[USER]\AppData\Local\Programs\Python\PythonXX
```

**PowerShell vs CMD:**
- Test in both if issues arise
- PowerShell may have execution policy restrictions
- Consider WSL for Unix-like environment

---

### macOS

**Path Handling:**
- Standard Unix paths: `/usr/local/bin`, `/home/user/.local/bin`
- Homebrew prefix varies: `/usr/local` (Intel) or `/opt/homebrew` (Apple Silicon)

**Common Locations:**
```
npm global: /usr/local/lib/node_modules
Python user: ~/Library/Python/X.Y/bin
```

**Permission Issues:**
- Avoid using `sudo` for installations
- Use user-specific install locations
- Check Gatekeeper for executable permissions

---

### Linux

**Path Handling:**
- Follow XDG Base Directory specification
- User installs: `~/.local/bin`
- System installs: `/usr/local/bin`

**Common Locations:**
```
npm global: /usr/local/lib/node_modules
Python user: ~/.local/bin
```

**Package Manager Integration:**
- Consider system package manager (apt, yum) if available
- User installs avoid sudo and permission issues

---

## Best Practices Summary

**Installation:**
- ✅ Always pre-install (never rely on npx/on-demand)
- ✅ Use global installs for regular-use servers
- ✅ Use virtual environments for conflict-prone Python servers
- ✅ Verify installation before configuring

**Configuration:**
- ✅ Use absolute paths everywhere
- ✅ Include environment variables for stdout control
- ✅ Start minimal, add complexity incrementally
- ✅ Document custom configurations

**Testing:**
- ✅ Test after every configuration change
- ✅ Verify performance benchmarks
- ✅ Test error handling explicitly
- ✅ Keep test commands documented

**Maintenance:**
- ✅ Pin versions in documentation
- ✅ Update servers periodically
- ✅ Monitor for breaking changes
- ✅ Keep configuration backed up

---

## Related Documentation

- [README.md](README.md) - Project overview and quick start
- [EXAMPLES.md](EXAMPLES.md) - Configuration examples
- [IMPLEMENTATION.md](IMPLEMENTATION.md) - Detailed case study

---

**Author:** Jordan Waxman  
**Last Updated:** November 2025  
**Framework Version:** 1.0
