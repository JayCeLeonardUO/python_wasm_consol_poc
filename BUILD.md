# How these files were built

Everything in this directory except `index.html`, `python.worker.mjs` and
`coi-serviceworker.js` is a verbatim build output of stock CPython. No Pyodide.

| File | What it is |
|---|---|
| `python.wasm` | CPython 3.14.8 interpreter, all extension modules statically linked (11.1 MB) |
| `python.mjs` | Emscripten glue: memory, in-memory filesystem, TTY, module loader (572 KB) |
| `python3.14.zip` | pyc-only standard library, written into the virtual FS before start (3.8 MB) |
| `python.worker.mjs` | CPython's `web_example` worker, plus a 3-line fix to report the real exit status |
| `index.html` | CPython's `web_example` page with a provenance note and the service-worker shim |
| `coi-serviceworker.js` | [gzuidhof/coi-serviceworker](https://github.com/gzuidhof/coi-serviceworker) @ 7b1d2a0, adds COOP/COEP headers that GitHub Pages cannot send |

## Source

- CPython branch `3.14`, commit `2224ffff9cb837c30ac36867f22ffb2bf7a5d5c0` (3.14.8+)
- Emscripten SDK 4.0.12 (the version pinned in `Platforms/emscripten/config.toml`)
- Node 24 (used only as the build-time host runner)
- Host: Pop!_OS 22.04, x86_64, clang 19 from the emsdk

## Commands

```bash
git clone --depth 1 --branch 3.14 https://github.com/python/cpython.git
cd cpython
python3.13 Platforms/emscripten install-emscripten --emsdk-cache ../emsdk-cache
python3.13 Platforms/emscripten build            --emsdk-cache ../emsdk-cache
# outputs land in cross-build/wasm32-emscripten/build/python/
#   python.mjs python.wasm python3.14.zip python.sh web_example/ web_example_pyrepl_jspi/
```

The build script configures with
`--host=wasm32-emscripten --with-build-python=... --disable-shared --enable-wasm-dynamic-linking`
and links with `-sMAIN_MODULE -sALLOW_MEMORY_GROWTH -sFORCE_FILESYSTEM -sEXIT_RUNTIME`
(see `configure.ac`, "WASM flags"). The binary is not stripped or post-processed.

## Why the worker + SharedArrayBuffer design

CPython also ships a JSPI-based example (`web_example_pyrepl_jspi`) that runs the real
pyrepl on the main thread. It needs JavaScript Promise Integration, and on Chrome 154 it
crashes the tab on the first keystroke whenever Chrome's heap-profiler field trial is active
(the profiler's stack scanner walks off the end of the JSPI stack). The worker design here
blocks on stdin with `Atomics.wait` on a SharedArrayBuffer instead, which needs
cross-origin isolation but no JSPI, and works in Chrome and Firefox.

## Verify the artifact yourself

```bash
wasm-objdump -x -j Import python.wasm | grep -c '<- env\.'   # Emscripten host imports
./cross-build/wasm32-emscripten/build/python/python.sh -c "import sys; print(sys.platform)"   # emscripten
```

## Why scikit-learn is not in this build

`python3.14.zip` is the standard library only. scikit-learn needs numpy, scipy, OpenBLAS and
Cython extensions compiled for wasm32-emscripten. Pyodide publishes exactly those, but for its
`pyemscripten_2026_0` ABI: CPython 3.14 built with **Emscripten 5.0.3**, `-fwasm-exceptions`,
`-sSUPPORT_LONGJMP=wasm`, and Pyodide's own dynamic-library loader with RPATH support. This
build uses Emscripten 4.0.12 and CPython's stock loader, so those wheels do not load here.
Matching the ABI would mean rebuilding CPython to Pyodide's spec and cross-compiling the whole
SciPy stack, which is the Pyodide project itself. The page therefore offers Pyodide 314 as a
second runtime for sklearn/numpy/pandas and keeps this build as the from-source proof.
