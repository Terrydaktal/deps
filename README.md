# deps - Multi-Language Dependency Mapper

`deps` is a robust, environment-aware dependency scanner designed to map the full system and language-level footprint of Linux scripts and binaries. It goes beyond simple name listing by verifying installations, resolving environments, and deeply inspecting source code across Bash, Python, and JavaScript.

## Features

### 🌍 Environment Awareness
- **Virtual Environment Detection:** Automatically identifies and uses local `.venv` or `venv` directories.
- **Wrapper Support:** Resolves `VENV_PY` style variables in shell wrappers to check dependencies against the *correct* Python environment.
- **System Fallback:** If a dependency isn't in your virtual environment, it automatically checks user-local and system-wide site-packages.

### 🐍 Python Deep Inspection
- **Source Analysis:** Uses AST (Abstract Syntax Tree) to find imports.
- **Optional Dependency Detection:** Identifies imports wrapped in `try...except` blocks and marks them as `(OPTIONAL)`, excluding them from fatal error counts.
- **Embedded Code:** Extracts and scans Python source code from Bash heredocs and `python -c` strings.
- **Transitive Filtering:** Ignores internal library noise by excluding virtual environment folders from the scan.

### 📦 JavaScript (Node.js) Support
- **Recursive Scanning:** Follows `require()`, `import`, and dynamic `import()` statements through your project.
- **Version Verification:** Compares required versions in `package.json` against the actually installed versions in `node_modules`.
- **Symlink Resolution:** Correctly finds project roots even when scripts are symlinked into global `bin` folders.

### 🐚 System & Shell Robustness
- **Dynamic Binaries:** Uses `ldd` to map shared library dependencies for compiled executables.
- **Resilient Shell Parsing:**
    - Handles complex shell array assignments (e.g., `CMD=(rg --json)`).
    - Recognizes and skips internal script functions.
    - Awareness of multi-line quotes (e.g., long `awk` blocks).
    - Massive ignore list for common shell/awk keywords to minimize false positives.

## Usage

```bash
# Scan a global binary or script
./deps uv
./deps chatbot

# Scan a local file
./deps ./my_script.py

# Specify a project directory for context
./deps --project ~/Dev/my_project crawler.py
```

## Output Example

```text
System dependencies for /home/lewis/.local/bin/chatbot
  INSTALLED /bin/chromium -> /usr/bin/chromium
  INSTALLED bash -> /bin/bash
  ...

JavaScript dependencies for /home/lewis/Dev/chatbot
  INSTALLED chalk (^4.1.2) -> .../node_modules/chalk (installed: 4.1.2)
  INSTALLED commander (^14.0.2) -> .../node_modules/commander (installed: 14.0.2)

Environment Consistency Check
  OK uv pip check: No broken requirements found.

Summary
  System deps checked: 22, missing: 0
  Python deps checked: 0, missing: 0
  JavaScript deps checked: 9, missing: 0
```

## How it Works

`deps` uses a hybrid approach to ensure accuracy:
1.  **Phase 1: Binary/Shebang Resolution:** Identifies if the target is a binary or a script and resolves its real path.
2.  **Phase 2: Static Analysis:** Parses the source code (or binary headers) to find external calls and imports.
3.  **Phase 3: Environment Discovery:** Finds the relevant Python/Node.js environment (venv, node_modules).
4.  **Phase 4: Truth-Based Verification:** Queries the actual filesystem and package managers to confirm the dependency is `INSTALLED` and functional.

## Installation

```bash
git clone https://github.com/Terrydaktal/deps.git
cd deps
chmod +x deps
# Move to your PATH if desired
mv deps ~/.local/bin/
```

## Requirements
- Python 3.10+
- `pipreqs` (for Python scanning)
- `shutil`, `ldd`, `nm` (standard on most Linux systems)
