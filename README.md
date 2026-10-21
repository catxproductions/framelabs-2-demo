# FrameLab — original project with CPU worker and optional WebGPU compute

This package is based on the supplied original FrameLab HTML project. Its built-in building examples, interface, and authoritative structural solver remain in place.

## Run

Serve the `framelab-original-modular` folder over HTTP/HTTPS (for example, GitHub Pages or `python -m http.server`), then open `index.html`. The CPU worker and WebGPU require a supported browser context; do not judge worker support from a `file://` launch.

## What the compute integration does

- `js/compute-worker.js` scans a snapshot of active structural members' demand/capacity values on a dedicated CPU worker.
- `js/compute-backends.js` resolves the worker relative to its own script URL, so the worker path also works when FrameLab is deployed under a GitHub Pages subpath such as `/framelabs/`.
- WebGPU initialization is lazy. It is attempted only for large snapshots (2,048+ members) to avoid GPU startup and readback overhead for normal-sized structures. If WebGPU is unavailable, initialization fails, the device is lost, or a dispatch fails, the scan falls back to the CPU worker; if that worker fails, the same diagnostic uses a main-thread fallback.
- The **Parallel compute** metric reports the active backend and the supplemental scan summary.
- The scan excludes failed/snapped members and does not alter simulation state.

## Important limitation

This is an additional parallel diagnostic, not a rewrite of the solver. FrameLab's authoritative structural load propagation, connectivity, gravity, member failure, collisions, debris/fire/explosion simulation, and rendering remain in the original main-thread JavaScript. The diagnostic alone does not guarantee higher frame rates; the next substantial optimization would require moving a well-bounded, expensive part of the real solver into a worker and validating equivalent results against the current implementation.

## Checks performed

Run `node --check js/main.js`, `node --check js/compute-backends.js`, `node --check js/compute-worker.js`, and `node --test tests/compute-smoke.test.cjs`. The smoke tests validate the built-in example JSON, local assets, GitHub Pages worker URL, CPU scan math, GPU initialization fallback, and worker-crash fallback. Browser runtime testing should additionally be performed in a normal browser. WebGPU availability and performance vary by browser, OS, driver, and GPU.
