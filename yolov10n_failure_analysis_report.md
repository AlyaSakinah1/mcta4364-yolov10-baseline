# Failure Analysis: YOLOv10n on COCO 2017 Validation

## 1. Executive Summary
While **YOLOv10n** (`yolov10n.pt`) achieves a strong real-time latency of **2.9 ms/image** at `imgsz=640` and an overall validation score of **39.5% AP<sub>50:95</sub>** (**55.0% AP<sub>50</sub>**), performance varies sharply across object scales and semantic categories. 

A joint inspection of the per-class metrics (`results/per_class_ap.csv`) and qualitative failure cases (`figure5_failed.png`) reveals that the nano-scale architecture struggles substantially with **small, densely packed, and heavily occluded objects** (AP<sub>small</sub> = 20.2%) compared to large, distinct foreground subjects (AP<sub>large</sub> = 55.8%).

---

## 2. Quantitative Breakdown: Scale & Class Disparities

### Scale Sensitivity (AP<sub>small</sub> vs. AP<sub>large</sub>)
| Metric / Subset | Score (AP<sub>50:95</sub>) | Performance Gap | Primary Architectural Bottleneck |
| :--- | :---: | :---: | :--- |
| **AP<sub>large</sub>** | **55.8%** | Baseline | High spatial resolution retained in deeper feature maps (`P5`). |
| **Overall AP** | **39.5%** | -16.3% | Balanced trade-off between lightweight backbone and detection head. |
| **AP<sub>small</sub>** | **20.2%** | **-35.6%** | Aggressive downsampling loses fine-grained edge/texture cues. |

### Worst-Performing Classes (`results/per_class_ap.csv`)
The lowest-scoring categories in `results/per_class_ap.csv` directly mirror the scale and occlusion bottlenecks seen in AP<sub>small</sub>:
- **`hair drier` (0.3% AP):** Extreme class rarity in COCO combined with high intra-class shape variance and small pixel footprint.
- **`backpack` (10.7% AP) & `handbag` (10.7% AP):** Typically appear as small secondary accessories worn by or overlapping with `person` instances, leading to severe foreground-on-foreground occlusion and low confidence scores below the `conf=0.25` threshold.
- **`book` (12.1% AP):** Frequently arranged in dense, repetitive clusters (e.g., bookshelves or stacks) where individual spine boundaries blur together at 640 × 640 input resolution.

In contrast, large, isolated classes with distinct silhouettes—such as **`bear` (71.8%)**, **`cat` (70.2%)**, and **`airplane` (69.3%)**—anchor the upper bound of the model's AP<sub>large</sub> performance.

---

## 3. Qualitative Case Study (`figure5_failed.png`)

Visual inspection of the Ground Truth (GT) vs. Prediction counts in `figure5_failed.png` illustrates how these quantitative drops manifest in practice:

- **Complete Recall Collapse in Cluttered Scenes (Image `000000000632`):**
  - **Ground Truth:** 13 `book` instances (alongside bedroom furniture).
  - **YOLOv10n Prediction:** **0 `book` instances detected** (0% recall).
  - **Root Cause:** In image `632`, the books are tightly packed along shelving units, occupying small bounding boxes with minimal inter-object contrast. Because YOLOv10n prioritizes throughput via a compact channel width and NMS-free one-to-one head assignment, dense grids of small, low-contrast items fail to cross the confidence threshold, directly driving the **`book` class down to 12.1% AP**.
- **Accessory Suppression & Occlusion:**
  - Across crowded street and indoor scenes in `figure5_failed.png`, dominant `person` bounding boxes are reliably localized, while attached `backpack` (10.7% AP) and `handbag` (10.7% AP) instances are systematically missed when partially occluded by arms or torsos.

---

## 4. Actionable Recommendations & Repository Next Steps

1. **Commit Missing Artifacts:**
   Ensure `results/per_class_ap.csv` (generated during the validation loop) and `figure5_failed.png` are tracked and committed alongside this report:
   ```bash
   git add results/per_class_ap.csv results/failure_analysis.md figure5_failed.png
   git commit -m "Add YOLOv10n failure analysis report and per-class AP metrics"
