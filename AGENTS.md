# Repository guide

## Project and layout

`jev` is a Python package with a Rust extension built by Maturin and PyO3.
Python requires 3.12 or newer; `.python-version` selects 3.14. Rust uses edition
2024. FastAPI and Uvicorn are dependencies, but no ASGI app or routes exist yet.

- `src/jev/__init__.py`: Python entry point; `jev` calls `jev.main()`.
- `src/lib.rs`: Rust implementation of the `jev._core` extension, currently
  exposing `hello_from_bin()`.
- `src/jev/_core.pyi`: Python type declarations for the extension.
- `pyproject.toml`, `uv.lock`: Python packaging, dependencies, and development tools.
- `Cargo.toml`, `Cargo.lock`: Rust crate configuration and dependencies.
- `tests/`: currently contains only `.gitkeep`; no test suite exists yet.
- `docs/`: Mintlify documentation. Read `docs/AGENTS.md` before editing it.
- `.pre-commit-config.yaml`: whitespace, final-newline, YAML, and large-file checks.
- `.github/build-maturin.yml`: generated wheel-build configuration; it is currently
  outside `.github/workflows/`.

The root README still contains Mintlify starter content. `dockerfile` and
`docker/compose.yml` are empty placeholders; there is no working Docker setup.

## Development commands

Run these from the repository root with `uv` and a Rust toolchain installed:

```sh
uv sync --locked
uv run --locked jev
uv run --locked ruff check src tests
uv run --locked ruff format --check src tests
uv run --locked pytest
cargo fmt --all -- --check
cargo check --locked
cargo test --locked
```

The CLI should print `Hello from jev!`. Pytest currently reports no tests and
exits with code 5; this is not a passing test suite. Rebuild the installed
extension after Rust changes before checking Python behavior, using
`uv sync --locked --reinstall-package jev` if necessary.

## Change conventions

- Keep changes small and follow the existing Python/Rust package structure.
- For extension API changes, update Rust exports, `_core.pyi`, Python callers,
  and relevant tests together. Keep the module name `jev._core` consistent with
  both manifests and the Rust module declaration.
- Add focused tests in `tests/test_*.py` for new behavior, including Python calls
  into Rust when the extension changes. Use Rust unit tests for internal logic.
- Use Ruff for Python formatting and linting, and rustfmt for Rust formatting.
- Manage Python dependencies in `pyproject.toml` with uv and Rust dependencies in
  `Cargo.toml` with Cargo. Update the corresponding lockfile when dependencies change.
- `requirements.txt` is generated; regenerate it with
  `uv export --frozen --no-dev -o requirements.txt` after Python dependency updates.
- Do not hand-edit generated extension binaries or add build output from
  `target/`, `.venv/`, `dist/`, or `__pycache__/` to commits.
- Enable verbose logging for development, keeping secrets out of logs.
- Run checks relevant to the change and report any checks that could not run.
