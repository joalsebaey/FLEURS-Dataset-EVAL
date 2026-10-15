# 🎙️ stt-eval-kit

**How good is Whisper on Egyptian Arabic, really?** This kit measures it on [FLEURS `ar_eg`](https://huggingface.co/datasets/google/fleurs) under clean and nasty conditions (phone line, noise, echo, fast speech) and gives you numbers you can actually trust: WER/CER with confidence intervals, error types, hallucination flags, and speed.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/stt-eval-kit/blob/main/notebook/stt_eval_colab.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## Quickstart (Colab, no setup)

1. Click the **Open in Colab** badge above.
2. `Runtime → Change runtime type → GPU`.
3. In step 1, set `REPO_URL` to your repo URL and run all cells in order.

First results arrive in a few minutes with the default 100-sample run.

## How the notebook works

| Step | What it does |
|---|---|
| **0. GPU check** | Stops early if no GPU is attached. |
| **1. Get the code** | Clones this repo into `kit/` and moves into `src/`. No `REPO_URL`? It asks you to upload a zip of the repo instead. |
| **2. Install** | Installs the few packages Colab doesn't ship with. `datasets<4` avoids a `torchcodec` issue. |
| **3. Unit tests** | Checks the metric code itself before you trust any number. |
| **4. Quick run** | 100 random utterances, `whisper-small`, all conditions. |
| **5. Full run** *(optional)* | Whole test split, `whisper-small` vs `whisper-large-v3-turbo`. |
| **6. Results** | Prints summary tables and shows the plots. |
| **7. Download** | Zips `outputs/` and downloads it. |

## Run locally

```bash
git clone https://github.com/YOUR_USERNAME/stt-eval-kit.git
cd stt-eval-kit
pip install -r requirements.txt
cd src
python -m pytest -q tests
python run_eval.py --max-samples 100
```

A GPU is strongly recommended for anything beyond the quick run.

### Handy flags

| Flag | Meaning |
|---|---|
| `--models a b` | One or more Hugging Face Whisper models (same samples and settings for each). |
| `--max-samples N` | Number of utterances. `0` = the full split. |
| `--conditions clean telephone` | Run only these conditions. |
| `--noise-dir DIR` | Add real-noise conditions from a folder of audio files (e.g. MUSAN, DEMAND). |
| `--batch-size N` | Lower it if you run out of GPU memory. |
| `--no-auto-language` | Skip the language-detection condition. |
| `--seed`, `--bootstrap` | Reproducibility seed and number of bootstrap resamples. |

## Conditions tested

`clean` · `white_20dB` · `white_10dB` · `white_5dB` · `telephone` (8 kHz, µ-law) · `speed_0.85` · `speed_1.15` · `low_volume_-25dB` · `reverb_0.5s` · `clean_auto_language` (no forced language) · `real_noise_{20,10,5}dB` (with `--noise-dir`)

## What you get

Everything lands in `outputs/<model>/`:

| File | Contents |
|---|---|
| `summary_by_condition.csv` | The main table: WER, CER, 95% CI, sub/del/ins rates, RTF, flag rates. |
| `utterances_<condition>.csv` | Per-utterance scores, flags and audio features. |
| `slices_clean.csv` | WER by gender and utterance length. |
| `worst_30_clean.csv` | The 30 worst utterances, for manual review. |
| `top_confusions_clean.csv` | Most frequent substitutions, deletions and insertions. |
| `*_wer_by_condition.png`, `*_wer_vs_duration.png` | Plots. |
| `report.json` | Full config (model, seed, normalization) so results are reproducible. |

## Repo layout

```
├── notebook/   Colab notebook
├── src/        run_eval.py, the stt_eval package, and tests/
├── docs/       Design notes: why each metric and check exists
├── requirements.txt
└── LICENSE
```

## Good to know

- WER depends on text normalization, so always quote it with the result. The config is saved in `report.json`.
- FLEURS is clean, read speech with references close to Modern Standard Arabic. Don't treat the numbers as real Egyptian phone-call performance.
- Failure flags and the wrong-script check are heuristics, not verdicts. Review flagged utterances yourself.

More detail in [docs/design-notes.md](docs/design-notes.md).

## License

[MIT](LICENSE)
