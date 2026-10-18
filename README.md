# Python WASM Console (proof of concept)

A single HTML file that gives you a real CPython REPL in the browser.
The interpreter runs **client-side in WebAssembly** via [Pyodide](https://pyodide.org).
There is no backend, no container, no Docker, nothing to provision.

**Live demo:** https://jayceleonarduo.github.io/python_wasm_consol_poc/
(served straight from this repo by GitHub Pages)

## Why

The goal is to prove that an interpreter can be shipped as a static asset:
host `index.html` anywhere that serves files (GitHub Pages, S3, nginx, a USB stick)
and every visitor gets their own isolated Python interpreter running on their own CPU.

| Traditional "online Python" | This repo |
|---|---|
| Server runs the code | Browser runs the code |
| Docker/Firecracker sandbox per user | Browser sandbox, zero infra |
| Scales with your bill | Scales with the visitor's laptop |
| Network round trip per statement | Local execution, no round trip |

## Run it

Any static file server works:

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

Opening `index.html` directly from disk (`file://`) also works in Chrome and Firefox,
because the runtime is fetched from the jsDelivr CDN with permissive CORS headers.

## What it does

- Full REPL semantics using Pyodide's `PyodideConsole`: multi-line blocks, `...` continuation
  prompts, syntax errors, tracebacks, and top-level `await`.
- Streaming `stdout` / `stderr`, so `print(..., end="")` behaves correctly.
- Command history (↑ / ↓), Tab inserts 4 spaces, Ctrl+L clears, Ctrl+C cancels the current input.
- Packages load automatically on import (see below).

## What can you import?

Far more than the stdlib. Three tiers:

**1. Prebuilt for WebAssembly (just `import` it).**
Pyodide 0.28 ships 340+ packages compiled to wasm, including real C/Fortran extensions.
The console scans each statement for imports and fetches what is missing from the CDN
before running it. Highlights: numpy, scipy, pandas, **scikit-learn**, matplotlib,
statsmodels, sympy, networkx, pillow, opencv, xgboost, lightgbm, scikit-image, shapely,
lxml, beautifulsoup4, nltk, cryptography, pydantic, sqlalchemy, sqlite3, requests, bokeh,
altair. Full list: https://pyodide.org/en/stable/usage/packages-in-pyodide.html

```python
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
X, y = load_iris(return_X_y=True)
LogisticRegression(max_iter=500).fit(X, y).score(X, y)
```

The first `import sklearn` downloads about 23 MB (mostly scipy); the browser caches it
after that. A link can arrive pre-warmed: `index.html?packages=scikit-learn,pandas`.

**2. Any pure-Python package on PyPI (via micropip).**

```python
import micropip
await micropip.install("seaborn")
import seaborn
```

**3. Not possible (today).**
torch, tensorflow, jax, numba, polars, pyarrow, duckdb, transformers, onnxruntime.
These have native extensions nobody has built for wasm, or need things the browser
sandbox does not offer: real threads / multiprocessing, raw sockets, subprocesses, a GPU.
Memory tops out around 2–4 GB per tab.

## Layout

```
index.html   the whole app: markup, styles, and ~150 lines of JS
README.md
```

The runtime (~10 MB, cached by the browser after first visit) is loaded from
`https://cdn.jsdelivr.net/pyodide/v0.28.3/full/`. To be fully offline, download that
directory next to `index.html` and point `loadPyodide({ indexURL: "./pyodide/" })` at it.

## Deploying

This repo is deployed with GitHub Pages, source = `main` branch, folder = `/`.
Every push to `main` redeploys. That is the entire deployment pipeline.
