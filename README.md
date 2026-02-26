# deps - Multi-Language Dependency Mapper

`deps` is a robust, environment-aware dependency scanner designed to map the full system and language-level footprint of Linux scripts and binaries. It goes beyond simple name listing by verifying installations, resolving environments, and deeply inspecting source code across Bash, Python, and JavaScript.

## Features

### 🌍 Universe Discovery
- **Recursive Script Following:** Automatically follows references from one script to another (e.g., a Bash script calling a JS file).
- **Per-File Reporting:** Generates a distinct dependency list for every unique file discovered in the project "universe."
- **Multi-Language Support:** Seamlessly transitions between Bash, Python, and JavaScript while following dependencies.

### 🌍 Environment Awareness
- **Virtual Environment Detection:** Automatically identifies and uses local `.venv` or `venv` directories.
- **Wrapper Support:** Resolves `VENV_PY` style variables in shell wrappers to check dependencies against the *correct* Python environment.
- **System Fallback:** If a dependency isn't in your virtual environment, it automatically checks user-local and system-wide site-packages.

### 🐍 Python Deep Inspection
- **Source Analysis:** Uses AST (Abstract Syntax Tree) to find imports and system calls.
- **Optional Dependency Detection:** Identifies imports wrapped in `try...except` blocks and marks them as `(OPTIONAL)`, excluding them from fatal error counts.
- **Argparse & Pathlib Support:** Scans `argparse` defaults and understands `pathlib.Path` joins to find external tool requirements.
- **Embedded Code:** Extracts and scans Python source code from Bash heredocs and `python -c` strings.
- **Transitive Filtering:** Ignores internal library noise by excluding virtual environment folders from the scan.

### 📦 JavaScript (Node.js) Support
- **Recursive Scanning:** Follows `require()`, `import`, and dynamic `import()` statements through your project.
- **Version Verification:** Compares required versions in `package.json` against the actually installed versions in `node_modules`.
- **Symlink Resolution:** Correctly finds project roots even when scripts are symlinked into global `bin` folders.

### 🐚 System & Shell Robustness
- **Dynamic Binaries:** Uses `ldd` to map shared library dependencies for compiled executables.
- **Existence-Aware Filtering:** Differentiates between structural shell keywords (always ignored) and potential commands (like `commit` or `ninja`). Potential commands are only reported if they actually exist on your system, eliminating noise from git arguments or variable names.
- **Resilient Shell Parsing:** Handles complex shell array assignments, recognizes script-internal functions, and understands multi-line quotes (e.g. `awk` blocks).

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

## Unified Master List Format

`deps` provides a consolidated, dynamically aligned report per target:

```text
chatbot (Bash script)
  [bin] INSTALLED  /bin/chromium -> /bin/chromium
  [bin] INSTALLED  /home/lewis/Dev/chatbot/chatbot.js -> /home/lewis/Dev/chatbot/chatbot.js
  [bin] INSTALLED  bash -> /bin/bash
  ...

chatbot.js (JavaScript script)
  [bin] INSTALLED  node -> /bin/node
  [js]  INSTALLED  chalk (^4.1.2) -> .../node_modules/chalk (installed: 4.1.2)
  [js]  INSTALLED  commander (^14.0.2) -> .../node_modules/commander (installed: 14.0.2)
```

### Category Tags
- **`[bin]`**: System Commands / Executables (found in scripts)
- **`[lib]`**: Shared Libraries (found in compiled binaries)
- **`[py]`**: Python Modules
- **`[py-emb]`**: Embedded Python code (inside Bash wrappers)
- **`[js]`**: JavaScript / npm Packages

## How it Works

`deps` uses a hybrid approach to ensure accuracy:
1.  **Phase 1: Binary/Shebang Resolution:** Identifies if the target is a binary or a script and resolves its real path.
2.  **Phase 2: Discovery Loop:** Recursively follows every script reference found to build a complete project "universe."
3.  **Phase 3: Static Analysis:** Parses source code (AST) or binary headers (`ldd`) to find external calls and imports.
4.  **Phase 4: Truth-Based Verification:** Queries the actual filesystem and package managers to confirm the dependency is `INSTALLED` and functional in the specific detected environment.

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
- `shutil`, `ldd` (standard on most Linux systems)
