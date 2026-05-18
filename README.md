# 🎙️ Audio Transcriber (faster-whisper)

Local, high-performance transcription of audio and video files into text. Runs directly on your computer, **completely free**, and entirely offline.

---

## 🌟 Features & Capabilities

1. **Batch Processing:** Scans a specified folder (defaults to `C:\Audio`).
2. **Dual-Format Output:** For each processed media file, it generates:
* 📄 `filename.txt` — Clean transcription text (one sentence per line).
* 🎬 `filename.srt` — Fully timed SubRip subtitles compatible with media players and video editors.


3. **Smart Skipping:** Automatically detects and skips files that already have existing transcripts, saving you time. Use the `--overwrite` flag to override this.
4. **100% Private:** Zero cloud dependencies. Your data never leaves your machine.

---

## 📐 System Architecture & Workflow

To help you understand how the components interact, here is a high-level view of the application pipeline:

### Component Interaction

```
┌────────────────────────────────────────────────────────┐
│                      LOCAL HOST                        │
│                                                        │
│  ┌──────────────┐      ┌─────────────┐                 │
│  │ Media Files  │ ───> │   FFmpeg    │                 │
│  │ (MP3, MP4)   │      │ (Demux/PCM) │                 │
│  └──────────────┘      └──────┬──────┘                 │
│                               │                        │
│                               ▼                        │
│                     ┌──────────────────┐               │
│                     │  faster-whisper  │               │
│                     │  (CTranslate2)   │               │
│                     └─────────┬────────┘               │
│                               │                        │
│                ┌──────────────┴──────────────┐         │
│                ▼                             ▼         │
│         ┌─────────────┐               ┌─────────────┐  │
│         │ CPU Execution│               │   GPU/CUDA  │  │
│         │ (Fallback)  │               │ (5x-20x Acc)│  │
│         └─────────────┘               └─────────────┘  │
└────────────────────────────────────────────────────────┘

```

### Script Execution Lifecycle

```
[Start run.bat] ───> Check for Existing .txt/.srt?
                          ├── Yes ───> [Skip File] (Unless --overwrite)
                          └── No  ───> [FFmpeg Audio Extraction]
                                              │
                                              ▼
                                   [faster-whisper Engine]
                                              │
                                              ▼
                                   Generate Output Files:
                                   ├── meeting.txt (Plaintext)
                                   └── meeting.srt (Subtitles)

```

---

## ⚙️ Installation (One-Time Setup)

### 1. Python

Install **Python 3.10 or newer** from [python.org](https://www.python.org/downloads/).

> ⚠️ **CRITICAL:** Make sure to check the box **"Add Python to PATH"** during installation. If missed, the automation scripts (`run.bat`) will fail.

### 2. FFmpeg (Required for Multimedia Formats)

FFmpeg handles audio extraction from formats like MP3, MP4, MKV, etc.

* **The Easy Way (Windows Package Manager):**
Open Command Prompt or PowerShell as Administrator and run:
```cmd
winget install Gyan.FFmpeg

```


*Close and restart your terminal after completion to refresh global PATH variables.*
* **Alternative Manual Method:**
1. Download the build zip from [gyan.dev/ffmpeg/builds](https://www.gyan.dev/ffmpeg/builds/).
2. Extract it to `C:\ffmpeg`.
3. Manually add `C:\ffmpeg\bin` to your system Environment Variables `PATH`.



Verify the installation by running:

```cmd
ffmpeg -version

```

### 3. Application Setup

1. Navigate to the directory containing `transcribe.py`.
2. Double-click **`setup.bat`**.

This script initializes a isolated Python virtual environment (`.venv\`) and installs `faster-whisper` and `tqdm`. This takes roughly 2–5 minutes depending on your internet connection.

### 4. GPU Acceleration (Optional but Recommended)

If you have an NVIDIA GPU, you can achieve a **5–20× performance speedup**. After running `setup.bat`, open your terminal in the project directory and execute:

```cmd
.venv\Scripts\activate.bat
pip install torch --index-url https://download.pytorch.org/whl/cu121

```

> 💡 **Note:** For full GPU acceleration via `faster-whisper`, ensure your system has appropriate NVIDIA CUDA and cuDNN libraries configured.

---

## 🚀 How to Run

### The Simplest Way

1. Place your audio/video files into `C:\Audio`.
2. Double-click **`run.bat`**.
3. *Note on First Run:* The script will download the chosen model file (~3 GB for `large-v3`) from HuggingFace. All subsequent runs are 100% offline.

### Advanced Control (Command Line)

```cmd
.venv\Scripts\activate.bat
python transcribe.py --input "C:\Audio" --model large-v3 --language en

```

---

## 🛠️ Configuration Options

To see all available flags, execute:

```cmd
python transcribe.py --help

```

| Argument | Short | Default | Description |
| --- | --- | --- | --- |
| `--input` | `-i` | `C:\Audio` | Target directory containing media files |
| `--model` | `-m` | `large-v3` | Model tier: `tiny`, `base`, `small`, `medium`, `large-v2`, `large-v3` |
| `--language` | `-l` | `sk` | Audio language code (`en`, `sk`, `cs`, `de`, `auto`...) |
| `--device` | `-d` | `auto` | Execution hardware: `cuda`, `cpu`, or `auto` |
| `--compute-type` |  | `auto` | Precision type quantization: `int8`, `float16`, `float32` |
| `--overwrite` |  | *(Disabled)* | Overwrites existing `.txt` and `.srt` transcripts if found |
| `--no-srt` |  | *(Disabled)* | Disables generation of `.srt` subtitle files |
| `--beam-size` |  | `5` | Higher values slightly increase accuracy but slow down processing |

---

## 📊 Model Comparison & Performance

* **WER** = Word Error Rate (lower numbers mean higher accuracy).
* For non-English languages (e.g., Slovak, Czech), we **strongly recommend** using at least the `medium` or `large-v3` models. Smaller models are heavily optimized for English.

| Model Size | Disk Space | RAM Required | Accuracy (Slovak WER) | Execution Speed (CPU) |
| --- | --- | --- | --- | --- |
| **tiny** | 75 MB | 1 GB | ~35% (Poor) | ~30× realtime |
| **base** | 145 MB | 1 GB | ~25% | ~16× realtime |
| **small** | 480 MB | 2 GB | ~18% | ~6× realtime |
| **medium** | 1.5 GB | 5 GB | ~12% | ~2× realtime |
| **large-v3** | 3.0 GB | 10 GB | **~8% (Best)** | ~1.5× realtime |

### Benchmark Examples (1 Hour of Audio)

* **8-Core CPU:** `tiny` (~2 mins) | `medium` (~25 mins) | `large-v3` (~45 mins)
* **NVIDIA RTX 3060 GPU:** `large-v3` transcribes 1 hour of audio in approximately **8 minutes**.

---

## 📂 Supported Formats & Outputs

### Input Formats

Accepts any multimedia format readable by your local FFmpeg installation:

* **Audio:** `.mp3`, `.wav`, `.m4a`, `.flac`, `.ogg`, `.opus`, `.wma`, `.aac`
* **Video:** `.mp4`, `.mkv`, `.mov`, `.avi`, `.webm`, `.wmv`, `.flv` *(Automatically extracts audio track)*

### Output Formats

For an input file named `meeting.mp3`, the script outputs:

#### 1. `meeting.txt`

Continuous prose, split line-by-line per sentence. Perfect for reading documentation.

#### 2. `meeting.srt`

Structured subtitle file with precise syntax time-blocks:

```srt
1
00:00:00,000 --> 00:00:04,500
Hello, welcome to today's training session.

2
00:00:04,500 --> 00:00:09,200
Today we will cover advanced system configurations.

```

An execution log named `transcribe.log` will also be written to the root folder for debugging.

---

## 🔍 Troubleshooting

* **Error: `ffmpeg not found**`
* FFmpeg is either not added to your system PATH, or you forgot to open a completely new terminal window after installing it.


* **Processing speed is incredibly slow on CPU**
* Switch to a lighter model (e.g., `--model medium` or `small`), or configure NVIDIA GPU acceleration.


* **Poor text quality / Hallucinations**
* Ensure you are using the `--model large-v3` flag. Smaller models drop drastically in quality for non-English audio.


* **Out of Memory (OOM) error on GPU**
* Drop your precision configuration by passing `--compute-type int8_float16` instead of `float16`, or drop down a model size tier.


* **Misspelled specialized technical terminology/names**
* Whisper base models are trained on generalized global datasets. Corporate acronyms, specific medical/legal jargon, or rare last names may require a quick find-and-replace script post-transcription.



---

## 🔒 Data Privacy Notice

Everything runs locally. No audio data, generated text, or metadata is ever transmitted to external servers or cloud APIs. The software requires an internet connection **solely on its initial execution** to pull down the open-source weight layers directly from HuggingFace. All subsequent runs can be executed with your network completely disconnected.
