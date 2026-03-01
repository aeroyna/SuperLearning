# pip

`pip` is the standard package installer for Python.

## Basic Usage

```bash
# Install
pip install requests

# specific version
pip install "requests==2.28.1"

# Upgrade
pip install --upgrade requests

# Uninstall
pip uninstall requests
```

## Requirements Files
For reproducible environments, use `requirements.txt`.

```bash
# Freeze current env to file
pip freeze > requirements.txt

# Install from file
pip install -r requirements.txt
```

### Limitations of `requirements.txt`
*   It is a flat list. It doesn't distinguish between direct dependencies (what you added) and transitive dependencies (what they typically need).
*   Upgrading is hard (you have to manually check versions).

## pip-tools
A popular middle-ground.
1.  Define `requirements.in` (Direct deps).
2.  Run `pip-compile` to generate `requirements.txt` (Locked transitive deps).
