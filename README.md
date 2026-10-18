# CPython in the browser, built from source

**Live:** https://jayceleonarduo.github.io/python_wasm_consol_poc/

This repo proves an interpreter can be deployed client-side with no server and no Docker.
One page, two runtimes, picked with the switch at the top:

| Runtime | What it is | What you can import |
|---|---|---|
| **Self-built CPython 3.14** (default) | stock CPython 3.14.8 compiled to WebAssembly with Emscripten 4.0.12 straight from the CPython repo's `Platforms/emscripten build` target. `python.wasm`, its JS glue and a zipped stdlib are committed here and served as static files. No Pyodide. | the standard library |
| **Pyodide 314** | the same recipe done by the Pyodide project: CPython 3.14.2 + Emscripten 5.0.3 plus 356 packages cross-compiled for wasm, loaded from jsDelivr on first click. | scikit-learn, numpy, scipy, pandas, matplotlib, sympy, networkx, pillow, opencv, xgboost, lightgbm, … plus any pure-Python PyPI package via `micropip` |

Either way every visitor runs the interpreter on their own CPU. Nothing is executed server-side.

## Use it

Self-built tab:
- **Start REPL** gives an interactive `>>>` prompt that keeps state between lines.
- **Run code** runs the editor contents as a script, like `python main.py`, prints, and exits.

Pyodide tab:
- Type at the prompt. Imports of prebuilt packages are fetched automatically before the line
  runs; the first `import sklearn` downloads about 25 MB (mostly scipy) and is then cached.
- **Run sklearn demo** types and runs an iris classification example.
- `?runtime=pyodide&packages=scikit-learn` opens that tab with sklearn preloaded.
- Anything pure-Python on PyPI: `import micropip; await micropip.install("seaborn")`.

Not possible in either runtime: torch, tensorflow, jax, numba, polars, pyarrow, duckdb, or
anything needing threads, sockets, subprocesses or a GPU. Works in Chrome and Firefox.

## Why sklearn is on the Pyodide tab

scikit-learn needs numpy, scipy, OpenBLAS and its own Cython extensions, all cross-compiled
for wasm. Pyodide's prebuilt wheels target the `pyemscripten_2026_0` ABI (Emscripten 5.0.3,
wasm exceptions, Pyodide's dynamic loader) and cannot load into an Emscripten 4.0.12 build.
Cross-compiling that stack yourself is, quite literally, the Pyodide project.

## Files

| File | Size | What it is |
|---|---|---|
| `python.wasm` | 11.1 MB | the interpreter; all extension modules statically linked |
| `python.mjs` | 572 KB | Emscripten glue: memory, virtual filesystem, TTY |
| `python3.14.zip` | 3.8 MB | pyc-only standard library, written into the virtual FS at start |
| `python.worker.mjs` | | runs the interpreter in a Web Worker; stdin blocks on a SharedArrayBuffer |
| `index.html` | | the terminal page (from CPython's `web_example`, with a provenance note) |
| `coi-serviceworker.js` | | adds the cross-origin isolation headers that GitHub Pages cannot send |

[BUILD.md](BUILD.md) has the exact source commit, toolchain versions and commands.

## Run locally

```bash
python3 -m http.server 8000
# open http://localhost:8000/   (first load registers the service worker and reloads once)
```

## Deploying

GitHub Pages, source = `main`, folder = `/`. Pushing to `main` is the whole pipeline.
Do not put the `.wasm` in Git LFS: Pages does not serve LFS objects.

## History

The first iteration of this proof used Pyodide's prebuilt CPython from a CDN.
It is preserved at the git tag `pyodide-poc`.
