# Issue Reports

This repository contains detailed reports on software installation and compatibility issues encountered on **Linux 6.17.0-PRoot-Distro** (Termux environment).

## Reports

### 1. Google Gemini CLI Installation Failure
**File:** [`gemini-cli-installation-summary.md`](./gemini-cli-installation-summary.md)

**Summary:** Installation of `@google/gemini-cli` failed due to persistent network timeouts when downloading dependencies and compiling native modules via node-gyp.

**Key Issues:**
- Native module `tree-sitter-bash` build failed with ETIMEDOUT errors
- Multiple installation methods attempted (npm global, npx, local tarball, mirror registries)
- All attempts failed due to network connectivity issues with npmjs.org

**Status:** Installation failed - Requires stable network connection to retry

**Platform:** Linux 6.17.0-PRoot-Distro (Termux), Node.js v25.2.1, npm v11.6.2

---

### 2. OpenCode CLI Critical Runtime Failure
**Files:**
- [`opencode-installation-issue-report.md`](./opencode-installation-issue-report.md)
- [`opencode-fix-documentation.md`](./opencode-fix-documentation.md)

**Summary:** OpenCode v1.1.13 installation appears successful but the tool is completely non-functional due to Bun runtime virtual filesystem (bunfs) incompatibility with PRoot environments.

**Key Issues:**
1. **UX Issue:** Installation script doesn't update current shell PATH (easily worked around with `source ~/.bashrc`)
2. **Critical Bug:** Bun's `bunfs` cannot resolve module paths in PRoot, causing all core commands to fail

**Error:**
```
Error: Cannot find module '/home/arnab/.cache/opencode/node_modules/opencode-copilot-auth'
from '/$bunfs/root/src/index.js'
```

**Working Commands:** `--version`, `--help`, `models`, `debug paths`
**Broken Commands:** `run`, `stats`, `debug config` (anything that loads plugins)

**Severity:** CRITICAL - Tool is completely unusable on this platform

**Root Cause:** Bun's virtual filesystem (bunfs) is incompatible with PRoot's user-space virtualization layer

**Recommended Fixes:**
1. Detect PRoot/container environments and use Node.js runtime instead of Bun
2. Disable bunfs via `BUN_DISABLE_BUNFS=1` environment variable
3. Use absolute filesystem paths instead of Bun virtual paths
4. Add platform compatibility warnings at install time

**Status:** No user workaround available - Requires OpenCode team to implement PRoot compatibility

---

## Platform Information

All issues encountered on:
- **OS:** Linux 6.17.0-PRoot-Distro
- **Environment:** Termux (Android Linux environment)
- **Shell:** bash
- **Python:** 3.13.7

## Related Links

- [Gemini CLI](https://github.com/google/gemini-cli)
- [OpenCode](https://opencode.ai) | [GitHub](https://github.com/sst/opencode)
