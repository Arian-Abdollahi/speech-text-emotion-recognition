# Dataset and embedding setup

## Speech

Source used by the notebook: https://www.kaggle.com/datasets/dmitrybabko/speech-emotion-recognition-en

Download through your own account and extract to `data/speech-emotion-recognition-en/`, or set `SPEECH_DATA_DIR`. The notebook expects the downloaded collection's CREMA-D, RAVDESS, SAVEE, and TESS directory layout. Inspect the parser paths if your download uses another layout. Labels are inferred from corpus filenames and mapped to seven categories. The saved run retained 12,162 clips.

Split tables are written to `artifacts/speech_splits/`. Their absolute audio paths are local generated files, not committed data.

## Text

The supplied source used a Kaggle input directory named `emotion-detection-from-text` and a file named `tweet_emotions.csv`, with `content` and `sentiment` columns. The archive does not identify the exact dataset publisher or version. Obtain the matching 40,000-tweet, 13-label collection under its own terms and place it at `data/text/tweet_emotions.csv`, or set `TEXT_DATA_PATH`. The notebook merges the labels into six classes; this is not the same label scheme as the speech model.

## Word embeddings

Download the Stanford GloVe 6B collection from https://nlp.stanford.edu/projects/glove/ and put `glove.6B.50d.txt` in `data/glove/`, or set `GLOVE_PATH`.

## External access

The speech notebook loads `facebook/wav2vec2-base` through Hugging Face: https://huggingface.co/facebook/wav2vec2-base . Initial access requires network availability or a populated local cache.

The text preprocessing uses NLTK tokenizers and lemmatizers. Install required resources locally, for example:

```bash
python -m nltk.downloader punkt punkt_tab wordnet omw-1.4 stopwords
```

The optional bot needs the system `ffmpeg` executable for voice format conversion. No raw audio, tweets, embeddings, credentials, or trained weights are stored here. Respect each dataset's license and privacy terms.
