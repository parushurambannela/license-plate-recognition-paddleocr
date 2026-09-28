# Car License Plate Detection & Recognition — PP-OCRv5 vs PP-OCRv4

Two Google Colab notebooks fine-tuning and comparing PaddleOCR's `PP-OCRv5_server_rec` and
`PP-OCRv4_server_rec` text-recognition models on license plates.

Repo: https://github.com/parushurambannela/license-plate-recognition-paddleocr

## Files

- `PPOCR_License_Plate_Training.ipynb` — dataset prep, fine-tuning, evaluation, export, publishes
  the trained models as a GitHub Release.
- `PPOCR_License_Plate_Inference.ipynb` — downloads the published models, runs full
  detection+recognition inference, annotated visualizations, side-by-side comparison, batch
  stats, error analysis.
- `images/` — 200 real car photos + `labels.csv` (ground-truth plate numbers), checked directly
  into the repo.

## Dataset

200 real photos of cars with known plate numbers (matched from `number_plate.xlsx` by
filename). Since these are full photos rather than pre-cropped plates, the training notebook
runs PaddleOCR's pretrained pipeline once per photo and keeps whichever detected text box's
recognized text is closest to the known ground truth — that crop plus the real label becomes a
training example. No manual bounding-box annotation needed.

- 170 photos -> auto-cropped -> fine-tuning set (80/20 train/val split).
- 30 photos -> held out untouched (full photos, never touched during training) -> published for
  the inference notebook to test detection + recognition together.

## Models & approach

- Only the **recognition** models are fine-tuned (`PP-OCRv5_server_rec`, `PP-OCRv4_server_rec`).
  Detection uses the pipeline's default detector in both cases, so the recognizer is the only
  variable being compared.
- Training uses the classic PaddleOCR repo (`tools/train.py` / `tools/eval.py` /
  `tools/export_model.py`) against the official `configs/rec/PP-OCRv5/PP-OCRv5_server_rec.yml`
  and `configs/rec/PP-OCRv4/PP-OCRv4_server_rec.yml` configs, with dataset paths/epochs/batch
  size edited in Python (via PyYAML) rather than passed as command-line overrides.

### Getting data between the two notebooks, without Drive or manual uploads

- **Input dataset** (`images/`, small): lives directly in this repo. The training notebook just
  does `git clone` at the top and reads `images/labels.csv`.
- **Trained model weights** (large — a couple hundred MB combined, over GitHub's 100MB per-file
  limit for plain git): the training notebook zips the two exported models plus the 30 held-out
  test photos and publishes them as assets on a GitHub Release (tag `trained-models`) of this
  same repo, using the GitHub REST API + a personal access token pasted in at runtime. The
  inference notebook downloads those assets with a plain `wget` — no auth needed since the repo
  is public. Re-running the training notebook replaces the previous release with the newly
  trained weights.

## How to run

1. Open `PPOCR_License_Plate_Training.ipynb` in Colab, set runtime to a T4 GPU, run all cells.
   Section 9 will ask for a GitHub [personal access token](https://github.com/settings/tokens)
   (classic, `repo` scope) to publish the trained models.
2. Open `PPOCR_License_Plate_Inference.ipynb` in Colab and run all cells — it downloads the
   release published in step 1, so run the training notebook first.

## Reference

[PaddleOCR official documentation](https://www.paddleocr.ai/main/en/index.html)
