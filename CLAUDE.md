# Global Claude Code Instructions

## Coding Standards

- Prefer simpler solutions first. Overengineered code is generally bad, especially if it adds complexity. 
- Prefer fewer command line arguments, unless I ask for it to be an argument explicitly. Feel free to hardcode constants in clearly marked places in scripts and source modules instead. (A good guideline is that more than 4 CLI arguments or more than 2 optional arguments is a sign that we have too many.)
- Prefer functions to classes, and prefer stateless functions to stateful functions. 

## Python Standards

- **Do NOT make Python scripts executable** (no `chmod +x` on `.py` files) unless explicitly requested
- Always invoke Python scripts via `uv python script.py` or `python script.py`, not as `./script.py`.
- If there is a `uv.lock` file present in the repository, use uv.
- Never install Python packages, I will do that.

## Testing Requirements

After writing or modifying code, always validate changes before reporting completion:

### Unit Tests
- Run unit tests **only for files that were modified**, not the entire test suite
- Use pytest's targeted test discovery:
  - If you modified `src/foo/bar.py`, look for and run `tests/test_bar.py` or `tests/foo/test_bar.py`
  - Use `pytest tests/ -k "test_bar"` or run the specific test file directly
- If no matching test file exists, note this in your response

### Modified Scripts
- If you modified a script in `scripts/`, `examples/`, or similar directories, execute it to verify it works
- **Sandboxing**: Always run scripts with a timeout to prevent hangs:
  - Use `timeout 60` prefix (or appropriate duration) for potentially long-running scripts
  - For Python scripts that might hang: `timeout 60 python script.py`
  - If a script requires user input or runs indefinitely, run with `--help` or a dry-run flag if available
- **DO NOT execute scripts that would incur production costs** (cloud API calls, GPU usage, external paid services, etc.). Instead, flag that you are skipping execution and explain why.
- Do not execute scripts that would:
  - Make network requests to external services (unless that's the purpose being tested)
  - Modify production data

### Reporting
- Always report test/script results before marking work as done
- If tests fail, fix the issues before reporting completion
- If you cannot run tests (missing dependencies, cost concerns, etc.), explicitly note this
