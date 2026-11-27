# Failure Analysis: YOLOv10n on COCO 2017 Validation

## 1. Executive Summary
While **YOLOv10n** (`yolov10n.pt`) achieves a strong real-time latency of **2.9 ms/image** at `imgsz=640` and an overall validation score of **39.5% $\text{AP}_{50:95}$** (**55.0% $\text{AP}_{50}$**), performance varies sharply across object scales and semantic categories. 

A joint inspection of the per-class metrics (`results/per_class_ap.csv`) and qualitative failure cases (`figure5_failed.png`) reveals that the nano-scale architecture struggles substantially with **small, densely packed, and heavily occluded objects** ($\text{AP}_{\text{small}} = 20.2\%$) compared to large, distinct foreground subjects ($\text{AP}_{\text{large}} = 55.8\%$).

---

## 2. Quantitative Breakdown: Scale & Class Disparities

### Scale Sensitivity ($\text{AP}_{\text{small}}$ vs. $\text{AP}_{\text{large}}$)
| Metric / Subset | Score ($\text{AP}_{50:95}$) | Performance Gap | Primary Architectural Bottleneck |
| :--- | :---: | :---: | :--- |
| **$\text{AP}_{\text{large}}$** | **55.8%** | Baseline | High spatial resolution retained in deeper feature maps (`P5`). |
| **Overall $\text{AP}$** | **39.5%** | $-16.3\%$ | Balanced trade-off between lightweight backbone and detection head. |
| **$\text{AP}_{\text{small}}$** | **20.2%** | **$-35.6\%$** | Aggressive downsampling loses fine-grained edge/texture cues. |

### Worst-Performing Classes (`results/per_class_ap.csv`)
The lowest-scoring categories in `results/per_class_ap.csv` directly mirror the scale and occlusion bottlenecks seen in $\text{AP}_{\text{small}}$:
* **`hair drier` ($0.3\%\text{ AP}$):** Extreme class rarity in COCO combined with high intra-class shape variance and small pixel footprint.
* **`backpack` ($10.7\%\text{ AP}$) & `handbag` ($10.7\%\text{ AP}$):** Typically appear as small secondary accessories worn by or overlapping with `person` instances, leading to severe foreground-on-foreground occlusion and low confidence scores below the `conf=0.25` threshold.
* **`book` ($12.1\%\text{ AP}$):** Frequently arranged in dense, repetitive clusters (e.g., bookshelves or stacks) where individual spine boundaries blur together at $640 \times 640$ input resolution.

In contrast, large, isolated classes with distinct silhouettes—such as **`bear` ($71.8\%$)**, **`cat` ($70.2\%$)**, and **`airplane` ($69.3\%$)**—anchor the upper bound of the model's $\text{AP}_{\text{large}}$ performance.

---

## 3. Qualitative Case Study (`figure5_failed.png`)

Visual inspection of the Ground Truth (GT) vs. Prediction counts in `figure5_failed.png` illustrates how these quantitative drops manifest in practice:

* **Complete Recall Collapse in Cluttered Scenes (Image `000000000632`):**
  * **Ground Truth:** 13 `book` instances (alongside bedroom furniture).
  * **YOLOv10n Prediction:** **0 `book` instances detected** ($0\%$ recall).
  * **Root Cause:** In image `632`, the books are tightly packed along shelving units, occupying small bounding boxes with minimal inter-object contrast. Because YOLOv10n prioritizes throughput via a compact channel width and NMS-free one-to-one head assignment, dense grids of small, low-contrast items fail to cross the confidence threshold, directly driving the **`book` class down to $12.1\%\text{ AP}$**.
* **Accessory Suppression & Occlusion:**
  * Across crowded street and indoor scenes in `figure5_failed.png`, dominant `person` bounding boxes are reliably localized, while attached `backpack` ($10.7\%\text{ AP}$) and `handbag` ($10.7\%\text{ AP}$) instances are systematically missed when partially occluded by arms or torsos.

---

## 4. Actionable Recommendations & Repository Next Steps

1. **Commit Missing Artifacts:**
   * Ensure `results/per_class_ap.csv` (generated during the validation loop) and `figure5_failed.png` are tracked and committed alongside this report:
     ```bash
     git add results/per_class_ap.csv results/failure_analysis.md figure5_failed.png
     git commit -m "Add YOLOv10n failure analysis report and per-class AP metrics"
     ```
2. **Inference & Architecture Mitigations for Small/Dense Objects:**
   * **Increase Input Resolution:** Evaluate at `imgsz=1280` (or use SAHI / tiled inference) to preserve feature resolution for `book`, `backpack`, and `handbag` instances, directly targeting the $20.2\%\ \text{AP}_{\text{small}}$ bottleneck.
   * **Confidence Threshold Tuning:** Lower `conf` from `0.25` to `0.10–0.15` when recall on small accessories or cluttered shelves is prioritized over precision.
   * **Model Scaling:** Benchmark `yolov10s.pt` or `yolov10m.pt` if the latency budget permits $>3\text{ ms/image}$, as wider `P3` feature maps substantially improve small-object feature representation.