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

## Unified Output Format

`deps` provides a consolidated master list per target, categorized by type:

```text
main.py (Python script)
  [bin]           INSTALLED  hd-bet -> /home/lewis/Dev/mri/.venv/bin/hd-bet [via python package: HD_BET]
  [bin]           INSTALLED  mri_synthstrip -> /home/lewis/Dev/mri/scripts/mri_synthstrip
  [py]            INSTALLED  nibabel==5.3.3 -> /home/lewis/Dev/mri/.venv/lib/python3.12/site-packages (installed: 5.3.3)
  [py]            INSTALLED  numpy==2.4.2 -> /home/lewis/Dev/mri/.venv/lib/python3.12/site-packages (installed: 2.4.2)
  [py]            INSTALLED  SimpleITK==2.5.3 (OPTIONAL) -> ...
```

### Category Tags
- **`[bin]`**: System Commands / Executables
- **`[lib]`**: Shared Libraries (for compiled binaries)
- **`[py]`**: Python Modules
- **`[py-ext]`**: Embedded Python code (inside Bash wrappers)
- **`[js]`**: JavaScript / npm Packages

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
- `shutil`, `ldd` (standard on most Linux systems)
