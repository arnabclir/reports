# OpenCode Installation Issue Report

**Date:** 2026-01-11
**Installation Method:** `curl -fsSL https://opencode.ai/install | bash`
**OpenCode Version:** 1.1.13
**Installation Status:** **FAILED** - Critical runtime errors despite successful installation

---

## Environment Information

**OS:** Linux 6.17.0-PRoot-Distro
**Shell:** bash
**Python Version:** 3.13.7
**Curl Version:** 8.18.0

### Current PATH (before sourcing .bashrc):
```
/home/arnab/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games:/data/data/com.termux/files/usr/bin:/system/bin:/system/xbin
```

---

## Issue Description

The OpenCode installation script successfully downloads and installs the binary to `/home/arnab/.opencode/bin/opencode`, but the command is not immediately accessible in the current shell session after installation.

### Installation Process:

1. **Initial pip attempt failed:**
   ```bash
   pip install opencode
   ERROR: Could not find a version that satisfies the requirement opencode
   ERROR: No matching distribution found for opencode
   ```

2. **Curl install script execution appeared successful:**
   ```bash
   curl -fsSL https://opencode.ai/install | bash
   ```

   Output showed:
   - Installing opencode version: 1.1.13
   - Successfully added opencode to $PATH in /home/arnab/.bashrc
   - Displayed success message with ASCII art logo
   - Exit code: 0

3. **Post-installation verification failed:**
   ```bash
   opencode --version
   bash: opencode: command not found
   ```

   ```bash
   which opencode
   (no output, exit code 1)
   ```

---

## Testing Results (After PATH Fix)

After sourcing `.bashrc` to fix the PATH issue, extensive testing revealed critical runtime errors:

### Working Commands:
1. **Version check:** `opencode --version` → `1.1.13` ✅
2. **Help command:** `opencode --help` → Displays full help ✅
3. **Models list:** `opencode models` → Lists available models ✅
   - opencode/big-pickle
   - opencode/glm-4.7-free
   - opencode/gpt-5-nano
   - opencode/grok-code
   - opencode/minimax-m2.1-free
4. **Debug paths:** `opencode debug paths` → Shows all paths ✅

### Failed Commands (All with same error):

1. **Stats command:**
   ```bash
   opencode stats
   ```
   ```
   Error: Unexpected error, check log file at /home/arnab/.local/share/opencode/log/2026-01-11T181150.log
   error: Cannot find module '/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth' from '/$bunfs/root/src/index.js'
   ```

2. **Run command:**
   ```bash
   opencode run "hello"
   ```
   ```
   Error: Cannot find module '/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth' from '/$bunfs/root/src/index.js'
   ```

3. **Debug config:**
   ```bash
   opencode debug config
   ```
   ```
   Error: Cannot find module '/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth' from '/$bunfs/root/src/index.js'
   ```

### Log Analysis:

The log file (`/home/arnab/.local/share/opencode/log/2026-01-11T181150.log`) shows:

1. Installation appears to bootstrap correctly
2. Plugin installation via bun succeeds:
   ```
   bun add @opencode-ai/plugin@1.1.13 --exact
   installed @opencode-ai/plugin@1.1.13
   ```
3. Plugin loading starts:
   ```
   INFO  service=plugin name=CodexAuthPlugin loading internal plugin
   INFO  service=plugin path=opencode-copilot-auth@0.0.11 loading plugin
   ```
4. **Critical error occurs:**
   ```
   ERROR  service=default name=ResolveMessage message=Cannot find module '/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth' from '/$bunfs/root/src/index.js' code=ERR_MODULE_NOT_FOUND
   ```

### Module Verification:

The module **does exist** in the expected location:
```bash
ls -la /home/arnab/.cache/opencode/node_modules/
# Shows: opencode-copilot-auth (directory exists and is accessible)
```

### Root Cause:

The error indicates a **Bun virtual filesystem (bunfs) path resolution issue**:
- The module exists at `/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth`
- Bun is trying to load it from a virtual path: `/$bunfs/root/src/index.js`
- The virtual filesystem is not correctly mapping to the actual filesystem paths

This appears to be a bug specific to **Linux 6.17.0-PRoot-Distro** or the **Bun runtime** on this platform.

---

## Root Cause Analysis

### Investigation Steps:

1. **Checked installation directory:**
   ```bash
   ls -la /home/arnab/.opencode/bin/
   -rwxr-xr-x 1 arnab arnab 137115052 Jan 11 07:09 opencode
   ```
   - **Result:** Binary exists and is executable (137 MB)

2. **Checked .bashrc modifications:**
   ```bash
   grep opencode ~/.bashrc
   # opencode
   export PATH=/home/arnab/.opencode/bin:$PATH
   ```
   - **Result:** PATH export was successfully added to ~/.bashrc

3. **Verified binary works with manual PATH update:**
   ```bash
   export PATH=/home/arnab/.opencode/bin:$PATH && opencode --version
   1.1.13
   ```
   - **Result:** Binary is functional when PATH is manually updated

### Identified Problem:

The installation script modifies `~/.bashrc` to add OpenCode to PATH, but does **not**:
1. Source the `.bashrc` file after modification
2. Update the current shell session's PATH environment variable
3. Inform the user that they need to restart their shell or run `source ~/.bashrc`

This creates a confusing user experience where the installation reports success, but the command is not immediately available.

---

## Additional Observations

1. **Missing .local/bin in PATH:** The default PATH does not include `/home/arnab/.local/bin/`, even though other binaries (like `droid` and `claude`) are installed there.

2. **No explicit instructions:** The installation output does not mention the need to source `.bashrc` or restart the shell.

3. **Silent failure:** Users may assume the installation failed because the command isn't accessible, when in fact it was installed successfully.

---

## Reproduction Steps

1. Run: `curl -fsSL https://opencode.ai/install | bash`
2. Observe success message
3. Attempt to run: `opencode --version`
4. Command not found error occurs

---

## Suggested Fixes

### Option 1: Source .bashrc after installation
```bash
# At the end of the install script
source ~/.bashrc
```

### Option 2: Update PATH for current session
```bash
# After adding to .bashrc, also set for current session
export PATH="/home/arnab/.opencode/bin:$PATH"
```

### Option 3: Add user notification
Add a clear message after installation:
```
OpenCode installed successfully!

Please run the following command to update your PATH:
  source ~/.bashrc

Or restart your terminal session.
```

### Option 4: Create symlink in existing PATH location
Instead of relying on PATH modification, create a symlink:
```bash
ln -s ~/.opencode/bin/opencode /usr/local/bin/opencode
# or
ln -s ~/.opencode/bin/opencode ~/.local/bin/opencode
```

---

## Workaround

Users can manually fix this by running:
```bash
source ~/.bashrc
```

Or by adding the following line to their current session:
```bash
export PATH=/home/arnab/.opencode/bin:$PATH
```

---

## Severity Assessment

**Severity:** **CRITICAL**
**Impact:** **OpenCode is completely non-functional** - While basic commands work, any command that requires loading plugins or running the core AI features fails with a module resolution error. The tool cannot be used for its primary purpose.
**Workaround Available:** No - The bunfs path resolution issue appears to be a platform-specific bug that cannot be worked around by users.
**Risk:** **HIGH** - Users will install OpenCode expecting it to work, but will encounter cryptic error messages when trying to use the actual AI features.

## Updated Issue Summary

1. **Initial Issue:** Installation script doesn't source `.bashrc`, making the command unavailable immediately (UX issue, easily fixable)

2. **Critical Issue:** After fixing the PATH, OpenCode fails to load any plugins or run core functionality due to a Bun virtual filesystem path resolution bug on Linux 6.17.0-PRoot-Distro

3. **Result:** OpenCode is effectively unusable on this platform, despite successful installation and some basic commands working

---

## Related Information

- Installation URL: https://opencode.ai/install
- Documentation: https://opencode.ai/docs/
- GitHub Repository: https://github.com/sst/opencode

---

## Additional Platform Notes

**PRoot Environment:**
This appears to be running inside a PRoot (pseudo-root) environment, which may have implications for Bun's virtual filesystem implementation. PRoot creates a user-space chroot-like environment that may not be fully compatible with Bun's `bunfs` virtual filesystem layer.

**Potential Bun Compatibility Issues:**
1. PRoot may not fully support the `memfd_create` system call that Bun uses for certain operations
2. Symbolic link handling in PRoot environments may differ from native Linux
3. Path resolution across the PRoot virtualization boundary may cause Bun's module loader to fail

**Suggested Investigation for OpenCode Team:**
1. Test OpenCode on standard Linux distributions (not PRoot-based)
2. Check Bun's compatibility with PRoot environments
3. Consider adding a fallback mechanism when bunfs path resolution fails
4. Add better error messages that indicate platform compatibility issues

**Alternative Installation Methods Attempted (Failed):**
1. `pip install opencode` - Package not found on PyPI
2. `brew install opencode` - Not applicable (not macOS)
3. `scoop install opencode` - Not applicable (not Windows)
4. `npm i -g opencode-ai@latest` - Not attempted due to pip failure

**Platform-Specific Commands for Debugging:**

To help diagnose this issue on similar systems, the OpenCode team should consider adding platform detection:
```bash
# Check for PRoot environment
[ -f /.dockerenv ] || [ -d /proc/self/root ] && echo "Running in container/PRoot"

# Check Bun version
bun --version

# Test Bun module loading
bun test  # If there's a test suite
```
