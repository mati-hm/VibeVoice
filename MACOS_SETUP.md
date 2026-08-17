# VibeVoice on macOS Apple Silicon

This note records the verified local setup for this fork on Mati's MacBook Pro. The main `README.md` remains the general/upstream-style project documentation.

## Verified environment

- Host: Apple Silicon MacBook Pro (M5 Pro, 24 GB unified memory)
- macOS: native ARM64 environment
- Project checkout: `~/Code/VibeVoice`
- Python: 3.11.15, managed with `uv`
- Virtual environment: `~/Code/VibeVoice/.venv`
- PyTorch: 2.13.0
- Apple MPS: built and available
- Model tested: `vibevoice/VibeVoice-1.5B`
- UI: Gradio on local loopback

No Docker, CUDA, Rosetta, Conda, or global Python installation is required for this setup.

## Install

Clone the repository:

```bash
cd ~/Code
git clone git@github.com:mati-hm/VibeVoice.git
cd VibeVoice
```

Create an isolated Python 3.11 environment:

```bash
uv python install 3.11
uv venv --python 3.11
source .venv/bin/activate
```

Install the project in editable mode:

```bash
uv pip install -e .
```

Verify Apple MPS support:

```bash
python - <<'PY'
import torch
print("PyTorch:", torch.__version__)
print("MPS built:", torch.backends.mps.is_built())
print("MPS available:", torch.backends.mps.is_available())
PY
```

Verified result:

```text
PyTorch: 2.13.0
MPS built: True
MPS available: True
```

## Run

From a fresh terminal:

```bash
cd ~/Code/VibeVoice
source .venv/bin/activate

python demo/gradio_demo.py \
  --model_path vibevoice/VibeVoice-1.5B \
  --device mps
```

Open the local Gradio interface at:

```text
http://127.0.0.1:7860
```

Do not add `--share` unless a public Gradio tunnel is explicitly wanted.

## Apple Silicon behavior

The current fork contains an MPS-specific path in `demo/gradio_demo.py`:

- device: `mps`
- model dtype: `torch.float16`
- attention implementation: `sdpa`
- model moved directly to MPS after loading

This path has been verified working on the M5 Pro.

The following startup messages are expected and were not blockers during testing:

```text
APEX FusedRMSNorm not available, using native implementation
audio_utils not available, will fall back to soundfile for audio loading
```

A tokenizer-class warning may also appear during model loading; the tested 1.5B model continued to load and generate successfully.

## Model storage

Model weights are downloaded through Hugging Face and stored in the normal Hugging Face cache rather than in the Git repository.

Relevant cache root:

```text
~/.cache/huggingface/hub/
```

The tested VibeVoice model appears under a path equivalent to:

```text
~/.cache/huggingface/hub/models--vibevoice--VibeVoice-1.5B/
```

The Python virtual environment remains local under `.venv/` and should not be committed.

## Verified baseline

A one-speaker English generation using bundled voice `en-Alice_woman` completed successfully with:

```text
Model:            VibeVoice-1.5B
Device:           MPS
CFG Scale:        1.3
Inference Steps:  10
Seed:             random
Voice cloning:    enabled
```

Verified run:

```text
Generation time:  8.31 seconds
Audio duration:   9.47 seconds
Total chunks:     71
```

This short test therefore generated slightly faster than real time on the M5 Pro.

A previous run using fixed seed `42` clipped the final word; regenerating with a random seed produced intact audio. Treat that as sampling variability rather than an installation failure unless it becomes reproducible across multiple seeds.

## Routine defaults

For normal interactive use, start with:

```text
Model:            VibeVoice-1.5B
Device:           MPS
CFG Scale:        1.3
Inference Steps:  10
Seed:             random
```

Use a fixed seed only when reproducibility is specifically needed.

## Updating dependencies

The repository currently pins important compatibility-sensitive dependencies in `pyproject.toml`, including its Transformers version. Prefer reinstalling from the project definition rather than independently upgrading core packages inside the environment:

```bash
source .venv/bin/activate
uv pip install -e .
```

If upstream/fork dependency requirements change, follow the repository's current `pyproject.toml` and README rather than this historical verified-version snapshot.
