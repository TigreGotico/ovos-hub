# ovos-hub

A Docker image monorepo for [OpenVoiceOS](https://github.com/OpenVoiceOS). Each subdirectory holds a Dockerfile and its supporting files that build one container image in the `jarbasai/*` namespace, plus `docker-compose.yml` stacks to run them. The repo does not contain a Python package. It builds and orchestrates containers.

## Install

You need Docker or Podman. The compose files use `userns_mode: keep-id` and `label=disable` for rootless Podman.

Copy `.env.example` to `.env` in each stack directory you plan to run, then fill in the paths and credentials for that stack.

Build every image:

```bash
bash build.sh            # uses the build cache
bash build_no_cache.sh   # forces a clean rebuild
```

Build a single image:

```bash
docker build -t jarbasai/ovos-core:latest ./core/core
```

Build the base images first (`./base`, `./base-sound`, `./base-rocm`). Every other image is based on one of them.

## Usage

Each top-level stack has its own `docker-compose.yml` and `.env.example`: `core/`, `skills/`, `tts/`, `stt/`, `hivemind/`, `translate/`, `utils/`.

```bash
docker compose -f core/docker-compose.yml up -d
```

### Layout

- `base/`, `base-sound/`, `base-rocm/`: shared base images. `base` is `debian:trixie-slim` with the `ovos` user, XDG directories, a healthcheck script (`files/ovos-hc.py`), and a default `mycroft.conf`. It pins the OVOS pip/uv channel through `UV_CONSTRAINT` to `OpenVoiceOS/ovos-releases` constraints. `base-sound` adds audio support. `base-rocm` uses `rocm/pytorch` for GPU-based STT, TTS, and translation.
- `core/`: `core` (the ovos-core skills service), `messagebus`, `messagebus-rust`, `gui-websocket`, `audio`, `phal`, and three listener variants (`dinkum-listener`, `simple-listener`, `classic-listener`).
- `skills/`: one image per OVOS skill. Each image entrypoint runs `ovos-skill-launcher $SKILL_ID`.
- `tts/`: one `ovos-tts-server` image per engine (piper, mimic, mimic3, matxa, cotovia, nos, sam, google-tx).
- `stt/`: one `ovos-stt-server` image per engine (whisper, nemo, hitz, mynorthai), built on the rocm base.
- `translate/nllb/`: `ovos-translate-server` with NLLB and a fasttext language detector.
- `hivemind/`: `core`, `player`, `persona`, `chatroom`, `webchat`, and `matrix-bot` images for [HiveMind](https://github.com/JarbasHiveMind).
- `utils/`: `ovos-yaml-editor` and `ovos-skill-settings-editor` web tools.
- `build.sh` and `build_no_cache.sh` list every image and its tag.

To bump a dependency, edit the relevant `files/requirements.txt` file or the `uv pip install` line in a Dockerfile. OVOS package versions float on the alpha channel through `UV_CONSTRAINT`.

## Validation

There are no automated tests. To validate an image, build it, run the container, and check that its `HEALTHCHECK` passes.

Status endpoints: port 8080 for STT, 9666 for TTS, 9686 for translate, and 5678 for HiveMind. A messagebus probe runs through `base/files/ovos-hc.py`.

## Related projects

- [OpenVoiceOS/ovos-core](https://github.com/OpenVoiceOS/ovos-core): the skills service packaged by the `core` images.
- [OpenVoiceOS/ovos-releases](https://github.com/OpenVoiceOS/ovos-releases): the constraints channel that pins OVOS package versions in the base images.
- [JarbasHiveMind](https://github.com/JarbasHiveMind): the HiveMind projects packaged by the `hivemind` images.
