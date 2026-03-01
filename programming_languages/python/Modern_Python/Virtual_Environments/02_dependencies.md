# Managing Dependencies

Once inside a virtual environment, you manage packages using `pip`.

## The "Global" Problem
Without virtual environments, `pip install requests` installs to your system Python.
1.  **Conflicts**: Project A needs `requests==1.0`, Project B needs `requests==2.0`. You can't have both.
2.  **Permissions**: You might need `sudo` (security risk).
3.  **Pollution**: Hard to tell what packages a specific project actually needs.

## Workflow

1.  **Create & Activate**: `python -m venv venv && source venv/bin/activate`
2.  **Install**: `pip install flask`
3.  **Verify**: `which python` should point to `.../venv/bin/python`.

## Checking Installed Packages

```bash
pip list
```

## Reproducibility
To ensure others can run your code, you must list your dependencies.

### `requirements.txt`
The standard format.

```bash
# Save current state
pip freeze > requirements.txt

# Install from state
pip install -r requirements.txt
```

### Modern Tools
Tools like **Poetry** and **Pipenv** manage virtual environments *and* dependencies automatically, abstracting the manual `venv` creation steps away. See [Dependency Management](../Dependency_Management/01_pip.md).
