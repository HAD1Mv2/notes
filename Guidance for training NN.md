- [Data Imbalance](#data-imbalance)
  - [1. Data-Level Solutions (Resampling)](#1-data-level-solutions-resampling)
    - [WeightedRandomSampler](#weightedrandomsampler)
  - [2. Loss-Level Solutions (Reweighting)](#2-loss-level-solutions-reweighting)
    - [For Binary Classification (BCEWithLogitsLoss)](#for-binary-classification-bcewithlogitsloss)
    - [For Multi-Class Classification (CrossEntropyLoss)](#for-multi-class-classification-crossentropyloss)
  - [3. Evaluation Metrics Warning](#3-evaluation-metrics-warning)
  - [4. Advanced Techniques](#4-advanced-techniques)
    - [1. Focal Loss (Loss-Level Solution)](#1-focal-loss-loss-level-solution)
    - [2. SMOTE (Data-Level Solution)](#2-smote-data-level-solution)
- [Tensor indexing](#tensor-indexing)
- [Loading Custom Loss Function to Device](#loading-custom-loss-function-to-device)
  - [Method 1: Load weigths directly (`.to(device`))](#method-1-load-weigths-directly-todevice)
  - [Method 2: If Writing a Custom Loss Class](#method-2-if-writing-a-custom-loss-class)
  - [Method 3: The Quick Fix (Match Device On-the-Fly)](#method-3-the-quick-fix-match-device-on-the-fly)
  - [Weights or the loss function (`.to(device)`)](#weights-or-the-loss-function-todevice)
    - [Why moving the custom loss function is better](#why-moving-the-custom-loss-function-is-better)
    - [The Cleanest Design Pattern](#the-cleanest-design-pattern)
    - [What happens in your training loop?](#what-happens-in-your-training-loop)
- [The Role of Class Imbalance Weights in Training Calculation](#the-role-of-class-imbalance-weights-in-training-calculation)


# Data Imbalance

To handle imbalanced data when training a neural network in PyTorch, the most effective approach is to adjust loss function weights using WeightedRandomSampler or loss weights like pos_weight, which forces the model to pay more attention to underrepresented classes.
Here is a comprehensive guide to the best techniques for managing imbalanced datasets in PyTorch.

------------------------------
## 1. Data-Level Solutions (Resampling)

Instead of modifying the loss function, you can alter the batch generation process so the neural network sees an equal distribution of classes during training.

### WeightedRandomSampler

[PyTorch](https://pytorch.org/) provides a native sampler that ensures every batch contains a balanced representation of classes by sampling minority classes more frequently.

```python
import torch
from torch.utils.data import DataLoader, WeightedRandomSampler

# Example: 100 samples (90 of Class 0, 10 of Class 1)
target = torch.cat([torch.zeros(90), torch.ones(10)]).long()

# 1. Compute class counts and weights

class_counts = torch.bincount(target)  # tensor([90, 10])
class_weights = 1.0 / class_counts.float()  # tensor([1/90, 1/10])

# 2. Assign a weight to each individual sample
sample_weights = class_weights[target]

# 3. Create the sampler
sampler = WeightedRandomSampler(
    weights=sample_weights, 
    num_samples=len(sample_weights), 
    replacement=True
)

# 4. Pass to DataLoader (set shuffle=False when using a sampler)
train_loader = DataLoader(dataset, batch_size=32, sampler=sampler)
```

------------------------------
## 2. Loss-Level Solutions (Reweighting)
If you prefer not to oversample or undersample data, you can penalize the model more heavily when it misclassifies minority class samples.

### For Binary Classification (BCEWithLogitsLoss)
Use the `pos_weight` parameter to scale the loss of the positive class.

```python
import torch.nn as nn
# Formula: num_negative_samples / num_positive_samples# Example: 90 negative, 10 positive -> pos_weight = 9.0
pos_weight = torch.tensor([9.0])
criterion = nn.BCEWithLogitsLoss(pos_weight=pos_weight)
```

### For Multi-Class Classification (CrossEntropyLoss)
Use the weight parameter to pass a 1D tensor assigning a weight to each class.

```python
import torchimport torch.nn as nn
# Example: Class 0 (90 samples), Class 1 (10 samples), Class 2 (5 samples)
class_counts = torch.tensor([90.0, 10.0, 5.0])
# Inverse frequency weighting
weights = 1.0 / class_counts
weights = weights / weights.sum()  # Normalize weights
criterion = nn.CrossEntropyLoss(weight=weights)
```

------------------------------
## 3. Evaluation Metrics Warning
When dealing with imbalanced datasets, never use Accuracy as your primary metric. A dummy model predicting the majority class will still yield high accuracy. Instead, track these metrics using the torchmetrics library:

* **Precision & Recall**: To monitor false positives and false negatives.
* **F1-Score / Macro F1**: The harmonic mean of precision and recall.
* **Precision-Recall AUC (PR-AUC)**: Better than ROC-AUC for severe class imbalances. 

------------------------------
## 4. Advanced Techniques
If standard resampling or reweighting does not work, consider these advanced pipelines:

* **Focal Loss**: Dynamically scales the loss based on prediction confidence, forcing the model to focus on "hard" minority examples rather than "easy" majority ones. 
  
* **Pre-computed Augmentation**: Use external libraries like imbalanced-learn to apply **SMOTE (Synthetic Minority Over-sampling Technique)** on your feature vectors before passing them into PyTorch. 

------------------------------


The two most powerful advanced techniques for handling severe class imbalance in PyTorch are Focal Loss and SMOTE (Synthetic Minority Over-sampling Technique).

------------------------------
### 1. Focal Loss (Loss-Level Solution)
Standard Cross-Entropy Loss treats all misclassifications equally. In highly imbalanced data, the massive volume of "easy-to-classify" majority samples dominates the gradient, drowning out the signal from the rare minority samples.
Focal Loss solves this by adding a modulating factor $(1 - p_t)^\gamma$ to the loss. This **down-weights easy examples** and forces the model to focus heavily on hard, misclassified examples (which are typically your minority class).

 **<u>Mathematical Intuition</u>**

$$\text{Focal Loss} = -\alpha_t (1 - p_t)^\gamma \log(p_t)$$ 

* $p_t$: The model's estimated probability for the correct class.
* γ (Gamma): The focusing parameter. When γ = 0, this is standard Cross-Entropy. As γ increases (typically set to `2.0`), the loss for well-classified examples drops to near zero.
* α (Alpha): A balancing factor for class prevalence (similar to standard class weights).

**<u>PyTorch Implementation (Binary Class)</u>**

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class BinaryFocalLoss(nn.Module):
    def __init__(self, alpha=0.25, gamma=2.0, reduction='mean'):
        super(BinaryFocalLoss, self).__init__()
        self.alpha = alpha
        self.gamma = gamma
        self.reduction = reduction

    def forward(self, inputs, targets):
        # inputs: raw logits from the model, targets: binary labels (0 or 1)
        bce_loss = F.binary_cross_entropy_with_logits(inputs, targets, reduction='none')
        pt = torch.exp(-bce_loss)  # Prevents numerical instability
        
        # Calculate focal component
        focal_loss = self.alpha * (1 - pt) ** self.gamma * bce_loss
        
        if self.reduction == 'mean':
            return focal_loss.mean()
        elif self.reduction == 'sum':
            return focal_loss.sum()
        return focal_loss
```

------------------------------
### 2. SMOTE (Data-Level Solution)
Simple over-sampling duplicates minority samples, which often leads to severe overfitting. SMOTE prevents this by **synthesizing entirely new, artificial minority examples**.
It works by selecting a minority class sample, finding its k-nearest neighbors in the feature space, and drawing a line between them. It then places a new synthetic data point randomly along that line.

**<u>PyTorch Integration Pipeline</u>**

Because SMOTE is feature-based, it cannot easily generate raw data like images or complex text strings. It is best applied to **tabular data**, or **embeddings** extracted from a pre-trained encoder model.

You should use the external `imbalanced-learn` library to apply SMOTE *before* wrapping your data in a PyTorch `DataLoader`.

```bash
pip install imbalanced-learn
```

```python
import torchfrom torch.utils.data 
import TensorDataset, DataLoader
from imblearn.over_sampling import SMOTEimport numpy as np

# 1. Your original imbalanced PyTorch Tensors (Tabular or Embeddings)# X_raw: (10000, 64) features, y_raw: (10000,) labels
X_np = X_raw.numpy()
y_np = y_raw.numpy()

# 2. Apply SMOTE in CPU/Numpy space# This synthesizes new data points until classes are perfectly 50/50
smote = SMOTE(sampling_strategy='auto', random_state=42)
X_resampled, y_resampled = smote.fit_resample(X_np, y_np)

# 3. Convert back to PyTorch Tensors
X_resampled_tensor = torch.tensor(X_resampled, dtype=torch.float32)
y_resampled_tensor = torch.tensor(y_resampled, dtype=torch.float32).unsqueeze(1)

# 4. Create standard DataLoader (No special sampler needed anymore!)
balanced_dataset = TensorDataset(X_resampled_tensor, y_resampled_tensor)
train_loader = DataLoader(balanced_dataset, batch_size=64, shuffle=True)
```

------------------------------
**When to choose which?**

* **Choose Focal Loss** if you are training computer vision (images) or NLP (text tokens) models where creating synthetic raw data is impossible or alters the semantic meaning negatively.
* **Choose SMOTE** if you are working with tabular clinical data, financial fraud vectors, or fixed-length embeddings, and your minority class is so tiny that the model lacks geometric diversity to learn boundaries.

------------------------------

# Tensor indexing

Question: `class_weights[target]`
could you explain what this syntax mean ?

In PyTorch (and NumPy), the syntax `class_weights[target]` utilizes a technique called tensor indexing (or advanced indexing). It maps a small array of class-level weights directly to a larger array of individual samples based on their class labels.

Here is a step-by-step breakdown of exactly how it works.

------------------------------

**<u>The Goal</u>**

Imagine you have a batch of 5 samples belonging to two classes (0 and 1).

* Class 0 is common, so it gets a low weight: 0.1
* Class 1 is rare, so it gets a high weight: 0.9 [2] 

You need to assign the correct weight to each of your 5 individual samples based on their label.

------------------------------
**<u>Step-by-Step Execution</u>** 

1. Define the Weights Array (`class_weights`)

    This tensor stores unique weights indexed by the class ID. Index 0 holds the weight for Class 0, and index 1 holds the weight for Class 1. 
```python
# Index:            0     1
class_weights = torch.tensor([0.1,  0.9])
```

2. Define the Labels Array (`target`)
   
This tensor holds the actual class labels for your specific data samples.

```python
# Sample:          1st  2nd  3rd  4th  5th
target = torch.tensor([0,   0,   1,   0,   1])
```

3. Apply the Tensor Indexing (`class_weights[target]`)

When you pass the integer tensor target inside the brackets of `class_weights`, PyTorch replaces every label in target with the corresponding value located at that index inside `class_weights`.

* 1st sample (0) → looks up class_weights[0] → 0.1
* 2nd sample (0) → looks up class_weights[0] → 0.1
* 3rd sample (1) → looks up class_weights[1] → 0.9
* 4th sample (0) → looks up class_weights[0] → 0.1
* 5th sample (1) → looks up class_weights[1] → 0.9

**<u>The Output</u>**

```python
sample_weights = class_weights[target]
print(sample_weights)# Result: tensor([0.1, 0.1, 0.9, 0.1, 0.9])
```

------------------------------
**<u>Why is this used?</u>**

Instead of writing a slow `for` loop to look up the weight for every single item in your dataset, this syntax triggers highly optimized C++ vectorization under the hood. It instantly broadcasts the weights across millions of samples in milliseconds.

This resulting `sample_weights` tensor is exactly what `WeightedRandomSampler` requires to determine the selection probability of each individual data point.

------------------------------

# Loading Custom Loss Function to Device

Question: when using custom loss function with weights, how to put this in GPU ?, since I training the model using GPU and I encountered error saying that the weights is not on device

To fix this error, you must explicitly move the weight tensor to the same GPU device where your model features and targets are located. 

PyTorch requires all tensors involved in a mathematical operation (like calculating loss) to reside on the exact same hardware device (CPU or specific GPU).

Here is how to properly move your weights to the GPU using three different design patterns.

------------------------------
## Method 1: Load weigths directly (`.to(device`))

Define a generic device variable at the top of your script. Pass that device to your loss function weights during initialization.

```python
import torchimport torch.nn as nn

# 1. Define the device dynamically
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 2. Move your weights to that device during creation
class_counts = torch.tensor([90.0, 10.0])
weights = (1.0 / class_counts).to(device) # <--- Move to GPU here

# 3. Pass the GPU-bound weights to the loss function
criterion = nn.CrossEntropyLoss(weight=weights)

# 4. Move your model to the same device

model = MyNeuralNet().to(device)
# --- Inside your training loop ---

for inputs, targets in train_loader:
    # 5. Move batch data to GPU
    inputs, targets = inputs.to(device), targets.to(device)
    
    outputs = model(inputs)
    loss = criterion(outputs, targets) # No more device mismatch errors!
```

------------------------------

## Method 2: If Writing a Custom Loss Class

If you built a custom nn.Module class for your loss function (like the Focal Loss example), you should register the weights as a **buffer**.

Registering a tensor as a buffer ensures that whenever you call `criterion.to(device)` or `model.to(device)`, the weights will automatically move to the GPU alongside everything else without being treated as trainable model parameters.

```python
import torch
import torch.nn as nn

class CustomWeightedLoss(nn.Module):
    def __init__(self, class_weights_tensor):
        super().__init__()
        # register_buffer automatically handles .to(device) transfers
        self.register_buffer('weights', class_weights_tensor)

    def forward(self, inputs, targets):
        # self.weights will automatically be on GPU if the module is on GPU
        raw_loss = ... 
        weighted_loss = raw_loss * self.weights[targets]
        return weighted_loss.mean()
# Usage:
raw_weights = torch.tensor([0.1, 0.9])
criterion = CustomWeightedLoss(raw_weights).to(device) # Moves weights automatically!
```

------------------------------
## Method 3: The Quick Fix (Match Device On-the-Fly)
If you don't want to manage a global `device` variable, you can extract the device location directly from your target labels inside the training loop and copy the weights there instantly using `.to(targets.device)`.

```python
# Create weights on CPU initially
weights = torch.tensor([0.1, 0.9]) 

for inputs, targets in train_loader:
    inputs, targets = inputs.cuda(), targets.cuda()
    
    # Send weights to whatever GPU 'targets' is currently sitting on
    gpu_weights = weights.to(targets.device)
    
    criterion = nn.CrossEntropyLoss(weight=gpu_weights)
    
    outputs = model(inputs)
    loss = criterion(outputs, targets)
```

------------------------------

## Weights or the loss function (`.to(device)`)

Question: so it is better to put weights only on the device or the custom loss function ?

Put the custom loss function object itself on the device.
When you call `.to(device)` on a custom loss function object, it automatically moves all of its internal tensors—including your class weights—to the GPU for you.

Here is exactly why this happens and how to structure your code to make it work seamlessly.

------------------------------
### Why moving the custom loss function is better

In PyTorch, custom loss functions inherit from `nn.Module`. This gives them access to the same tracking mechanisms as your neural network layers.

If you register your weights using `register_buffer()` inside the loss function, calling `.to(device)` on the loss object achieves two things at once:

   1. It moves the **entire container** (the loss object) to the GPU.
   2. It automatically finds and moves **all internal weight** tensors to the GPU.

This keeps your main training loop exceptionally clean. You only have to call `.to(device)` once when initializing your components.

------------------------------

### The Cleanest Design Pattern

This is the standard, production-ready way to handle weights inside a custom loss function.

```python
import torchimport torch.nn as nn
import torch.nn.functional as F

class WeightedBinaryFocalLoss(nn.Module):
    def __init__(self, class_weights_tensor, gamma=2.0):
        super().__init__()
        self.gamma = gamma
        
        # KEY STEP: Registering as a buffer binds the tensor to the module.
        # It ensures the tensor moves to the GPU whenever the module moves.
        self.register_buffer('weights', class_weights_tensor)

    def forward(self, inputs, targets):
        # self.weights is guaranteed to be on the same GPU as inputs/targets
        bce_loss = F.binary_cross_entropy_with_logits(inputs, targets, reduction='none')
        pt = torch.exp(-bce_loss)
        
        # Apply the weights securely on the GPU
        focal_loss = self.weights[targets.long()] * (1 - pt) ** self.gamma * bce_loss
        return focal_loss.mean()

# --- Initialization Setup ---
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 1. Create raw weights on CPU
raw_weights = torch.tensor([0.2, 0.8])

# 2. Instantiate and move the ENTIRE loss function to the device
criterion = WeightedBinaryFocalLoss(raw_weights).to(device)

# 3. Move your model to the device
model = MyNeuralNetwork().to(device)
```
------------------------------

### What happens in your training loop?

Because you moved the entire custom loss function during setup, your training loop remains simple and free of messy device management code:

```python
for inputs, targets in train_loader:
    # Move batch data to GPU
    inputs, targets = inputs.to(device), targets.to(device)
    
    # Forward pass
    outputs = model(inputs)
    
    # Just pass the data. The criterion already lives on the GPU!
    loss = criterion(outputs, targets) 
    
    # Backward pass
    loss.backward()
```

------------------------------

# The Role of Class Imbalance Weights in Training Calculation

Question: so, when we use the class weights in the calculation in the training loop ? is it when we calculate the loss ?

Yes, exactly. You apply the class weights in the training loop at the exact moment you calculate the loss.
The weights are completely ignored during the forward pass of your neural network. Their only job is to multiply and scale the raw error values generated by the model before you run the backward pass.

------------------------------

**<u>The Precise Workflow Sequence</u>**

Here is the exact step-by-step sequence of a training iteration to show you exactly where the weights do their work:

```python
for inputs, targets in train_loader:

    # 1. Move raw batch data to GPU
    inputs, targets = inputs.to(device), targets.to(device)
    
    # 2. Reset the gradients
    optimizer.zero_grad()
    
    # 3. Forward Pass: The model predicts without knowing about weights
    outputs = model(inputs)
    
    # 4. LOSS CALCULATION: The weights are applied here!
    # Under the hood, PyTorch looks at the 'targets', finds the corresponding 
    # class weight, and multiplies the error of that specific sample by that weight.
    loss = criterion(outputs, targets) 
    
    # 5. Backward Pass: Scaled errors produce larger gradients for minority classes
    loss.backward()
    
    # 6. Update Model Weights
    optimizer.step()
```

------------------------------

**<u>What Actually Happens Inside the Loss Calculation?</u>**

To visualize it, suppose your batch contains **an easy majority class sample** and **a rare minority class sample**.

   1. **The Model Predicts**: The model makes a wrong prediction for both samples.
   2. **Raw Error**: The loss function calculates a raw error of 1.0 for both mistakes.
   3. **Weight Application**:
   
      1. The majority sample weight is low (0.1). Its adjusted loss becomes: $1.0 \times 0.1 = \mathbf{0.1}$
      2. The minority sample weight is high (0.9). Its adjusted loss becomes: $1.0 \times 0.9 = \mathbf{0.9}$
   4. **The Result**: When `loss.backward()` runs, the gradients driven by the minority sample are **9 times stronger** than the majority sample. This forces the neural network to drastically alter its internal parameters to correct its mistake on the rare class.

------------------------------

