---
description: Run Python scripts on Termux with dependency management
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Python on Termux

You are running Python on Termux (Android). Follow these guidelines:

## Environment Facts
- Python 3.12 is available at `/data/data/com.termux/files/usr/bin/python`
- Use `pip` for package management (uv does NOT work on Android/Termux)
- Virtual environments work normally with `python -m venv`

## Running Scripts

### Simple script (no dependencies)
```bash
python $ARGUMENTS
```

### Script with dependencies
1. First check if dependencies are installed: `pip list`
2. Install missing packages: `pip install <package>`
3. Run the script: `python $ARGUMENTS`

### If the script has inline dependencies (PEP 723)
Look for a comment block like:
```python
# /// script
# dependencies = ["requests", "rich"]
# ///
```
Install those packages manually before running:
```bash
pip install requests rich
python script.py
```

## Virtual Environments
If the user wants isolation:
```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python script.py
```

## Common Issues

### "Module not found"
Install the missing module: `pip install <module_name>`

### Permission errors
Never use `sudo` in Termux. Use `pip install --user <package>` if needed.

### Build failures
Some packages need build tools:
```bash
pkg install build-essential python-cython
```

## Quick Reference
| Task | Command |
|------|---------|
| Run script | `python script.py` |
| Install package | `pip install package` |
| List packages | `pip list` |
| Create venv | `python -m venv venv` |
| Activate venv | `source venv/bin/activate` |
