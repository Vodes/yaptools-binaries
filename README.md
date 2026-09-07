# muxtools-binaries

Build and package portable tools for Linux and Windows x86-64. Source builds use one manylinux_2_34-derived Linux image; Windows builds cross-compile through MinGW. Build tools, LLVM, and MinGW come from the image's repositories. The final image digest freezes their versions.

Packages: fdkaac, FLAC, Opus tools, FFmpeg/FFprobe, MKVToolNix, and multilib x265. Source-built Linux executables target x86-64-v2 and glibc 2.34. x265 also provides explicit AVX2, AVX512, and zn4 variants, each supporting 8/10/12-bit encoding. Imported packages declare additional runtime requirements where necessary.

## Local development

Install Docker and uv, then:

```sh
uv sync --frozen
uv run muxtools-build validate
uv run pytest -q
uv run mypy
docker build -t muxtools-builder:local -f builder/Dockerfile .
uv run muxtools-build build flac --target linux-x86_64 --image muxtools-builder:local
uv run muxtools-build test dist/flac-1.5.0-linux-x86_64.tar.zst
```

Builds, downloads, and source checkouts go under ignored `build/`; final archives go under ignored `dist/`. `context/` remains ignored local reference material. Builds always run when requested, regardless of published versions. Windows artifacts must be smoke-tested on Windows; CI supplies native runners.

Use `uv run muxtools-build --help` for matrix generation, repackaging, update discovery, and publishing commands. See [the architecture and artifact contracts](docs/architecture.md) for recipe authoring, runtime requirements, and catalog recovery.

## Releases

PRs and pushes build test artifacts only. Official publication requires manually running **Build and test** on `main` with **publish** checked. It defaults to off. Before the first release, run **Qualify builder**, publish the tested image through its own manual checkbox, and adopt its GHCR digest in `builder/lock.toml`.

Archives use `.tar.zst` on both platforms and contain `.metadata.toml`. The new published `versions.json` lives on the `catalog-v1` release. This is a clean break from the old ZIP/JSON contract; historical releases remain available. No automatic upstream update publishes binaries.
