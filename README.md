# YOLOv10: Real-Time End-to-End Object Detection (Baseline Reproduction)

**Course:** MCTA 4364 Machine Vision, Semester 1, 2026/2027
**Track:** 2 (Paper)

## Members
| Name | Matric no. |
|---|---|
| Nur Alya Sakinah | 2319444 |
| Arifa Aqilah | 2311530 |
| [Member 3] | [Matric] |

## Source
- Paper: Wang et al., "YOLOv10: Real-Time End-to-End Object Detection", NeurIPS 2024. https://arxiv.org/abs/2405.14458
- Official code: https://github.com/THU-MIG/yolov10
- We ran the YOLOv10 implementation and weights from the Ultralytics package (https://github.com/ultralytics/ultralytics), not the official repository's own code, because its requirements.txt pins torch==2.0.1, which cannot be installed on current Google Colab (Python 3.13).

## What we wrote vs reused
- Reused: YOLOv10-N architecture and pretrained weights (yolov10n.pt), the Ultralytics package, COCO val2017 data and labels.
- Written by us: the evaluation notebook, per-class analysis, qualitative examples and figures.

## Environment
- Google Colab, Tesla T4 GPU
- Python 3.13, PyTorch 2.14.1, Ultralytics 8.4.220 (see requirements.txt)

## Folder structure
- `notebooks/` : YOLOv10_baseline.ipynb
- `figures/` : result and example images
- `requirements.txt`
- `README.md`

## How to run the baseline
1. Open `notebooks/YOLOv10_baseline.ipynb` in Google Colab.
2. Runtime → Change runtime type → T4 GPU.
3. Run all cells in order (installs Ultralytics, downloads COCO val2017 and labels, evaluates yolov10n.pt at 640 px, batch 32).

## Baseline results (COCO val2017)
| Metric | Paper | Ours |
|---|---|---|
| AP (mAP50-95) | 38.5 | 39.5 |
| AP50 | n/a | 55.0 |
| Small / medium / large AP | n/a | 20.2 / 43.5 / 55.8 |
| Inference time | 1.84 ms (TensorRT) | 2.9 ms (PyTorch, T4) |

## AI tool use
We used Claude (Anthropic) to help with Colab setup, evaluation code and report drafting. All experiments were run by the group.
