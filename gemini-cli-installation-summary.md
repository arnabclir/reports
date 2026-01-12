# Gemini CLI Installation Summary

## Date
2026-01-11

## Attempted Installation
Goal: Install Google Gemini CLI (@google/gemini-cli)

## Installation Method
The standard installation method for Gemini CLI is:
```bash
npm install -g @google/gemini-cli
```

## Issues Encountered

### 1. Native Module Build Failure
The installation failed when attempting to build the `tree-sitter-bash` native module via node-gyp. The error occurred due to network timeout when downloading Node.js headers:

```
npm error gyp ERR! configure error 
npm error gyp ERR! stack Error: read ETIMEDOUT
npm error gyp ERR! stack at TLSWrap.onStreamRead (node:internal/stream_base_commons:216:20)
```

### 2. Network Connectivity Issues
- Multiple npm install attempts timed out even with increased timeouts (up to 360 seconds)
- Tests showed slow connectivity to npmjs.org (connection took 3+ seconds for headers)
- Alternative registries were tried but also experienced timeouts

### 3. Installation Attempts Made
1. `npm install -g @google/gemini-cli` - Timed out on native module build
2. `npm install -g @google/gemini-cli --ignore-scripts` - Timed out on download
3. `npx @google/gemini-cli` - Timed out during package download
4. Using npm mirror registry (npmmirror.com) - Timed out
5. Downloaded tarball manually and attempted local install - Timed out

## Environment Information
- OS: Linux 6.17.0-PRoot-Distro (Termux)
- Node.js: v25.2.1
- npm: v11.6.2
- Python: 3.13.7
- curl: 8.18.0 (available)
- git: Not installed
- npm: Available

## Alternative Approaches Suggested
1. **npx method**: `npx @google/gemini-cli` - Avoids global install but still requires network access
2. **Docker**: `docker run --rm -it us-docker.pkg.dev/gemini-code-dev/gemini-cli/sandbox:0.1.1`
3. **Build from source**: Clone GitHub repository and build locally (requires git)

## Current Status
**Installation failed** due to persistent network timeouts during dependency downloads and native module compilation.

## Files Downloaded
- `/home/arnab/package/` - Extracted gemini-cli-0.23.0.tgz
- `/home/arnab/gemini-cli.tgz` - Original tarball (1.6MB)

## Next Steps to Try When Network is Stable
1. Retry `npm install -g @google/gemini-cli` when network conditions improve
2. Try installing from a different network location
3. Configure npm to use a local mirror or proxy if available
4. Install dependencies separately to troubleshoot specific problematic packages
