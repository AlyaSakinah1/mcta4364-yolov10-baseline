# Dataset: COCO val2017

This document describes the data used to reproduce the YOLOv10-N baseline (MCTA 4364, Track 2).

## 1. Source and licence

| Item | Detail |
| ---- | ------ |
| Dataset | Microsoft COCO 2017 (Common Objects in Context), **validation split (val2017)** |
| Images | http://images.cocodataset.org/zips/val2017.zip |
| Labels used | `coco2017labels.zip` from the Ultralytics assets release (COCO annotations converted to YOLO text format) |
| Website | https://cocodataset.org |
| Why this dataset | The YOLOv10 paper reports its results on COCO val2017, so using the same data lets us compare our numbers directly with the paper's. |

**Licence and terms of use**
- COCO annotations are released under a Creative Commons Attribution 4.0 licence (CC BY 4.0).
- The images come from Flickr and each image keeps its own Flickr terms of use. COCO does not own the images.
- We use the data only for academic coursework and do not redistribute the images in this repository. They are downloaded by the notebook at run time.
- Please confirm the current wording on https://cocodataset.org/#termsofuse.

## 2. Size, classes and format

| Property | Value |
| -------- | ----- |
| Number of images | 5,000 (4,952 with at least one object, 48 background images with no labels) |
| Number of labelled instances | 36,335 (as counted from the YOLO label files by Ultralytics) |
| Number of classes | 80 object categories (person, car, cat, chair, book, ...) |
| Image format | JPEG; most images are roughly 640 x 480 pixels (original size varies) |
| Label format | One `.txt` file per image; each line is `class x_center y_center width height`, normalised to 0-1 |
| Task | Object detection (bounding boxes) |

## 3. Train / validation / test split

This is a **Track 2 reproduction of a pretrained model**, so we do not train anything. We evaluate the pretrained `yolov10n.pt` checkpoint on the standard COCO validation set.

| Split | Used? | Notes |
| ----- | ----- | ----- |
| Train (train2017) | No | The pretrained weights were already trained on COCO train2017 by the Ultralytics/YOLOv10 authors. We did not retrain. |
| Validation (val2017) | **Yes**, all 5,000 images | Used for all quantitative results and qualitative examples. |
| Test (test2017) | No | COCO test labels are not public (evaluation requires the COCO server). |

**How the data is set up in the notebook**
- The Ultralytics `coco.yaml` is copied to `coco_val.yaml`.
- `path` is set to `/content/datasets/coco` and `val` to `val2017.txt`.
- `train` is set to `val2017.txt` only as a **placeholder** because the config requires the field. It is never used because we only call `model.val()`.
- The `download` field is removed so nothing is downloaded automatically.

## 4. Preprocessing and augmentation

| Step | Detail |
| ---- | ------ |
| Resize | Letterbox resize to 640 x 640 (keeps aspect ratio, pads the rest), `imgsz=640` |
| Normalisation | Pixel values scaled to 0-1 by the Ultralytics pipeline |
| Batch size | 32 |
| Confidence threshold (qualitative examples) | 0.25 |
| Augmentation | **None.** Augmentation is a training-time step, and we only evaluate. |

## 5. Sample images and basic analysis

- **Sample images:** `figures/sample_images.png` shows ground-truth boxes on val2017 images.
- **Class distribution:** see `notebooks/02_eda.ipynb` for the instances-per-class chart and the box-size (small / medium / large) distribution.

Observed from the evaluation output:
- **Strong class imbalance.** `person` has 10,777 instances (2,693 images), while rare classes are tiny, for example `hair drier` has 11 instances (9 images) and `toaster` has 9 images.
- **Small-object difficulty.** COCO AP is 20.2 for small objects, 43.5 for medium and 55.8 for large.
- **Hard categories.** Classes such as `book` (12.1 AP), `backpack` (10.7), `handbag` (10.7) and `spoon` (11.2) score lowest, while `bear` (71.8), `cat` (70.2) and `airplane` (69.3) score highest.
- **Crowded and cluttered scenes.** Some images contain many instances of one class (for example 13 books in one image, where the model detected none), which makes annotation and detection harder.

## 6. Limitations

1. **No training or retraining.** Results reflect the pretrained checkpoint only. We cannot say how the model behaves when trained on a different split.
2. **Evaluation only.** We have no held-out test set, so val2017 is used both for reporting and for choosing the qualitative examples. Examples were picked by hand and may not represent the average case.
3. **Class imbalance.** Rare classes have very few instances, so their per-class AP is noisy (for example `hair drier` AP is 0.3 on 11 instances).
4. **Label conversion.** We use YOLO-format labels converted by Ultralytics, so the instance count can differ slightly from the official COCO annotation file.
5. **Image licensing.** Images are not ours to redistribute, so the repository stores only a handful of example figures, not the dataset.

## 7. References

- T.-Y. Lin et al., "Microsoft COCO: Common Objects in Context", ECCV 2014. https://arxiv.org/abs/1405.0312
- COCO dataset website: https://cocodataset.org
- Ultralytics: https://github.com/ultralytics/ultralytics
