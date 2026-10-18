# Python WASM Console (proof of concept)

A single HTML file that gives you a real CPython REPL in the browser.
The interpreter runs **client-side in WebAssembly** via [Pyodide](https://pyodide.org).
There is no backend, no container, no Docker, nothing to provision.

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
- Pure-Python packages install at runtime from PyPI:

  ```python
  import micropip
  await micropip.install("requests")
  ```

  Pyodide also ships prebuilt wheels for numpy, pandas, scipy, matplotlib and more.

## Layout

```
index.html   the whole app: markup, styles, and ~150 lines of JS
README.md
```

The runtime (~10 MB, cached by the browser after first visit) is loaded from
`https://cdn.jsdelivr.net/pyodide/v0.28.3/full/`. To be fully offline, download that
directory next to `index.html` and point `loadPyodide({ indexURL: "./pyodide/" })` at it.

## Deploying

Push to GitHub and enable Pages on the `main` branch. That is the entire deployment.
