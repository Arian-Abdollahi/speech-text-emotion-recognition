# Reproduction notes

## Status

This repository preserves supplied research experiments and historical outputs. It does not include datasets, GloVe embeddings, trained weights, a saved tokenizer, or an exact original environment lock. No new training or Telegram session was run during packaging.

## Environments and run order

Use separate Python 3.11 virtual environments for `requirements-speech.txt` and `requirements-text.txt`. Legacy bot dependencies are listed separately in `requirements-bot-legacy.txt`: their old Tornado dependency can conflict with modern Jupyter. Port or isolate that stack before using the bot. Dependency ranges are inferred compatibility suggestions; GPU training and dependency resolution still need validation on the target machine. Match PyTorch and torchaudio builds.

Launch Jupyter from the repository root or `notebooks/`. Download datasets first; see `data/README.md`.

- Speech: run `01_speech_emotion.ipynb` in order. It writes split CSVs and `artifacts/speech_best_model.pt`.
- Text: inspect `02_text_emotion_and_bot.ipynb`, correct the known model issues before meaningful training, then generate `artifacts/text_best_model.pt` and `artifacts/tokenizer.pkl`.
- Bot: after compatible training, set `TELEGRAM_BOT_TOKEN` and `RUN_TELEGRAM_BOT=1` in the process environment before running its final cell. No real token belongs in a notebook or commit. `.env.example` documents variable names but is not automatically loaded.

In the project-oriented text notebook, bot libraries are imported only when bot startup is explicitly enabled. The historical original still imports them earlier. `ffmpeg` is additionally needed for voice conversion. Transcription uses an external Google service.

## Packaging changes

- Added project-oriented notebook copies with repository-relative data/artifact paths and optional environment overrides.
- Commented out inline pip commands and credential-prompting dataset downloads in those copies.
- Separated speech and text checkpoint names to prevent overwriting `best_model.pt` between experiments.
- Cleared outputs only in project-oriented copies; kept historical outputs in `originals/`.
- Replaced the hard-coded Telegram token with `os.environ["TELEGRAM_BOT_TOKEN"]` in both text copies and made startup opt-in. The original text file is explicitly named `text.sanitized.ipynb`.
- Deferred optional bot imports until startup in the project-oriented copy, keeping legacy packages out of the text-training dependency file.
- Removed the blank first page from the report; retained the remaining report body. See `CREDITS.md`.

Model architecture, preprocessing, splitting, training, and historical scores were not silently changed. Original notebooks retain hosted paths and should be consulted as historical evidence rather than run unchanged locally.

## Issues requiring new experiments

1. Correct batch/sequence axes in the text encoder and positional encoding; verify with shape-based tests, then retrain. Use the same head count in training and bot inference. Add a padding mask and validate preprocessing consistency.
2. Give text training a reproducible, stratified train/validation/test split. The current 20% validation partition is named `test`; the saved training run is interrupted.
3. Audit the text stopword option: the nested function shadows its Boolean argument, so stopword removal is applied even when the caller passes `False`. The generic binary metric helper also does not support the six-class task.
4. Audit speech filename labels. Use dataset-qualified speaker IDs to create disjoint speaker groups and evaluate each corpus separately.
5. For speech, pass attention masks, pool over valid frames, reload the best checkpoint, and align scheduler length with the actual epoch budget. Confirm feature-extractor/processor compatibility with the chosen Transformers version.
6. Use unique temporary files, robust failures, and compatible inference artifacts before deploying the Telegram prototype. Do not deserialize untrusted tokenizer pickles.

`VERIFICATION.md` records packaging checks. They are not a claim that full training or deployment has been validated.
