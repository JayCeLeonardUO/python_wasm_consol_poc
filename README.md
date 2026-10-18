# CPython in the browser, built from source

**Live:** https://jayceleonarduo.github.io/python_wasm_consol_poc/

This repo proves an interpreter can be deployed client-side with no server and no Docker.
The page runs **stock CPython 3.14.8 compiled to WebAssembly with Emscripten 4.0.12**,
straight from the CPython repo's own `Platforms/emscripten build` target. No Pyodide,
no third-party runtime. `python.wasm`, its JS glue and a zipped standard library are
committed here and served as static files by GitHub Pages. Every visitor runs their own
interpreter on their own CPU.

## Use it

- **Start REPL** gives an interactive `>>>` prompt that keeps state between lines.
- **Run code** runs the editor contents as a script, like `python main.py`, prints, and exits.
- Works in Chrome and Firefox. Nothing is sent anywhere after the files load.

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
