# sd-webui-forge-docker

[![Nightly Build](https://github.com/delfianto/sd-webui-forge-docker/actions/workflows/build.yml/badge.svg)](https://github.com/delfianto/sd-webui-forge-docker/actions/workflows/build.yml)

A Dockerfile whose entire purpose in life is to stop me from reinstalling `torch` on bare metal for the fourth time this year.

## The situation

Stable Diffusion WebUI Forge is great. Forge *Classic* is great. Forge *Neo* — the actively-maintained continuation of the continuation — is also great, assuming you enjoy the ritual of `git clone`, discovering your Python is the wrong minor version, building a venv, watching `pip` argue with `xformers` for twenty minutes, and then doing all of it again on the next machine. This repo exists so that ritual happens exactly once, inside a container, and then never again for as long as the image tag doesn't change.

It is not clever. It clones the [`neo` branch of `sd-webui-forge-classic`](https://github.com/haoming02/sd-webui-forge-classic), gives it Python 3.13 and CUDA 13.2, and gets out of the way.

## What's actually in here

- **Base**: `nvidia/cuda:13.2.0-base-ubuntu24.04`
- **Python**: 3.13, via the `deadsnakes` PPA, because Ubuntu 24.04's system Python has opinions
- **Package manager**: [`uv`](https://github.com/astral-sh/uv) — because waiting for `pip` to resolve a Stable Diffusion dependency tree is a young person's game
- **App**: a shallow clone of `haoming02/sd-webui-forge-classic` (`neo` branch) with its `.git` immediately deleted, because we are shipping an application, not a history lesson
- **Entrypoint**: on first boot, builds a venv at `/app/venv` and installs `requirements.txt`. On every boot after that, it notices the venv already exists and skips straight to launching — assuming you mounted `/app/venv` as a volume, which you should, unless you enjoy re-installing `xformers` on every container restart. We've been over this.

## Image

Published to `ghcr.io/delfianto/sd-webui-forge:latest`, rebuilt nightly (00:00 UTC) by GitHub Actions, plus a `sha-<short>` tag on every build so you can pin a specific one when `latest` inevitably breaks something at 2 AM.

```bash
docker pull ghcr.io/delfianto/sd-webui-forge:latest
```

## Running it

This repo ships the image, not a deployment. For a full working example — GPU device, volumes, Traefik labels, the works — see [`ai/sd-webui-forge`](https://github.com/delfianto/compose/tree/main/ai/sd-webui-forge) in my homelab compose repo. The short version:

```bash
docker run --rm \
  --gpus all \
  -p 7860:7860 \
  -v ./venv:/app/venv \
  -v ./uv_cache:/app/uv_cache \
  -v ./models:/comfyui \
  -v ./output:/app/output \
  ghcr.io/delfianto/sd-webui-forge:latest \
  --listen --port 7860 --cuda-malloc --forge-ref-comfy-home /comfyui
```

Anything after the image name is passed straight through to `launch.py`, so any Forge/A1111 CLI flag works here too.

## Building it yourself

```bash
git clone https://github.com/delfianto/sd-webui-forge-docker
cd sd-webui-forge-docker
docker build -t sd-webui-forge:local .
```

Nothing fancy — no build args, no multi-stage cleverness. The Dockerfile is short enough to just read.

## Disclaimer

This image contains someone else's Stable Diffusion fork, glued to a specific CUDA version, wrapped in a venv that gets built once and trusted forever. If a new dependency shows up upstream and the venv is stale, the fix is "delete the venv volume and let it reinstall," not "open an issue." Nightly builds mean `latest` tracks upstream `neo` fairly closely, which is either a feature or a warning depending on how upstream's week is going.

No warranty, express or implied, on your VRAM, your patience, or the eleven browser tabs of LoRA recommendations you're about to open.

## Credits

- [haoming02/sd-webui-forge-classic](https://github.com/haoming02/sd-webui-forge-classic) — the actual application. This repo just puts a container around it.
- [astral-sh/uv](https://github.com/astral-sh/uv) — for making the install step merely slow instead of geologically slow.

## License

MIT — see [LICENSE](LICENSE).
