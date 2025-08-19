---
title: "ConvNext based HSI Oil Spill Detection"
layout: "single"
excerpt: "Finetuning ConvNext model for classifying 3 types of oil spills"
image: "path/to/image.jpg"
header:
    overlay_image: "/assets/images/hsi_oilspill.png"
    overlay_filter: 0.5
---
# Overview
This project tackles oil spill detection using hyperspectral satellite imagery. Instead of full images, the approach extracts **256×256 patches** from **1024×1024 hyperspectral scenes (20 spectral bands)**.  
The task is a **4-class classification problem**:

- **Class 0:** Clean water  
- **Class 1:** Gasoline spill  
- **Class 2:** Motor oil spill  
- **Class 3:** Thinner spill  

The study uses **ConvNeXt** (a modern CNN architecture) adapted for hyperspectral inputs to distinguish between these oil spill types.

---

# Dataset
- **Source:** D. Rivas-Lalaleo and C. Hernandez, “Hydrocarbon spill hyperspectral dataset (hshd),” 2024.
- **Data format:** Originally .hdr files, but they were converted to `.npz` files containing hyperspectral images (shape: `1024 × 1024 × 20`)  
- Sample image of water type: <img src="/assets/images/hsi_oill/water_fcc.png" alt="Water Type Sample" width="200"  />

### Preprocessing
- Images are split into **256×256 patches**  to increase the dataset size and introduce spatial context.
- Patches are **normalized band-wise**  
- Labels are assigned based on the source file (`clean_arrays.npz`, `gasoline_arrays.npz`, etc.)  
- Each 1024×1024 image yields multiple patches → dataset expanded significantly  

### Splitting
- **70% training**  
- **30% testing**  

---

# Methodology

## Model
- **Backbone:** ConvNeXt-Small (`torchvision.models`)  
- **Modifications for hyperspectral data:**  
  - First convolution layer changed from **3 channels (RGB)** → **20 channels**  
  - Classifier head modified for **4 output classes**  
- Model wrapped with **DataParallel** for GPU usage  

## Training Setup
- **Loss:** CrossEntropyLoss  
- **Optimizer:** AdamW (learning rate = `1e-4`, weight decay = `1e-2`)  
- **Scheduler:** ReduceLROnPlateau (reduces LR when validation loss plateaus)  
- **Epochs:** 20 (with early stopping, patience = 5)  
- **Batch size:** 16  

### Evaluation Metrics
- Accuracy  
- Weighted F1-score  
- Precision, Recall, AUROC  
- Confusion Matrix  

## Training Procedure
Each epoch consists of:
1. Forward + backpropagation with training patches  
2. Metrics logged with **TorchMetrics (GPU accelerated)**  
3. Validation run to compute loss, accuracy, F1, and generate confusion matrix  

### Visualization
- Training/testing loss curves  
- Confusion matrix heatmaps  
- Classification report (per-class precision/recall/F1)  

---
# Final Results

## Training vs Test Accuracy
- **Training accuracy** steadily increased to **~99.9% by epoch 20**  
- **Test accuracy** peaked at **~96.9% in the final epoch**  
- Indicates **good generalization** without severe overfitting  
## Training vs Test Loss
- **Loss decreased consistently** during training  
- **Test loss closely tracked training loss** → suggests **stability** and no major variance  
## Evaluation Metrics
- **Test Accuracy:** ~96.9%  
- **Weighted Test F1 Score:** ~0.96  
### Confusion Matrix
- Normalized heatmap shows **strong per-class classification**  
- Misclassifications are minimal  
- Most errors are **between similar oil spill types** (e.g., motor oil vs gasoline)  

## Classification Report (sklearn)
- **Precision, Recall, and F1-scores:** ~0.95–0.97 across all 4 classes  
- No single class is disproportionately misclassified  

---
# Conclusion
- The **ConvNeXt model**, once adapted for **20-band hyperspectral data**, can effectively classify oil spill types.  
- Evaluation includes both **aggregate metrics** (accuracy, weighted F1) and **per-class performance** via confusion matrices.  
- Results demonstrate that **hyperspectral data contains rich spectral information** that deep CNNs like ConvNeXt can exploit for fine-grained classification of oil spill types.  
- The use of **patch-based learning** improves robustness and increases dataset size, but also introduces challenges of **spatial context loss**.  
- Future work could explore **multi-scale patch extraction** or **attention mechanisms** to better capture spatial relationships in hyperspectral data.