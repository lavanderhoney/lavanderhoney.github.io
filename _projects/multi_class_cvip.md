---
title: "CVIP Multi-class Classification"
layout: single
excerpt: "An ensemble-based pipeline combining CNNs, Random Forests, and XGBoost for ASL gesture recognition."
image: /assets/images/cvip_thumbnail.png
header:
  overlay_image: /assets/images/ensemble.png
  overlay_filter: 0.5
---

## Overview
An ensemble-based approach combining multiple ML/DL models to enhance ASL gesture recognition accuracy.

### Problem Statement
This project tackles the "Multi-Class Abnormality Classification for Video Capsule Endoscopy" challenge from Capsule Vision 2024. Video Capsule Endoscopy (VCE) is a procedure used to inspect the gastrointestinal (GI) tract for diseases by analyzing a video to find abnormalities. This analysis is a tedious process, involving the review of over a million frames from a 6-8 hour procedure. By leveraging deep learning, this project aims to automate the detection of abnormalities, thereby saving gastroenterologists valuable time.

### Dataset
The dataset for this challenge is composed of images from the SEE-AI project, KID, Kvasir-Capsule, and a private AIIMS VCE dataset. It is highly imbalanced, with a significant portion of the images belonging to the 'Normal' class. 

The dataset is divided into ten classes: 
Angioectasia, Bleeding, Erosion, Erythema, Foreign Body, Lymphangiectasia, Normal, Polyp, Ulcer, Worms

Dataset Distribution:
- Training set: 37,607 images 
- Validation set: 16,132 images 
<img src="/assets/images/cvip/distribution.png" alt="dataset distribution" width="400px"/>

### Methodology
#### Fine tuning
- Fine-tuned a vision transformer (ViT) base model and a ConvNeXt small model.
- Adam and AdamW with a weight decay of 0.01 were used.
- A Cosine Annealing scheduler was employed to adjust the learning rate during training. 
- TrivialAugmentWide was applied for data augmentation.
- To counter the class imbalance, I used the Weighted Cross-Entropy loss function, and relied on metrics like F1-score, and not just accuracy.

#### Ensembling
Ensembling is a tehcnique in ML which involves combining the predictions of multiple modles to make one strong predictor.

I tried ensembles of the fine-tuned ViT and ConvNext models. It makes sense to combine these two, as they've different ways of "looking" at an image.
- A ViT's primary strength is its ability to capture global context from the very beginning.It's self-attention mechanism allows every image patch to directly compare and relate itself to every other patch, no matter how far apart they are. This makes it exceptionally good at understanding long-range dependencies and the overall composition of a scene right away.
- A ConvNeXt, like all CNNs, operates on the principle of locality. Its convolutional kernels are small filters that slide over the image, identifying local features like edges, corners, and textures in the initial layers. It builds an understanding of the larger picture hierarchically; as data passes through deeper layers, the network combines these simple local patterns into more complex and larger structures. It only gains a "global" view after many layers of processing.

Now, they were combined using two techniques: 
- **Soft Voting**: The predictions of ViT and ConvNext (probabilities of 10 classes) are combined using a weighted sum, and the both models are then trained again. I started with a fine-tuned baseline of both
    - Results: 
    <img src="/assets/images/cvip/soft_voting_metrics.png" alt="metrics" width="400px"/>
    <img src="/assets/images/cvip/soft_voting_cf.png" alt="confusion matrix" width="400px"/>

- **2 Level Stacking**: The outputs of the ViT and ConvNext models were used as features for a deep neural network to make the final predictions.This idea was inspired from a paper on similar medical image classification
    - Results:
    <img src="/assets/images/cvip/2stage_cnf.png" alt="2 Level Stacking" width="400px"/>
    <img src="/assets/images/cvip/2stage_metrics.png" alt="2 Level Stacking" width="400px"/>

**Tech Stack:** Python, PyTorch

🔗 [View Fine-tuning Notebook](https://www.kaggle.com/code/milapp180/cvip-multi-class-classification)

🔗 [View Ensemble Notebook](https://www.kaggle.com/code/milapp180/ensemble-and-asl-cvip/edit/run/207067289)
