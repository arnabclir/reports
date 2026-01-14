# Python in Termux

A practical guide for using Python in the Termux Android terminal environment.

**Last Updated:** 2025-01-14
**Tested Environment:** Termux on Android (Linux 6.1.118-android14)
**Python Version:** 3.12.12

---

## 🧪 Test Results & Findings

### ✅ What Works

**Python 3.12.12** is pre-installed and fully functional:
- Location: `/data/data/com.termux/files/usr/bin/python3.12`
- Symlinks: `python` → `python3.12`, `python3` → `python3.12`
- All standard Python features work normally

**Simple test script results:**
```bash
$ python test_hello.py
Hello from Python!
Python version: 3.12.12 (main, Oct 18 2025, 05:45:20) [Clang 19.0.1]
```

### ❌ Critical Limitation: uv Does NOT Work on Termux

**uv** (the fast Python package manager) **cannot be installed on Termux/Android**:

```
ERROR: there isn't a download for your platform aarch64-linux-android
```

The official uv installer does not support Android's `aarch64-linux-android` platform architecture.

**Workarounds that DON'T work:**
- ❌ Official curl installer script (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- ❌ `pip install uv` (build failures, no compatible wheels)

**Status:** This is a known limitation. See these GitHub issues:
- [Installing uv in Termux: Linux-aarch64 #2705](https://github.com/astral-sh/uv/issues/2705)
- [Termux on Android #2408](https://github.com/astral-sh/uv/issues/2408)
- [uv not detecting python in termux #7373](https://github.com/astral-sh/uv/issues/7373)

---

## Installing Python

### Check if Python is Installed

```bash
python --version
# or
python3 --version
```

### Install Python (if not installed)

```bash
pkg update
pkg install python
```

Verify installation:
```bash
python --version
which python  # Should show: /data/data/com.termux/files/usr/bin/python
```

---

## Managing Python Packages

Since uv doesn't work, use **pip** (the standard Python package manager) or **Termux packages**.

### Method 1: Using pip (Recommended for Python packages)

**Install a package:**
```bash
pip install requests
pip install rich
pip install numpy
```

**Install with version specifier:**
```bash
pip install "requests>=2.28.0"
pip install "django==4.2"
```

**Install from requirements file:**
```bash
pip install -r requirements.txt
```

**List installed packages:**
```bash
pip list
```

**Upgrade a package:**
```bash
pip install --upgrade requests
```

**Uninstall a package:**
```bash
pip uninstall requests
```

### Method 2: Using Termux Packages (For system-level packages)

Some Python packages are available through Termux's package manager:

```bash
# Search for Python packages
pkg search python

# Install common packages
pkg install python-numpy
pkg install python-pandas
pkg install python-matplotlib
```

### Virtual Environments (Recommended for Projects)

**Create a virtual environment:**
```bash
python -m venv myproject
cd myproject
source bin/activate
```

**Install packages in the virtual environment:**
```bash
pip install requests rich
```

**Deactivate when done:**
```bash
deactivate
```

---

## Running Python Scripts

### Basic Execution

```bash
python script.py
python3 script.py

# With arguments
python script.py arg1 arg2
```

### Executable Scripts (Shebang)

Add this as the first line in your script:
```python
#!/usr/bin/env python
```

Make it executable:
```bash
chmod +x script.py
./script.py
```

---

## Managing Dependencies

### requirements.txt (Standard Approach)

Create a `requirements.txt` file:
```
requests>=2.28.0
rich
numpy
pandas
```

**Install from requirements.txt:**
```bash
pip install -r requirements.txt
```

**Generate requirements.txt from current environment:**
```bash
pip freeze > requirements.txt
```

### pyproject.toml (Modern Approach)

Create a `pyproject.toml` file:
```toml
[project]
name = "myproject"
version = "0.1.0"
dependencies = [
    "requests>=2.28.0",
    "rich",
]

[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

**Install in editable mode:**
```bash
pip install -e .
```

---

## Inline Dependencies (PEP 723)

Python 3.11+ supports inline dependencies. Add this to the top of your script:

```python
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "requests",
#   "rich",
# ]
# ///

import requests
from rich import print

print("Hello with dependencies!")
```

**Note:** Unlike uv, pip won't automatically install these. You'll need to install them manually:
```bash
pip install requests rich
python script.py
```

---

## Useful Python Tools

### Jupyter Notebook

```bash
pip install jupyter
jupyter notebook
```

### HTTP Requests Testing

```bash
pip install httpie
http GET https://api.github.com/users/python
```

### Code Formatting

```bash
pip install black
black script.py
```

### Linting

```bash
pip install pylint
pylint script.py
```

---

## Troubleshooting

### pip Not Found

```bash
pkg install python-pip
```

### Module Not Found Errors

1. Check if the package is installed:
   ```bash
   pip list | grep package_name
   ```

2. Install if missing:
   ```bash
   pip install package_name
   ```

3. Check which Python you're using:
   ```bash
   which python
   python -m pip list  # Use this pip to be sure
   ```

### Permission Errors

Never use `sudo` in Termux. If you get permission errors:

```bash
# Use pip with --user flag
pip install --user package_name

# Or fix permissions
termux-setup-storage
```

### Package Build Failures

Some packages require build dependencies:

```bash
pkg install python-cython
pkg install build-essential
```

---

## Quick Reference Commands

| Task | Command |
|------|---------|
| Check Python version | `python --version` |
| Run script | `python script.py` |
| Install package | `pip install package` |
| List packages | `pip list` |
| Create venv | `python -m venv myenv` |
| Activate venv | `source bin/activate` |
| Freeze requirements | `pip freeze > requirements.txt` |
| Install from requirements | `pip install -r requirements.txt` |
| Upgrade pip | `pip install --upgrade pip` |

---

## Environment Variables

Useful environment variables to add to `~/.bashrc`:

```bash
# Python optimizations
export PYTHONOPTIMIZE=2

# Don't write .pyc files
export PYTHONDONTWRITEBYTECODE=1

# pip user install location
export PYTHONUSERBASE="$HOME/.local"
export PATH="$PYTHONUSERBASE/bin:$PATH"
```

Add to `~/.bashrc`:
```bash
echo 'export PYTHONUSERBASE="$HOME/.local"' >> ~/.bashrc
echo 'export PATH="$PYTHONUSERBASE/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

---

## Resources

- Official Python documentation: https://docs.python.org/3/
- pip documentation: https://pip.pypa.io/
- Termux Wiki (Python): https://wiki.termux.com/wiki/Python
- uv GitHub Issues (Termux discussions):
  - [Issue #2705 - Installing uv in Termux](https://github.com/astral-sh/uv/issues/2705)
  - [Issue #2408 - Termux on Android](https://github.com/astral-sh/uv/issues/2408)
  - [Issue #7373 - uv not detecting python](https://github.com/astral-sh/uv/issues/7373)

---

## Summary

**✅ Works Great:**
- Python 3.12.12 (pre-installed)
- pip package manager
- Virtual environments
- All standard Python packages

**❌ Doesn't Work:**
- uv package manager (no Android/Termux support)

**🎯 Recommendation:**
Use the traditional Python workflow with pip and virtual environments. It's reliable, well-documented, and fully supported on Termux.
