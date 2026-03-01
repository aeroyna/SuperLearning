# Publishing to PyPI

Once your package is structured and configured, sharing it with the world via PyPI (Python Package Index) is straightforward.

## 1. Build

Install `build`, the modern frontend for building packages.

```bash
pip install build
python -m build
```
This generates a `dist/` directory containing:
*   `.tar.gz` (Source Archive)
*   `.whl` (Wheel - Pre-built binary distribution)

## 2. Validator
Check your distribution for errors using `twine`.

```bash
pip install twine
twine check dist/*
```

## 3. Upload

To upload, you just run twine.

### TestPyPI (Highly Recommended)
First, upload to the test index to verify everything looks right.

```bash
twine upload --repository testpypi dist/*
```
You can install from there to test:
`pip install --index-url https://test.pypi.org/simple/ --no-deps my-package`

### Production PyPI
```bash
twine upload dist/*
```
You will be prompted for API token (setup one in your PyPI account settings).

## CI/CD (GitHub Actions)
Don't upload manually. Automate it on release creation.

```yaml
name: Publish Python 🐍 distribution to PyPI

on:
  release:
    types: [published]

jobs:
  build-n-publish:
    name: Build and publish Python 🐍 distribution to PyPI
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: "3.x"
    - name: Install pypa/build
      run: python -m pip install build --user
    - name: Build a binary wheel and a source tarball
      run: python -m build
    - name: Publish distribution 📦 to PyPI
      uses: pypa/gh-action-pypi-publish@release/v1
      with:
        password: ${{ secrets.PYPI_API_TOKEN }}
```
