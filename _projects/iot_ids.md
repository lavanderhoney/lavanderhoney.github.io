---
title: "IoT Intrusion Detection System"
layout: single
excerpt: "A binary classification system for detecting intrusions in IoT networks using time-series data."
image: /assets/images/iot_ids_thumbnail.png
header:
  overlay_image: /assets/images/iot_ids.png
  overlay_filter: 0.5
---

### Overview
---
This project implements and evaluates deep learning models for intrusion detection in Industrial Internet of Things (IIoT) networks using the X-IIoTID dataset. Three different architectures—a 1D Convolutional Neural Network (CNN), a Long Short-Term Memory (LSTM) network, and a Gated Recurrent Unit (GRU) network—are trained to classify network traffic as either 'Normal' or 'Attack'. The models are built using PyTorch, and the project demonstrates a complete workflow from data preprocessing and sequence creation to model training and evaluation, achieving high accuracy in identifying malicious activities.

### Problem Statement
---
The increasing integration of IoT devices in industrial settings exposes critical systems to significant cybersecurity threats. Traditional security measures are often insufficient for the unique characteristics of IIoT traffic. This project aims to develop an effective intrusion detection system (IDS) by leveraging deep learning techniques to analyze sequential network traffic data. The goal is to build a robust classifier that can accurately distinguish between benign and malicious network packets in real-time, thereby enhancing the security and integrity of IIoT environments.

#### Sequence framing
---
An important aspect of this problem is to correctly frame the problem as a time-series, and accordingly create the sequences for the models.

The problem of intrusion detection can be modelled as a real-time state detection or a **sequence classification** task, where the input is a sequence of network traffic records (from the captured packets) and the output is the corresponding class label (Normal or Attack), for this particular sequence of packets.
```
Input: [x_t-n, ..., x_t-1, x_t]
Target (Label): y_t
```

This is a valid approach because the label of the last packet often serves as a proxy for the state of the entire sequence up to that point. In network security, an attack is rarely a single, isolated event. It is typically a sequence of packets that builds up to a malicious action.  Therefore:

- The model learns precursors: The model is implicitly trained to recognize the patterns in x[0] and x[1] that lead to an attack being identified at time x[2]. If a port scan happens at x[0] and x[1], leading to a connection flood at x[2], the label y[2] (Attack) correctly flags the culmination of that sequence.

- Context is key: The features of x[2] alone might not be enough to classify it as an attack. It's the context provided by x[0] and x[1] that allows the model to make a correct prediction for y[2]. The model learns "when I see this pattern of packets, the last one is an attack."

Alternate approaches to sequence framing in this case include:

- Any-Attack-in-Sequence (OR Logic): This approach frames the problem such that if any packet in the sequence is labeled as an attack, the entire sequence is considered an attack. This can help in scenarios where attacks are distributed across multiple packets.
- All-Attacks-in-Sequence (AND Logic): This approach requires that all packets in the sequence be labeled as attacks for the entire sequence to be considered an attack. This can be useful in scenarios where a specific pattern of attacks must be present throughout the sequence.
### Dataset
---
The project utilizes the **X-IIoTID dataset** from Kaggle, a comprehensive collection of network traffic data specifically designed for evaluating IIoT security solutions. The dataset contains a total of **820,834 records** before cleaning, each representing a network flow with **68 distinct features**. These features capture a wide range of information, including:

-   **Flow Identifiers**: Source/Destination IP addresses and ports.
-   **Flow Statistics**: Duration, packet/byte counts, packet rates, and byte rates.
-   **Protocol Information**: Protocol type (TCP, UDP, ICMP) and service (HTTP, DNS, etc.).
-   **System Metrics**: CPU performance metrics like average user time, system time, and idle time.
-   **Security Alerts**: OSSEC alert levels and login attempt flags.

The dataset is labeled with a multi-class attack classification (`class1` and `class2`) and a binary classification (`class3`), which is used as the target for this project ('Normal' vs. 'Attack').

### Methodology

#### Data Preprocessing
---
1.  **Cleaning**: The dataset was first cleaned by replacing placeholder values (`-`, `?`) with NaN and removing duplicate and null entries. This resulted in a refined dataset of **591,003 records**.
2.  **Feature Engineering**:
    * The target variable `target` was created by mapping the `class3` column to binary values: `0` for 'Normal' and `1` for 'Attack'.
    * Categorical features `Protocol` and `Service` were one-hot encoded to be used in the model.
    * Boolean and string boolean values (`True`/`False`) were converted to integer representations (`1`/`0`).
3.  **Data Transformation**:
    * All numerical feature columns were converted to a float data type to ensure consistency.
    * The dataset was sorted by `Timestamp` to maintain the temporal order of network events, which is crucial for training sequence-based models.
4.  **Sequence Creation**: To prepare the data for the RNN and 1D CNN models, the time-series data was transformed into sequences. A sequence length of **10** was chosen, meaning each input sample for the model consists of 10 consecutive network traffic records.

#### Model Training
---
Three deep learning models were implemented in PyTorch and trained for 30 epochs to classify the network traffic sequences.

1.  **1D-CNN Model**:
    * **Architecture**: The CNN model uses two 1D convolutional layers (`Conv1d`) with 32 and 64 output channels, respectively, followed by `ReLU` activations and two fully-connected linear layers for downsampling. A `Dropout` layer (p=0.5) is used for regularization before the final linear layers.
    * **Input Shape**: The model takes input of shape `(batch_size, num_features, seq_length)`.

2.  **LSTM Model**:
    * **Architecture**: This model consists of a two-layer LSTM with a hidden size of 64. The output from the last time step is passed through a `LayerNorm` layer, a `ReLU` activated linear layer, a `Dropout` layer (p=0.5), and a final output neuron.
    * **Input Shape**: The LSTM takes input of shape `(batch_size, seq_length, num_features)`.

3.  **GRU Model**:
    * **Architecture**: Similar to the LSTM, this model uses a two-layer GRU with a hidden size of 64. The output from the final time step is processed through a fully connected network with `ReLU` activation and `Dropout`.
    * **Input Shape**: The GRU also takes input of shape `(batch_size, seq_length, num_features)`.

**Training Process**
---
- All models were trained using the `BCEWithLogitsLoss` function with `pos_weight` to handle class imbalance.
- I experimented with optimizers such as `Adam`,`Nadam`, `RMSProp` and `SGD` for the training, each with a learning rate of 1e-4.
    - The 1D-CNN model trained well with `SGD`, `RMSProp`, and `Nadam`, giving smooth training and test curves, while with `Adam` it showed more fluctuations, and didn't converge at all.
    - The LSTM model didn't converge with `NAdam`, `RMSProp`, `SGD`, and trained smoothly with `Adam`.
    - The GRU model performed well on all the optimizers, and converged relatively smoothly.
- The data was split into training (80%) and validation (20%) sets, and performance was tracked using Accuracy and F1-Score.

**Conclusion**
---
All three models, with the best optimizer, performed well, with almost 98% accuracy and similar F1-score on the validation set.

I learnt that, when handling a complex dataset, it's important to experiment with different optimizers, as they can behave differently with each model.

---
**Tech Stack:** Python, PyTorch, Scikit-learn, Pandas, NumPy

🔗 [View Notebook](https://www.kaggle.com/code/milapp180/onion-routing)
