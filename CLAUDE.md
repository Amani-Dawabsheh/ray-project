# Ray Development Guide

## Quick References

- **Documentation**: [doc/source/ray-contribute/development.rst](doc/source/ray-contribute/development.rst) - Full build and development setup instructions.
- **Contributing Guide**: [CONTRIBUTING.rst](CONTRIBUTING.rst) - PR process and community channels.
- **CRITICAL**: After ANY Python code changes, run `pre-commit run --all-files` or at minimum `pre-commit run ruff black --all-files` to format and lint code.

## Project Structure

- **Python Core**: `python/ray/` - Main Ray Python package
- **C++ Core**: `src/ray/` - Core C++ implementation (raylet, GCS, object store)
- **RLlib**: `rllib/` - Reinforcement learning library
- **Documentation**: `doc/` - Sphinx documentation
- **Java Bindings**: `java/` - Java API
- **C++ API**: `cpp/` - C++ client library

### Ray Libraries

- `python/ray/data/` - Ray Data (scalable datasets)
- `python/ray/train/` - Ray Train (distributed training)
- `python/ray/tune/` - Ray Tune (hyperparameter tuning)
- `python/ray/serve/` - Ray Serve (model serving)
- `python/ray/workflow/` - Ray Workflows
- `python/ray/llm/` - LLM utilities
- `python/ray/dashboard/` - Ray Dashboard

## Environment Setup

### Python-Only Development (Recommended for most changes)

For editing RLlib, Tune, Autoscaler, and most Python files without compiling C++:

```bash
# 1. Create a virtual environment
conda create -c conda-forge python=3.9 -n ray-dev
conda activate ray-dev

# 2. Install latest Ray nightly wheel
pip install -U https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-3.0.0.dev0-cp39-cp39-manylinux2014_x86_64.whl

# 3. Link local Python files to installed package
python python/ray/setup-dev.py
```

### Full Build (C++ and Python)

```bash
# Install system dependencies (Ubuntu)
sudo apt-get update
sudo apt-get install -y build-essential curl clang-12 pkg-config psmisc unzip

# Install Bazel
ci/env/install-bazel.sh

# Install Node.js (for dashboard)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install 14 && nvm use 14

# Build dashboard
cd python/ray/dashboard/client
npm ci && npm run build
cd -

# Install Ray in editable mode
cd python/
pip install -r requirements.txt
pip install -e . --verbose
```

## Essential Commands

```bash
# Code quality - RUN AFTER EVERY EDIT
pre-commit run --all-files              # Run all linters and formatters
pre-commit run ruff black --all-files   # Quick: just ruff + black

# Individual linting tools
ci/lint/format.sh                       # Format Python, C++, shell scripts
ci/lint/lint.sh pre_commit              # Run pre-commit checks

# Testing
pytest python/ray/tests/test_basic.py  # Run specific test
pytest python/ray/tests/ -v            # Run test directory
pytest -x path/to/test.py              # Stop on first failure

# Build commands
pip install -e python/ --verbose       # Rebuild after C++ changes
bazel build //:ray_pkg                 # Build with Bazel directly

# Documentation (from doc/ directory)
pip install -r requirements-doc.txt    # Install doc dependencies
make develop                           # Build and serve docs locally
make linkcheck                         # Check for broken links
make doctest                           # Run doctests
```

## Testing

- **Framework**: pytest with 180s default timeout per test
- **Test locations**: `python/ray/tests/`, `rllib/tests/`, and `tests/` subdirectories within each module
- **Config**: [pytest.ini](pytest.ini)

```bash
# Run core tests
pytest python/ray/tests/

# Run RLlib tests
pytest rllib/tests/

# Run with specific marker
pytest -m "not slow" python/ray/tests/

# Run single test function
pytest python/ray/tests/test_basic.py::test_function_name -v
```

## Code Style

### Python
- **Formatter**: Black (v22.10.0)
- **Linter**: Ruff (replaces flake8, isort)
- **Type checker**: mypy (v1.7.0)
- **Docstring style**: Google style (pydoclint enforced)
- **Max line length**: 88 characters

### C++
- **Formatter**: clang-format (v12.0.1)
- **Linter**: cpplint
- **Build files**: Bazel with buildifier formatting

### Pre-commit Hooks

Install and run pre-commit:

```bash
pip install pre-commit
pre-commit install              # Set up git hooks
pre-commit run --all-files      # Run all checks manually
```

Key hooks: `ruff`, `black`, `mypy`, `clang-format`, `cpplint`, `buildifier`, `shellcheck`

## Code Quality Requirements

- **MANDATORY**: Run linting before committing: `pre-commit run --all-files`
- **MANDATORY**: All tests must pass before PR merge
- **Type hints**: Add type annotations for new code
- **Docstrings**: Follow Google style for public APIs

## Build Environment Variables

```bash
RAY_DEBUG_BUILD=debug          # Build with debug symbols
RAY_DEBUG_BUILD=asan           # Build with AddressSanitizer
RAY_DEBUG_BUILD=tsan           # Build with ThreadSanitizer
RAY_DISABLE_EXTRA_CPP=1        # Skip C++ extras (faster build)
SKIP_BAZEL_BUILD=1             # Skip Bazel build entirely
RAY_INSTALL_JAVA=1             # Include Java bindings
```

## Code Searching

- **DO NOT** search in `bazel-*` directories - these are build outputs
- **DO NOT** search in `python/ray/core/generated/` - auto-generated protobuf code
- Search for dependencies in `python/setup.py` and `python/requirements.txt`

## PR Process

1. Fork the repository and create a feature branch
2. Make changes and ensure all tests pass
3. Run `pre-commit run --all-files`
4. Submit PR with clear description
5. Address reviewer feedback (remove `@author-action-required` label when done)
6. Add `test-ok` label once CI passes

## Common Development Tasks

### Adding a new Python dependency

1. Add to `python/setup.py` in the appropriate extras section
2. Add to `python/requirements.txt` if needed for development
3. Run `pip install -e python/[extra_name]` to test

### Modifying protobuf definitions

1. Edit `.proto` files in `src/ray/protobuf/`
2. Rebuild with `bazel build //:ray_pkg` or `pip install -e python/`
3. Generated Python bindings appear in `python/ray/core/generated/`

### Dashboard development

```bash
cd python/ray/dashboard/client
npm install
npm start                      # Development server
npm run build                  # Production build
```

## Useful Links

- **Documentation**: https://docs.ray.io
- **GitHub Issues**: https://github.com/ray-project/ray/issues
- **Discourse Forum**: https://discuss.ray.io
- **Slack**: https://www.ray.io/join-slack
