# Human_Detection_from_high_altitudes
Lightweight small-human detection system for high-altitude drone imagery using YOLO + SAHI. Optimized with pruning, quantization, and TFLite conversion for real-time edge deployment. Designed for efficient aerial surveillance and precise payload-target identification on Android devices.

## Mobile-Optimized Human Detection (SAHI + YOLO)
README Documentation (Deployment-Oriented Version)

This document explains:

✅ A mobile-optimized architecture

📊 Complete inference time increase analysis

⚙️ Memory and latency trade-offs

🚀 Recommended Android deployment pipeline

This is written specifically for high-altitude small human detection use cases.

### 1️⃣ Why Your Current Multi-Model + SAHI Is NOT Mobile-Friendly

Your current system:

YOLOv10n

YOLOv9t

YOLOv8n

SAHI slicing

IoU fusion

Computational Impact

Let:

Single YOLO inference = T

Number of models = 3

SAHI slices per image = S

Total inference ≈ 3 × S × T

If:

T = 25 ms (YOLOv10n GPU)

S = 6 slices (1920×1080 with overlap)

Then:

Total ≈ 3 × 6 × 25 ms
      ≈ 450 ms

👉 ~0.45 seconds per image

On CPU/mobile → much slower (1–2 seconds).

Memory usage:

YOLOv10n ≈ 3–4 MB

YOLOv9t ≈ 5–6 MB

YOLOv8n ≈ 3–4 MB

Total model memory ≈ 12–14 MB (before runtime buffers)

On Android with TFLite:

RAM usage increases further due to tensors & buffers.

### 2️⃣ Mobile-Optimized Approach (Recommended Architecture)

Instead of 3 models + fusion, use:

✅ Single NAS-Optimized Model + SAHI (Lightweight Mode)

Architecture:

Input Image
      ↓
Adaptive SAHI (small slice count)
      ↓
Single Optimized YOLO (Pruned + Quantized)
      ↓
Human Class Filtering
      ↓
Output
### 3️⃣ Optimization Strategy
Step 1 — Use Only One Model

Recommended base:

YOLOv10n (smallest, efficient backbone)

Remove:

Multi-model ensemble

Late fusion overhead

Step 2 — Structured Pruning

Remove low-importance channels.

Typical reduction:

20–40% parameters

10–25% latency reduction

Step 3 — INT8 Quantization

Convert to:

ONNX → TFLite

Full INT8 quantization

Benefits:

4× smaller weights

30–50% faster inference on mobile CPU

Step 4 — Adaptive SAHI

Instead of fixed slicing:

Use slicing only if image resolution > threshold.

Example logic:

If image_width > 1500:
    Apply SAHI
Else:
    Run direct inference

This reduces unnecessary slicing overhead.

### 4️⃣ Inference Time Comparison (Complete Breakdown)

Assume 1920×1080 drone frame.

Setup	Models	SAHI	Approx Time
Single YOLO	1	No	25–40 ms
Single YOLO + SAHI	1	Yes	120–180 ms
Multi-Model (3)	3	No	75–120 ms
Multi-Model + SAHI	3	Yes	400–600 ms
Optimized (Pruned + INT8 + Adaptive SAHI)	1	Conditional	60–120 ms
### 5️⃣ Complete Inference Time Increase Explanation

Let:

Base single model time = T

Number of models = M

Number of slices = S

Then:

Total Time ≈ M × S × T

For your original pipeline:

M = 3

S ≈ 6–8

T ≈ 30 ms

Total ≈ 540–720 ms

That is:

15–20× slower than a single direct YOLO inference.

### 6️⃣ Recommended Mobile Architecture (Final)
Lightweight Android-Ready Pipeline
Drone Frame
      ↓
Resolution Check
      ↓
Conditional SAHI
      ↓
Single Pruned YOLOv10n (INT8 TFLite)
      ↓
Human Filtering
      ↓
Bounding Boxes
### 7️⃣ Expected Mobile Performance

On mid-range Android device:

Stage	Time
Preprocessing	10 ms
TFLite INT8 Model	40–70 ms
SAHI (if enabled)	+40 ms
Total	80–120 ms

That enables:

✔ ~8–12 FPS
✔ Real-time capable
✔ Battery efficient

### 8️⃣ Why This Is Better Than Multi-Model
Feature	Multi-Model	Optimized Single
Recall	High	High (after tuning)
Latency	Very High	Moderate
Memory	High	Low
Edge Deployable	No	Yes
Battery Friendly	No	Yes
### 9️⃣ Research vs Deployment Mode
Research Mode

Use:

Multi-model + SAHI

Maximum recall

Benchmarking

Deployment Mode

Use:

NAS-optimized YOLO

Pruning

Quantization

Adaptive SAHI

### 🔟 Final Recommendation for Your Project

Since your goal involves:

Drone-based human detection

Android deployment

Edge efficiency

The correct production strategy is:

Distill multi-model knowledge into one optimized student model, then prune + quantize + apply adaptive SAHI.

This keeps:

✔ High small-human recall
✔ Low latency
✔ Efficient memory usage
✔ Practical mobile deployment

## 📌 Conclusion

Your original Multi-Model + SAHI system:

Maximizes detection robustness

Increases inference time by ~15–20×

Not mobile efficient

The optimized single-model pipeline:

Maintains strong accuracy

Reduces latency by ~70–85%

Suitable for real-time Android deployment
