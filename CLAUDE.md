# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Official course materials for **MIT Introduction to Deep Learning (6.S191)** (introtodeeplearning.com). It is primarily a collection of Jupyter/Colab notebooks (the labs), backed by a small companion Python package, `mitdeeplearning`, that is published to PyPI and imported by the notebooks as `mdl`.

There is no application server, build pipeline, or CI here — the "product" is the notebooks plus the `mitdeeplearning` support package.

## Repository layout

- `lab1/`, `lab2/`, `lab3/` — the current course labs, each a self-contained set of notebooks:
  - `TF_*.ipynb` / `PT_*.ipynb` prefixes distinguish the TensorFlow vs. PyTorch version of the same lab.
  - `solutions/` — completed reference notebooks matching the student notebooks (same name + `_Solution` suffix).
  - `img/` — images embedded in the notebooks.
  - `README.md` — per-lab description of what the lab covers.
- `mitdeeplearning/` — the `mitdeeplearning` pip package (imported as `import mitdeeplearning as mdl` in notebooks). Contains helper code, datasets, and grading/test utilities that notebooks call into rather than reimplementing inline:
  - `lab1.py` — Irish folk music (ABC notation) loading, batching test helpers, ABC→WAV conversion (via bundled `bin/abc2wav`), and `test_*` assertion helpers used by lab1's TODO cells.
  - `lab2.py` — `TrainingDatasetLoader` (faces dataset, from the `data/faces/*` images), image-prediction plotting helpers for lab2 (MNIST/CNN + debiasing VAE labs).
  - `lab3.py` — LLM fine-tuning helpers: `create_dataloader` (Dolly-15k dataset + text-style datasets under `data/text_styles/`), `LLMClient` (thin OpenRouter/OpenAI-compatible chat wrapper).
  - `lab3_old.py` — legacy RL (Pong) helpers, kept for the `xtra_labs/rl_pong` notebook.
  - `util.py` — framework-agnostic (TF and PyTorch) plotting/training utilities: `PeriodicPlotter`, `LossHistory`, `plot_sample` (handles both `backend='tf'` and `backend='pt'`), `create_grid_of_images`, `display_model`.
  - `data/` — bundled datasets (`irish.abc`, `faces/{DF,DM,LF,LM}/*.png`, `text_styles/*.txt`) shipped as package data.
  - `bin/abc2wav` — bundled binary used by `lab1.abc2wav` to render ABC notation to WAV audio.
- `xtra_labs/` — supplementary/optional labs not part of the core curriculum (`llm_finetune`, `rl_pong`, `rl_selfdriving`, `uncertainty`). Some contents are drafts (e.g. `llm_finetune/NOT_FINAL`, `draft.py`).
- `setup.py` / `setup.cfg` — packaging config for publishing `mitdeeplearning` to PyPI.
- `test.py` — an ad hoc manual smoke-test script for the lab1 music pipeline (loads a song, converts to WAV, drops into `pdb`); not a pytest suite.

## How the labs and the package relate

Every notebook installs the published PyPI package at the top of the notebook (`!pip install mitdeeplearning --quiet`) rather than using the local `mitdeeplearning/` source directly. This means:

- **Changes to `mitdeeplearning/*.py` do not take effect in Colab/notebooks until a new version is released to PyPI** (bump `version` in `setup.py`, build, and upload). Keep this in mind when asked to "fix a bug used in the notebook" — the fix belongs in `mitdeeplearning/`, but it won't be exercised by a fresh Colab run until republished.
- Each notebook has a `TF_` or `PT_` prefix indicating TensorFlow vs. PyTorch. When editing one, check whether the parallel notebook (same lab, other framework) needs the equivalent change, and whether the corresponding notebook in `solutions/` also needs updating (solutions are complete versions of the same notebook with the `#TODO` blocks filled in).
- Notebooks contain a standard MIT copyright header cell and an "Open in Colab" badge cell at the top — preserve these when editing notebook content.
- Student notebooks intentionally contain `#TODO` blocks left for students to fill in; solution notebooks are the corresponding completed version. Don't fill in TODOs in student notebooks unless explicitly asked to do so (that would defeat the point of the exercise).

## Working with notebooks

Notebooks are `.ipynb` JSON files. Prefer the `NotebookEdit` tool over hand-editing JSON. When inspecting notebook content from the shell/Python, parse with `json` rather than treating it as plain text (cell source is stored as a list of strings).

## Development

There is no build/lint/test tooling configured (no linter config, no test framework, no CI workflow in this repo). Practically:

- To sanity-check `mitdeeplearning` package code changes, run relevant functions directly, e.g. `python test.py` exercises the lab1 music-loading → ABC → WAV pipeline (requires `mitdeeplearning` importable, e.g. `pip install -e .` from repo root).
- Framework-dependent code paths (TensorFlow vs. PyTorch) in `mitdeeplearning/util.py` and `lab2.py` are selected via a `backend`/`channels_last` argument rather than separate modules — when touching these, verify both branches (`backend='tf'` and `backend='pt'`) still behave consistently, matching whichever notebook prefix (`TF_`/`PT_`) exercises them.
- Installing the package locally for iteration: `pip install -e .` (uses `setup.py`; dependencies include `tensorflow`, `torch`-adjacent libs are expected to already be present since `setup.py` doesn't declare `torch` explicitly, plus `comet_ml`, `gym`, `opik`, `openai`, `transformers`, `datasets`, `peft`, `lion-pytorch`).

## Licensing convention

All code/content is © MIT Introduction to Deep Learning, MIT-licensed but with a required attribution notice ("© MIT Introduction to Deep Learning, http://introtodeeplearning.com") on reuse outside the course. New notebook files should carry the same copyright header cell as existing notebooks (see any `lab*/*.ipynb` second cell), and the year in headers/README should track the current course year (currently 2026) — check `README.md` and notebook headers for the year in use before adding new dated references.
