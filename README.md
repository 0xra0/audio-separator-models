# 🎵 Audio Separator Models

<p align="center">
  <img src="https://img.shields.io/badge/models-5-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/architectures-MDX%20%7C%20VR%20%7C%20BS--Roformer%20%7C%20Demucs-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/license-MIT-orange?style=for-the-badge" />
  <img src="https://img.shields.io/github/release/0xra0/audio-separator-models?style=for-the-badge" />
</p>

> Pre-trained model weights for [audio-separator](https://github.com/nomadkaraoke/python-audio-separator) — a high-quality audio source separation library supporting **vocals, instrumentals, drums, bass, and more**.

---

## 📦 Included Models

| File | Architecture | Primary Stem | Size |
|------|-------------|-------------|------|
| `model_bs_roformer_ep_317_sdr_12.9755.ckpt` | BS-Roformer | Vocals | ~639 MB |
| `MDX23C-8KFFT-InstVoc_HQ.ckpt` | MDX23C | Vocals / Instrumental | ~448 MB |
| `MDX23C_D1581.ckpt` | MDX23C | Vocals / Instrumental | ~183 MB |
| `UVR-MDX-NET-Inst_HQ_3.onnx` | MDX-NET (ONNX) | Instrumental | ~67 MB |
| `955717e8-8726e21a.th` | Demucs HTDemucs | Multi-stem | ~84 MB |

### Model Architecture Details

#### 🔵 BS-Roformer (`model_bs_roformer_ep_317_sdr_12.9755`)
- **SDR Score:** 12.9755 dB (state-of-the-art vocal separation)
- **Architecture:** Band-Split Roformer with Flash Attention
- **Sample Rate:** 44,100 Hz | **Channels:** Stereo
- **FFT Size:** 2048 | **Hop Length:** 441
- **Transformer Depth:** 12 layers, 8 heads, dim 512
- **Config:** `model_bs_roformer_ep_317_sdr_12.9755.yaml`

#### 🟢 MDX23C — High Quality (`MDX23C-8KFFT-InstVoc_HQ`)
- **Architecture:** MDX23C with 8K FFT
- **Stems:** Vocals + Instrumental (2-stem)
- **FFT Size:** 8192 | **Sample Rate:** 44,100 Hz
- **Config:** `model_2_stem_full_band_8k.yaml`

#### 🟡 MDX23C — D1581 (`MDX23C_D1581`)
- **Architecture:** MDX23C
- **Stems:** Vocals + Instrumental (2-stem)
- **Config:** `model_2_stem_061321.yaml`

#### 🔴 UVR-MDX-NET Instrumental HQ 3 (`UVR-MDX-NET-Inst_HQ_3.onnx`)
- **Architecture:** MDX-NET exported as ONNX (CPU/GPU compatible)
- **Primary Stem:** Instrumental
- **Runtime:** ONNX Runtime (no PyTorch required for inference)

#### ⚪ HTDemucs (`955717e8-8726e21a.th`)
- **Architecture:** Hybrid Transformer Demucs
- **Config:** `htdemucs.yaml`
- **Stems:** Multi-stem separation

---

## 🚀 Installation

### Prerequisites

- Python **3.9 – 3.12**
- pip
- (Optional) CUDA-compatible GPU for faster processing

### 1. Install `audio-separator`

```bash
# CPU-only
pip install "audio-separator[cpu]"

# GPU (CUDA 11.x)
pip install "audio-separator[gpu]"

# GPU (CUDA 12.x)
pip install "audio-separator[gpu]" --extra-index-url https://download.pytorch.org/whl/cu121
```

### 2. Clone this model repository

```bash
git clone https://github.com/0xra0/audio-separator-models.git
cd audio-separator-models
```

Or download individual model files directly from the [Releases](https://github.com/0xra0/audio-separator-models/releases) page.

---

## 🎛️ Usage

### Command Line Interface (CLI)

```bash
# Separate vocals using BS-Roformer (best quality)
audio-separator my_song.mp3 \
  --model_filename model_bs_roformer_ep_317_sdr_12.9755.ckpt \
  --model_file_dir /path/to/audio-separator-models

# Separate vocals + instrumental using MDX23C HQ
audio-separator my_song.mp3 \
  --model_filename MDX23C-8KFFT-InstVoc_HQ.ckpt \
  --model_file_dir /path/to/audio-separator-models

# Use ONNX model (no GPU required)
audio-separator my_song.mp3 \
  --model_filename UVR-MDX-NET-Inst_HQ_3.onnx \
  --model_file_dir /path/to/audio-separator-models

# Demucs multi-stem separation
audio-separator my_song.mp3 \
  --model_filename 955717e8-8726e21a.th \
  --model_file_dir /path/to/audio-separator-models
```

### Python API

```python
from audio_separator.separator import Separator

# Initialize with model directory
separator = Separator(
    model_file_dir="/path/to/audio-separator-models",
    output_dir="./output",
    output_format="WAV",       # WAV, FLAC, MP3
    normalization_threshold=0.9,
)

# --- BS-Roformer (best vocal quality) ---
separator.load_model("model_bs_roformer_ep_317_sdr_12.9755.ckpt")
vocals, instrumental = separator.separate("my_song.mp3")
print(f"Vocals saved to:       {vocals}")
print(f"Instrumental saved to: {instrumental}")

# --- MDX23C High Quality ---
separator.load_model("MDX23C-8KFFT-InstVoc_HQ.ckpt")
output_files = separator.separate("my_song.mp3")

# --- ONNX model (CPU-friendly) ---
separator.load_model("UVR-MDX-NET-Inst_HQ_3.onnx")
output_files = separator.separate("my_song.mp3")
```

### Batch Processing

```python
from audio_separator.separator import Separator
import glob

separator = Separator(
    model_file_dir="/path/to/audio-separator-models",
    output_dir="./output",
)
separator.load_model("model_bs_roformer_ep_317_sdr_12.9755.ckpt")

for audio_file in glob.glob("./songs/*.mp3"):
    print(f"Processing: {audio_file}")
    output = separator.separate(audio_file)
    print(f"  → {output}")
```

---

## ⚙️ Advanced Configuration

### Custom Output Settings

```python
separator = Separator(
    model_file_dir="/path/to/audio-separator-models",
    output_dir="./output",
    output_format="FLAC",           # Lossless output
    normalization_threshold=0.9,    # Prevent clipping
    amplification_threshold=0.6,    # Boost quiet outputs
    output_single_stem="Vocals",    # Only save vocals stem
    invert_using_spec=True,         # Better instrumental quality
    sample_rate=44100,
)
```

### GPU vs CPU

```python
# Force CPU (useful when GPU VRAM is limited)
separator = Separator(
    model_file_dir="/path/to/audio-separator-models",
    use_cpu=True,
)

# Use specific GPU
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "0"
separator = Separator(
    model_file_dir="/path/to/audio-separator-models",
)
```

---

## 📊 Model Comparison & Recommendations

| Use Case | Recommended Model | Reason |
|----------|-----------------|--------|
| Best vocal separation | `model_bs_roformer_ep_317_sdr_12.9755.ckpt` | Highest SDR score (12.97 dB) |
| Fast + high quality | `MDX23C-8KFFT-InstVoc_HQ.ckpt` | Great balance of speed & quality |
| CPU-only / no GPU | `UVR-MDX-NET-Inst_HQ_3.onnx` | ONNX runtime, no PyTorch GPU needed |
| Multi-stem (drums/bass) | `955717e8-8726e21a.th` | Demucs HTDemucs multi-stem |
| Lightweight separation | `MDX23C_D1581.ckpt` | Smaller model, faster inference |

---

## 📁 Repository Structure

```
audio-separator-models/
├── model_bs_roformer_ep_317_sdr_12.9755.ckpt   # BS-Roformer weights
├── model_bs_roformer_ep_317_sdr_12.9755.yaml   # BS-Roformer config
├── MDX23C-8KFFT-InstVoc_HQ.ckpt               # MDX23C HQ weights
├── MDX23C_D1581.ckpt                           # MDX23C D1581 weights
├── UVR-MDX-NET-Inst_HQ_3.onnx                 # ONNX model
├── 955717e8-8726e21a.th                        # HTDemucs weights
├── htdemucs.yaml                               # HTDemucs config
├── model_2_stem_061321.yaml                    # MDX23C D1581 config
├── model_2_stem_full_band_8k.yaml              # MDX23C HQ config
├── mdx_model_data.json                         # MDX model metadata
├── vr_model_data.json                          # VR model metadata
└── download_checks.json                        # Download integrity checks
```

---

## 🔗 Related Projects

- [python-audio-separator](https://github.com/nomadkaraoke/python-audio-separator) — The main library these models are designed for
- [Ultimate Vocal Remover (UVR)](https://github.com/Anjok07/ultimatevocalremovergui) — GUI application using similar models
- [Demucs](https://github.com/facebookresearch/demucs) — Facebook Research source separation

---

## 📄 License

Model weights are provided under their respective original licenses. Please refer to the original repositories for each model's license terms.

- **BS-Roformer** — [lucidrains/BS-RoFormer](https://github.com/lucidrains/BS-RoFormer)
- **MDX23C / MDX-NET** — [UVR-MDX-NET](https://github.com/kuielab/mdx-net)
- **HTDemucs** — [Facebook Research Demucs](https://github.com/facebookresearch/demucs) (MIT)

---

## 🤝 Contributing

If you have additional pre-trained model weights that work with `audio-separator`, feel free to open a Pull Request or Issue!

---

<p align="center">Made with ❤️ for the audio source separation community</p>
