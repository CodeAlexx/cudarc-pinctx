# cudarc-pinctx

Vendored fork of cudarc 0.11.9 with one patch: `src/driver/safe/threading.rs`
caches the per-thread bound CUDA context in a thread-local. `bind_to_thread()`
short-circuits when the requested context is already the cached one for this
thread.

Cuts cuCtxSetCurrent storm in EriDiffusion's flame-core: zimage rank=8 LoRA
training went from 91,637 cuCtxSetCurrent/step → 0/step (absent from top-15
driver APIs). Cumulative trainer-side win: 200,472 → 88,903 driver API
calls/step (−55.7%).

## Setup

Place as a sibling of `flame-core/` and `EriDiffusion-v2/`:

```
EriDiffusion/
├── flame-core/
├── EriDiffusion-v2/
└── cudarc-pinctx/   ← this directory
```

Both `flame-core/Cargo.toml` and `EriDiffusion-v2/Cargo.toml` have
`[patch.crates-io] cudarc = { path = "../cudarc-pinctx" }`.

## Upstream parity

Diff against upstream cudarc-0.11.9:

```
src/driver/safe/threading.rs  | thread-local CUcontext cache + fast-path
```

All other files are identical to the published 0.11.9 crate.

## Migration path

When cudarc 0.19+ ships with similar caching upstream, drop this fork:

```toml
# Remove the [patch.crates-io] section from both Cargo.tomls
# Bump cudarc dep to 0.19+ (API may have breaking changes; review)
```
