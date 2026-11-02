# Historical results and interpretation

All figures below are taken from saved notebook outputs. Training was not rerun during packaging.

## Speech experiment

- Four-corpus collection: 12,162 accepted clips; random 70/15/15 clip-level split with `random_state=42`.
- Wav2Vec2 base encoder, mean pooling, hidden linear layer, GELU, dropout 0.1, and seven-class output.
- AdamW learning rate 3e-5, weight decay 0.01, batch size 16, early-stopping patience 6.
- Test support: 1,825 clips. Rounded accuracy 0.83, macro F1 0.84, weighted F1 0.83.

| Emotion | Precision | Recall | F1 | Test support |
| --- | --- | --- | --- | --- |
| Anger | 0.93 | 0.81 | 0.86 | 300 |
| Disgust | 0.76 | 0.84 | 0.80 | 276 |
| Fear | 0.83 | 0.70 | 0.76 | 288 |
| Happiness | 0.85 | 0.85 | 0.85 | 271 |
| Neutral | 0.83 | 0.96 | 0.89 | 288 |
| Sadness | 0.75 | 0.75 | 0.75 | 296 |
| Surprise | 0.92 | 0.98 | 0.95 | 106 |

Speakers may appear in several splits, so these values do not measure generalization to unseen speakers. Labels are derived from filenames, with fuzzy matching for several corpus labels. Audit the mapping before drawing cross-corpus conclusions.

The loop saves its best validation checkpoint but does not reload it before final testing. Its scheduler is budgeted for 20 epochs while the training loop allows 50. Attention masks are not passed through training/evaluation, and mean pooling is not padding-aware. These issues should be resolved in a new benchmark.

## Text experiment

The report describes 40,000 tweets with 13 original labels merged into six categories. Text is cleaned and tokenized, padded to 500 tokens, embedded with frozen GloVe 50d vectors, and passed through a two-layer Transformer prototype. Training uses AdamW, an 80/20 split, and validation-based early stopping.

The saved run ends with `KeyboardInterrupt`. A saved validation loss of 1.6068 at epoch 6 is visible, but no completed independent test evaluation is provided. The 20% split named `test` in the code is used repeatedly for validation.

**Implementation issue:** tensors are passed as `[batch, tokens, embedding]`, but the encoder and positional encoding assume sequence-first layout. Attention therefore operates over the batch dimension. Training specifies 10 heads while bot reconstruction defaults to 5. Existing results must not be presented as a validated Transformer text classifier; correct the layout and matching model configuration, then retrain.

## Telegram prototype

The bot has text and voice handlers. Voice messages are converted to WAV and transcribed with Google speech recognition; the transcript is then classified by the text model. There is no integration with the acoustic Wav2Vec2 classifier. A hard-coded bot credential was removed during packaging and replaced by an environment-variable lookup.
