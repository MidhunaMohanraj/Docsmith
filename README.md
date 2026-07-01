# docsmith

AI-assisted documentation generator for Python codebases. Point it at a
directory, get back a `DOCS.md` with a written project overview plus a
full API reference — signatures, docstrings, classes, and functions.

- **Offline by default.** Docstring/signature extraction uses only Python's
  built-in `ast` module — no network calls, no dependencies beyond `click`.
- **AI-assisted overview.** If `ANTHROPIC_API_KEY` is set, docsmith asks
  Claude to write a short human-readable project summary from the extracted
  structure and prepends it to the reference docs.
- **Fails gracefully.** No API key, no `anthropic` package, or a network
  error — docsmith just skips the AI step and ships the reference docs.

## Install

```bash
pip install -r requirements.txt
pip install -e .
```

## Usage

```bash
# Offline reference docs only
docsmith path/to/your/project -o DOCS.md

# With an AI-written overview
export ANTHROPIC_API_KEY=sk-ant-...
docsmith path/to/your/project -o DOCS.md

# Force offline even if a key is set
docsmith path/to/your/project --no-ai
```

Try it on the bundled example:

```bash
docsmith examples/sample_project -o example_docs.md
cat example_docs.md
```

## How it works

1. `scanner.py` walks the target directory with `pathlib`, parses each
   `.py` file with `ast`, and extracts modules, classes, functions,
   signatures, and existing docstrings into plain dataclasses.
2. `generator.py` renders that structure into Markdown. If an API key is
   available, it also sends a compact summary (file → class/function names)
   to Claude and asks for a 3-5 sentence overview, which becomes the top of
   the output file.
3. `cli.py` wires it together behind a `docsmith` command built with
   `click`.

## Development

```bash
pip install -r requirements.txt
pytest
```

Tests cover the scanner (parsing, docstring extraction, ignoring
`__pycache__`/syntax-error files) and the generator's offline fallback
path — no network access required to run the suite.

## Project layout

```
src/docsmith/
  scanner.py     # ast-based extraction into ModuleInfo/ClassInfo/FunctionInfo
  generator.py   # Markdown rendering + optional Claude API call
  cli.py         # click-based command line entry point
tests/           # pytest suite, no network required
examples/        # tiny sample project used by tests and as a live demo
```

## License

MIT — see [LICENSE](LICENSE).
