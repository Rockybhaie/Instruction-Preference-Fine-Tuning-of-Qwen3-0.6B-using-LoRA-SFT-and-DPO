# NLP Assignment 04 — Track 1, Option A (SFT → DPO)

Instruction tuning and preference fine-tuning of **Qwen3-0.6B-Base** for instruction
following, evaluated with **BLEU + BERTScore** on a 10-prompt manual test set.

## Pipeline

```
Qwen3-0.6B-Base
   ├── 00_baseline_eval.ipynb   → baseline BLEU + BERTScore on 10 prompts
   ├── 01_sft_trials.ipynb      → 5 SFT LoRA trials → pick best SFT
   └── 02_dpo_trials.ipynb      → merge best SFT → 5 DPO LoRA trials → pick best
```

| Stage | Dataset | Subset |
|-------|---------|--------|
| SFT   | `HuggingFaceH4/ultrachat_200k`        | ~4k train / 500 val |
| DPO   | `HuggingFaceH4/ultrafeedback_binarized` | ~4k train / 500 val |
| Eval  | `test_set.json` (self-written)        | 10 prompts |

## How to run on Kaggle

1. Create a new Kaggle Notebook, **Settings → Accelerator → GPU T4 x2** (or P100),
   and **turn Internet ON** (needed to download models/datasets/pip packages).
2. Upload `test_set.json` as a Kaggle Dataset (or rely on the inline copy embedded
   in each notebook — both work).
3. Run the notebooks **in order**:
   - `00_baseline_eval.ipynb`
   - `01_sft_trials.ipynb` — at the end it prints/saves the **best SFT config** and
     saves the best adapter to `/kaggle/working/best_sft_adapter`.
   - `02_dpo_trials.ipynb` — set `BEST_SFT_CONFIG` near the top to the winner from
     notebook 01 (or attach notebook 01's output as an input dataset to reuse the
     saved adapter). It then runs the 5 DPO trials.
4. After each notebook, **Save Version** so outputs persist, then download the
   emitted result files (see below) and send them back to finalize the report.

## Result files each notebook emits (for the report)

- `baseline_results.csv` — per-prompt + mean BLEU/BERTScore for the base model.
- `sft_results.csv` — per-trial BLEU/BERTScore + val loss for all 5 SFT trials.
- `dpo_results.csv` — per-trial BLEU/BERTScore + val loss for all 5 DPO trials.
- `sample_outputs.md` — example generations (base vs SFT vs DPO) for the report.
- `best_sft_config.json` — the selected best SFT hyperparameters.

## Reproducibility

See `requirements.txt` for pinned package versions. All trials use a fixed seed and
identical greedy decoding (`do_sample=False`, fixed `max_new_tokens`) so base / SFT /
DPO are compared on equal footing.
