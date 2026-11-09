# NemoraSonar Phonk 1.1

Tells whether a track is **Brazilian Phonk** and tags its micro-styles: runs locally on CPU, in a second or two per track.
Based on [EfficientAT](https://github.com/fschmid56/EfficientAT) (`mn10_as`).

For each track the model returns:

| Output | Meaning |
|---|---|
| `genre` | Brazilian Phonk or Other Music, Other Music comes with no styles  |
| `styles` | The styles the track fully is, strongest first |
| `related_styles` | Styles it only leans towards, strongest first |
| `loose_styles` | Loose categories whose tracks need not sound alike (e.g. Experimental) |

Empty lists are a normal answer: they mean the model recognised nothing with confidence.

## Download

This repository holds only the documentation. The files are attached to the [latest release](../../releases/latest):

| File | What's inside |
|---|---|
| `nemorasonar-phonk-1.1.zip` | The model (`model.onnx`, 14 MB), its metadata and `predict.py` |
| `nemorasonar-phonk-1.1-training-code.zip` | The annotation site and the training pipeline |

## Quick start

Requires Python and `ffmpeg` on PATH. Unzip the model archive, then run this inside its folder:

```
pip install -r requirements.txt
python predict.py track.mp3
```

`model.onnx` is 14 MB and includes the mel frontend. Input: mono 32 kHz audio cut into 10-second windows every 5 seconds, all windows of a track in one batch.
Measured on a desktop CPU, a 110-second track took about 0.9 s in the model plus 0.5 s to decode the audio.

## How it was made

- **Encoder:** EfficientAT `mn10_as` (MobileNetV3, about 3M parameters, pretrained on ImageNet and AudioSet), frozen.
- **Head:** attention pooling over the windows, then per-style outputs and a ranking output for the order of styles (about 0.4M parameters).
- **Data:** 710 Brazilian Phonk tracks annotated by hand by one person, 82 styles, plus 300 tracks of other music checked by the same person. Slowed and sped-up variants (0.8x, 1.2x) were used as augmentation.
- **Thresholds:** Style 0.25, Related 0.5, Loose 0.5.

## Evaluation

5-fold cross-validation grouped by composition, so versions of one song never sit on both sides of a split. The released model was then trained on all tracks.

**Is it Brazilian Phonk?** Correct on 99.4% of Brazilian Phonk tracks and 99.0% of other music. On 200 further tracks from a personal library, never used in training, it called 198 other music.

Styles, on Brazilian Phonk tracks only:

| Output | Precision | Recall | F1 |
|---|---|---|---|
| Style | 0.840 | 0.678 | 0.750 |
| Related | 0.711 | 0.235 | 0.354 |
| Loose (25 marks) | 0.900 | 0.360 | 0.514 |

Style macro-F1 0.649. The first style is correct on 0.622 of tracks. On average 0.180 wrong styles per track.

**Blind checks.** While the dataset was being built, earlier versions of the model were run on batches of new tracks before those tracks were annotated, then scored against the annotator's labels. Across 17 batches they found 234 of 424 known styles (55%), with 60 wrong styles (about 80% precision). These numbers are for the earlier versions, not this exact model.

Per-style counts and scores are in [styles.md](styles.md).

## Limitations

- Styles exist for Brazilian Phonk only. The other-music examples came from one person's library, so unusual genres may be misjudged, and music close to phonk is the hardest case.
- Styles reflect one person's ear and taxonomy, and many are named after producers. The model partly learns a producer's sound, and is weaker on producers it has not heard.
- Some styles have only a handful of tracks, and those are unreliable. Several have F1 0 (see styles.md).
- It more often stays silent than invents a style: recall is much lower than precision.
- The numbers in `style_probs` are near 0 or 1 and are not calibrated probabilities.

## License

The code and model are released under the [MIT License](LICENSE), and include EfficientAT, MIT License, Copyright (c) 2022 Florian Schmid ([THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md)).

The encoder weights were trained on AudioSet (audio from YouTube), and this model on commercially released music. The training audio is not distributed here. The legal status of models trained on such data is not settled, check it yourself before commercial use.

## Citation

If you use this model, please also cite EfficientAT:

> F. Schmid, K. Koutini, G. Widmer. *Efficient Large-scale Audio Tagging via Transformer-to-CNN Knowledge Distillation.* ICASSP 2023.

## Training code

`nemorasonar-phonk-1.1-training-code.zip` holds the annotation site and the training pipeline. To use it, clone EfficientAT into `third_party/EfficientAT` and copy `config/project.example.json` to `config/project.json`.
