# Creating Virtual Environments

Virtual environments are isolated Python environments. They allow you to define specific dependencies for a project without polluting the system-wide Python installation.

## The `venv` Module
Since Python 3.3, `venv` is the standard tool included with Python.

### Creation
Run this in your project root:

```bash
# syntax: python -m venv <directory_name>
python3 -m venv venv
```
This creates a folder named `venv` containing a copy/symlink of the Python binary and a `site-packages` folder.

### Activation

**Linux/macOS**:
```bash
source venv/bin/activate
```

**Windows (PowerShell)**:
```powershell
.\venv\Scripts\Activate.ps1
```

Once activated, your terminal prompt usually changes, and `python` refers to the isolated version.

### Deactivation
```bash
deactivate
```

## `.gitignore`
**Crucial**: Never commit your `venv` directory to Git. It is specific to your machine.
Add `venv/` to your `.gitignore`.

## Alternative: `virtualenv`
`virtualenv` is the older, third-party predecessor to `venv`. It is still useful if you need to create environments for *older* Python versions than the one you are running.

```bash
pip install virtualenv
virtualenv -p python3.8 myenv
```
