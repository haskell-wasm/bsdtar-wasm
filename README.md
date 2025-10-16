# `bsdtar-wasm`

[![Chat on Matrix](https://matrix.to/img/matrix-badge.svg)](https://matrix.to/#/#haskell.wasm:matrix.org)

This repo hosts a `wasm32-wasi` build of the `bsdtar` CLI from the
[libarchive](https://libarchive.org) project, with
[`zstd`](https://facebook.github.io/zstd) support. It can be used to
extract a `.tar.zst` archive in the wasi vfs, useful when:

- You need to download a large binary artifact, and the default
  gzip/brotli compression provided by the CDN is not good enough
- You need to set up a vfs with many files, and issuing one `fetch`
  per file can become a bottleneck

## How to use

### The wasm modules

There are two `wasm32-wasi` modules available here:

- [`bsdtar.wasm`](https://haskell-wasm.github.io/bsdtar-wasm/bsdtar.wasm)
- [`zstd.wasm`](https://haskell-wasm.github.io/bsdtar-wasm/zstd.wasm)

They are command modules without any custom imports/exports, fully
single-threaded, compiled with LLVM ThinLTO and optimized by
`wasm-opt`, with `simd128` feature enabled. `bsdtar.wasm` only
supports `zstd`, doesn't support other filters like `xz` or `gzip`.

Simply run them and pass the right CLI arguments as well as the right
input (either stdin or a file in the wasi vfs), then check the exit
code and use the output.

Prefer `zstd.wasm` when only decompressing a single file, otherwise
use `bsdtar.wasm`.

### The wasi implementation

The wasm modules work fine with `wasmtime` and `node`/`uvwasi`. But
for browsers, the only wasi implementation that works right now is
[`haskell-wasm/browser_wasi_shim`](https://github.com/haskell-wasm/browser_wasi_shim),
which can be directly imported from this
[URL](https://esm.sh/gh/haskell-wasm/browser_wasi_shim). Upstream npm
release lacks some important bugfixes at the moment.

## Building

The wasm-enabled forks of the relevant projects are available at:

- [`haskell-wasm/zstd`](https://github.com/haskell-wasm/zstd)
- [`haskell-wasm/libarchive`](https://github.com/haskell-wasm/libarchive)

Start from `install-wasi.sh` in the above repos.

The wasm patches are based on upstream release branches instead of
development branches, and there's currently no plan to upstream them
due to limited time. That being said, the heavylifting is done by
upstream and the wasm patches are just bandaids to get things working
for my use-case; feel free to use them as starting points for building
whatever you have in mind.
