# llm-tps-calculator

A single-page calculator that estimates how fast a local LLM will generate text on a given machine: dedicated GPUs plus system RAM, or unified memory (Apple Silicon, iGPU mini PCs).

The estimate is memory-bound, in the style of llama.cpp: each generated token reads the active weights and the KV cache on every device that holds part of the model, plus the PCIe hops between devices.

## Usage

Open `index.html` in a browser. There is no build step, no dependency and no server; everything is in that one file.

1. Pick a model and a quantization.
2. Set the context size and how many tokens are already in it.
3. Pick the machine type, then the GPUs and RAM (or the chip and unified memory).

The results update as you type. Settings are saved in the browser's `localStorage`; **Reset** restores the defaults.

## What it shows

- **Generation speed** in tokens/s, and the time per token.
- **Prompt processing speed**, and the time to ingest the current context.
- **Memory required**, split into weights and KV cache.
- **Memory usage** per device: which part of the model lands on each GPU, in RAM, or on disk.
- **Time per generated token**, broken down by device and PCIe transfers.
- **Generation speed against filled context**, as a chart.
- Warnings, for example when the model overflows to disk, when GPUs from different vendors are mixed, or when a GPU sits on a narrow PCIe link.

## Advanced settings

The **Advanced settings** checkbox in the header reveals everything the simple view hides:

- Model architecture: parameter counts, layers, hidden size, KV heads, sliding-window layers, bits per weight.
- MTP drafter (speculative decoding).
- KV cache type.
- Per-GPU details: VRAM, memory bandwidth, TFLOPS, PCIe generation and lanes for the card and the slot, VRAM reserve.
- RAM reserve, CPU throughput, NVMe speed.
- Placement strategy, GPU fill order and the efficiency factors.
- The theoretical-ceiling figure and the per-device table.

Choosing **Custom** for the model or for a GPU shows that item's fields even when advanced settings are off.

## Quantization

Only formats at or below the precision a model was released in are offered. A model published in 8-bit gains nothing from an F16 conversion, so F16 is not listed for it. Native precisions come from the Hugging Face safetensors metadata (checked November 2026); models with no entry in the `NATIVE` table are assumed to have a BF16 release.

## Presets

All presets live in tables near the top of the script in `index.html`:

| Table | Content |
| --- | --- |
| `GPU_GROUPS` | GPUs by family: GeForce, Radeon, Arc, workstation, datacenter |
| `MODEL_PRESETS` | Open-weight models, with the architecture figures the KV formula needs |
| `NATIVE` | Models released below 16-bit |
| `UNIFIED_PRESETS` | Apple Silicon chips and iGPU mini PCs |
| `QUANTS` | Quantization formats and their bits per weight |

To add an entry, append a row in the same column order as its neighbours; the comment above each table lists the columns.

## Accuracy

The result is a theoretical ceiling. Real numbers are typically 10–30% lower, depending on the backend, the quantized kernels and system load.

Several inputs are approximate:

- GPU memory bandwidth and TFLOPS for workstation and datacenter cards are orders of magnitude, partly scaled from the consumer sibling.
- Some model figures (dense part, active parameters, drafter sizes) are estimates where the vendor publishes none; the comments in `MODEL_PRESETS` say which.

The formulas are listed in the page itself, under **Method**.
