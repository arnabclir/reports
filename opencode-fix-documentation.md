# OpenCode Installation Fixes

**Date:** 2026-01-11
**Issue Version:** OpenCode 1.1.13
**Platform:** Linux 6.17.0-PRoot-Distro

---

## Issue Summary

Two critical issues prevent OpenCode from functioning on this platform:

1. **UX Issue:** Installation script doesn't update current shell session PATH
2. **Critical Bug:** Bun's virtual filesystem (bunfs) path resolution fails in PRoot environments

---

## Issue 1: PATH Not Updated After Installation (UX Issue)

### Problem
The installation script (`curl -fsSL https://opencode.ai/install | bash`) modifies `~/.bashrc` to add OpenCode to PATH, but does not:
- Source the `.bashrc` file after modification
- Update the current shell session's PATH variable
- Inform the user that a restart is needed

### Fix Implementation

**Option A: Update PATH for Current Session (Recommended)**

Add to the installation script after PATH modification:

```bash
# After adding to .bashrc, also set for current session
export PATH="$HOME/.opencode/bin:$PATH"

# Verify it works immediately
if command -v opencode &> /dev/null; then
    echo "OpenCode is ready to use!"
else
    echo "Warning: opencode not found in PATH"
fi
```

**Option B: Source .bashrc After Installation**

```bash
# At the end of the install script
source ~/.bashrc

# Verify
opencode --version
```

**Option C: Add User Notification**

```bash
cat << 'EOF'

✓ OpenCode installed successfully!

To start using OpenCode, run:
  source ~/.bashrc

Or restart your terminal session.

EOF
```

### Workaround for Users

Run immediately after installation:
```bash
source ~/.bashrc
```

Or manually update PATH for current session:
```bash
export PATH=$HOME/.opencode/bin:$PATH
```

---

## Issue 2: Bun bunfs Path Resolution Failure (Critical)

### Problem

After fixing PATH, OpenCode fails with:
```
Error: Cannot find module '/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth' from '/$bunfs/root/src/index.js'
```

**Root Cause:**
- Module exists at actual path: `/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth`
- Bun tries to load from virtual path: `/$bunfs/root/src/index.js`
- Bun's virtual filesystem (bunfs) cannot resolve paths correctly in PRoot environment
- PRoot creates a user-space chroot-like environment incompatible with bunfs

### Fix Implementation

**Fix 1: Use Node.js Runtime Instead of Bun (Recommended)**

Modify OpenCode to detect PRoot/container environments and fallback to Node.js:

```javascript
// Detect PRoot/container environment
const isPRootOrContainer = () => {
  return process.env.PROOT === '1' ||
         process.env.IN_CONTAINER === '1' ||
         require('fs').existsSync('/.dockerenv') ||
         require('fs').existsSync('/proc/self/root');
};

// Use Node.js instead of Bun in PRoot
const runtime = isPRootOrContainer() ? 'node' : 'bun';

if (runtime === 'node') {
  // Load modules using Node's require() instead of Bun's bunfs
  const module = require('/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth');
}
```

**Fix 2: Disable Bun Virtual Filesystem**

Add environment variable to disable bunfs:

```bash
# Before running opencode
export BUN_RUNTIME_TRANSPILER_PATH=0
opencode run "hello"
```

Or modify OpenCode binary to skip bunfs:

```javascript
// In OpenCode's startup code
process.env.BUN_DISABLE_BUNFS = '1';
```

**Fix 3: Use Absolute Paths Instead of Bun Virtual Paths**

Modify module loading to use absolute filesystem paths:

```javascript
// Instead of Bun's virtual path resolution
// const module = await import('opencode-copilot-auth');

// Use Node's require with absolute path
const module = require(`${process.env.HOME}/.cache/opencode/node_modules/opencode-copilot-auth`);
```

**Fix 4: Add Platform Compatibility Check**

Add platform detection at installation time:

```bash
# Check for PRoot environment
if [ -f /.dockerenv ] || [ -d /proc/self/root ]; then
    echo "Warning: OpenCode may not work properly in PRoot/container environments."
    echo "Consider using a standard Linux distribution."
    echo ""
    echo "Press Enter to continue anyway or Ctrl+C to cancel."
    read
fi
```

### Workaround for Users

**No User Workaround Available**

The bunfs path resolution is a platform compatibility issue that cannot be fixed by users. The OpenCode team must update their code to support PRoot environments or provide a Node.js-based fallback.

---

## Recommended Action Plan for OpenCode Team

### Immediate Actions (Priority 1)

1. **Fix PATH Issue (Easy - 5 minutes)**
   - Add `export PATH="$HOME/.opencode/bin:$PATH"` to installation script
   - Add success message with verification
   - Add user notification if verification fails

2. **Add Platform Detection (Medium - 30 minutes)**
   - Detect PRoot/container environments
   - Show warning before installation
   - Add documentation about platform compatibility

### Short-term Actions (Priority 2)

3. **Add Node.js Fallback (Hard - 2-4 hours)**
   - Detect when Bun fails to load modules
   - Fallback to Node.js runtime
   - Test on multiple platforms including PRoot

4. **Improve Error Messages (Easy - 1 hour)**
   - Replace cryptic bunfs error with clear message
   - "OpenCode is not compatible with PRoot environments"
   - Link to platform compatibility documentation

### Long-term Actions (Priority 3)

5. **Test on Multiple Platforms**
   - Add CI tests for:
     - Standard Linux distributions
     - Docker containers
     - PRoot environments
     - Termux (Android)
     - WSL (Windows Subsystem for Linux)

6. **Consider Alternative Runtime**
   - Evaluate if Node.js is better cross-platform
   - Consider providing multiple builds (bun + node)
   - Document platform-specific limitations

---

## Testing Checklist for Fixes

### PATH Fix Testing

- [ ] Install from fresh shell session
- [ ] Run `opencode --version` immediately after installation (should work)
- [ ] Run `opencode --help` (should work)
- [ ] Check PATH is updated in current session
- [ ] Verify `.bashrc` is modified correctly
- [ ] Test on new shell session (should still work)

### Platform Compatibility Testing

- [ ] Test on standard Linux (Ubuntu/Debian)
- [ ] Test on PRoot environment
- [ ] Test in Docker container
- [ ] Test in Termux (Android)
- [ ] Test in WSL (Windows)
- [ ] Test with `opencode run` command
- [ ] Test with `opencode debug config` command
- [ ] Test with `opencode stats` command

---

## Alternative Installation Approach

If the OpenCode team cannot fix the bunfs issue quickly, consider:

**Provide npm/npx Installation:**

```bash
npm install -g @opencode-ai/cli
# or
npx @opencode-ai/cli run "hello"
```

This avoids the Bun runtime issue by using Node.js directly.

**Provide Docker Container:**

```bash
docker run -it --rm opencodeai/opencode:latest
```

Ensures consistent runtime environment.

---

## Platform-Specific Notes

### PRoot Environment (Termux, proot-distro)

**Issues:**
- PRoot creates user-space virtualization layer
- Bun's bunfs expects native filesystem
- Path resolution fails at PRoot boundary
- Some system calls (`memfd_create`) may not be supported

**Recommended Fix:**
- Use Node.js runtime instead of Bun
- Avoid virtual filesystem features
- Use absolute paths for all module loading

### Standard Linux Distributions

**Status:** Should work with current implementation

**Testing Required:**
- Ubuntu 20.04/22.04/24.04
- Debian 11/12
- Fedora 38/39
- Arch Linux

---

## Summary

| Issue | Severity | Workaround | Fix Required |
|-------|----------|------------|--------------|
| PATH not updated | Low | `source ~/.bashrc` | Add `export PATH` to install script |
| bunfs path resolution | Critical | None | Add Node.js fallback or disable bunfs |

**Recommendation:** Fix the PATH issue immediately (easy win), then work on platform compatibility for Bun/PRoot environments. Consider providing Node.js-based installation as fallback.
