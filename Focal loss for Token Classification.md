- [Focal Loss for Token Classification Problem](#focal-loss-for-token-classification-problem)
  - [How to use it in a Hugging Face Trainer](#how-to-use-it-in-a-hugging-face-trainer)
  - [Key Settings to Tweak](#key-settings-to-tweak)
  - [Understanting The Calculation](#understanting-the-calculation)
    - [The Input Example](#the-input-example)
      - [1. Simulated Raw Model Outputs (logits)](#1-simulated-raw-model-outputs-logits)
      - [2. Ground Truth Labels (targets)](#2-ground-truth-labels-targets)
    - [Step-by-Step Code Execution](#step-by-step-code-execution)
      - [Step 1: Flattening the Tensors](#step-1-flattening-the-tensors)
      - [Step 2: Masking Ignored Tokens](#step-2-masking-ignored-tokens)
      - [Step 3: Calculating Cross-Entropy Probabilities ($p\_t$)](#step-3-calculating-cross-entropy-probabilities-p_t)
      - [Step 4: Applying the Focal Loss Modulation](#step-4-applying-the-focal-loss-modulation)
      - [Step 5: Class Weighting (α)](#step-5-class-weighting-α)
      - [Step 6: Reduction](#step-6-reduction)


# Focal Loss for Token Classification Problem

Here is a complete, production-ready implementation of Multi-Class Focal Loss for token classification using **PyTorch**.
This implementation includes ignore_index masking (crucial for ignoring padding tokens and `[CLS]/[SEP]` tokens, which Hugging Face marks as `-100`).

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class TokenFocalLoss(nn.Module):
    def __init__(self, alpha=None, gamma=2.0, ignore_index=-100, reduction='mean'):
        """
        Multi-class Focal Loss for Token Classification.
        
        Args:
            alpha (Tensor, optional): A manual rescaling weight given to each class.
                                      Shape should be (num_classes,).
            gamma (float): Focusing parameter. Higher values down-weight easy tokens more.
            ignore_index (int): Specifies a target value that is ignored (e.g., -100 for pad tokens).
            reduction (str): 'mean', 'sum', or 'none'.
        """
        super(TokenFocalLoss, self).__init__()
        self.alpha = alpha
        self.gamma = gamma
        self.ignore_index = ignore_index
        self.reduction = reduction

    def forward(self, logits, targets):
        # logits shape: (batch_size, sequence_length, num_classes)
        # targets shape: (batch_size, sequence_length)
        
        # 1. Flatten tensors for element-wise calculation
        num_classes = logits.size(-1)
        logits = logits.view(-1, num_classes)
        targets = targets.view(-1)

        # 2. Create mask to filter out ignored tokens (like padding or special tokens)
        valid_mask = (targets != self.ignore_index)
        
        # If there are no valid tokens in the batch, return zero loss
        if not valid_mask.any():
            return torch.tensor(0.0, device=logits.device, requires_grad=True)

        # Filter active logits and targets
        active_logits = logits[valid_mask]
        active_targets = targets[valid_mask]

        # 3. Calculate Cross Entropy base probabilities (pt)
        # log_softmax is more numerically stable than softmax
        log_p = F.log_softmax(active_logits, dim=-1)
        
        # Gather the log probabilities of the true target classes
        log_pt = log_p.gather(dim=-1, index=active_targets.unsqueeze(1)).squeeze(1)
        p_t = torch.exp(log_pt)

        # 4. Calculate the Focal Loss modulation factor
        focal_weight = (1 - p_t) ** self.gamma
        loss = -focal_weight * log_pt

        # 5. Apply Alpha class weights if provided
        if self.alpha is not None:
            # Ensure alpha is on the correct device
            self.alpha = self.alpha.to(logits.device)
            # Gather alpha weight for each target token
            alpha_t = self.alpha.gather(dim=0, index=active_targets)
            loss = alpha_t * loss

        # 6. Apply reduction
        if self.reduction == 'mean':
            return loss.mean()
        elif self.reduction == 'sum':
            return loss.sum()
        else:
            return loss
```

## How to use it in a Hugging Face Trainer
To inject this into a standard Hugging Face workflow, override the compute_loss method of the Trainer:

```python
from transformers import Trainer
class FocalLossTrainer(Trainer):
    def compute_loss(self, model, inputs, return_outputs=False, num_items_in_batch=None):
        # Forward pass
        outputs = model(**inputs)
        logits = outputs.get("logits")
        labels = inputs.get("labels")
        
        # Initialize your Focal Loss
        # Example: 5 classes, giving more weight to rare entities
        alpha_weights = torch.tensor([0.1, 1.0, 1.0, 1.5, 1.5]) 
        loss_fct = TokenFocalLoss(alpha=alpha_weights, gamma=2.0, ignore_index=-100)
        
        # Calculate custom loss
        loss = loss_fct(logits, labels)
        
        return (loss, outputs) if return_outputs else loss
```

## Key Settings to Tweak

* **`gamma=2.0`**: This is the standard default. If your background tokens are extremely easy to predict and drowning out your metrics, try increasing this to `3.0` or `5.0`.
* **alpha Tensor**: If class `0` is the background tag (`O`) and classes `1-4` are actual entities, set class `0` to a low alpha value (e.g., `0.1`) and entities to higher values (e.g., `1.0` to `2.0`).

## Understanting The Calculation

To understand how this loss function works, let us trace a concrete example through the code step by step.
### The Input Example

Imagine we are doing Named Entity Recognition (NER) on a batch of sentences.

* **Batch Size**: 1 sentence
* **Sequence Length**: 3 tokens
* **Total Classes**: 3 classes (e.g., `0: Outside`, `1: Person`, `2: Location`)

#### 1. Simulated Raw Model Outputs (logits)
The model outputs raw, unnormalized scores for each of the 3 classes for every token.

```python
logits = torch.tensor([[[ 2.0, -1.0,  0.5],   # Token 1 (Likely Class 0)
                        [-0.5,  3.0,  0.0],   # Token 2 (Likely Class 1)
                        [ 1.0,  1.0,  1.0]]]) # Token 3 (Padding token/Special token)# Shape: (1, 3, 3) -> (batch_size, sequence_length, num_classes)
```

#### 2. Ground Truth Labels (targets)

```python
targets = torch.tensor([[0, 1, -100]]) # Shape: (1, 3) -> (batch_size, sequence_length)# Note: -100 is the ignore_index (padding)
```

------------------------------
### Step-by-Step Code Execution

#### Step 1: Flattening the Tensors
To perform calculations easily, the code flattens the batch and sequence dimensions together.

```python
logits = logits.view(-1, 3)
targets = targets.view(-1)
```

* **New logits shape**: `(3, 3)`
* **New targets shape**: `(3,)` -> `tensor([0, 1, -100])`

#### Step 2: Masking Ignored Tokens
We need to ignore the padding token (`-100`) so it does not affect our gradients.

```python
valid_mask = (targets != -100) # Returns: tensor([True, True, False])
active_logits = logits[valid_mask]
active_targets = targets[valid_mask]
```

* **active_logits**: Drops the 3rd row → `tensor([[ 2.0, -1.0, 0.5], [-0.5, 3.0, 0.0]])`
* **active_targets**: Drops `-100` → `tensor([0, 1])`

#### Step 3: Calculating Cross-Entropy Probabilities ($p_t$)
First, we turn the raw logits into log-probabilities using Softmax.

```python
log_p = F.log_softmax(active_logits, dim=-1)
```

For Token 1, applying Softmax to `[2.0, -1.0, 0.5]` turns into probabilities roughly equal to `[0.80, 0.04, 0.16]`.
Taking the log gives:

* **log_p**: `tensor([[-0.22, -3.22, -1.72], [-3.51, -0.01, -3.01]])`

Next, we extract only the log-probability of the correct target class for each token using gather.

* Token 1 target is 0 → extracts `-0.22`
* Token 2 target is 1 → extracts `-0.01`

```python
log_pt = log_p.gather(dim=-1, index=active_targets.unsqueeze(1)).squeeze(1)# log_pt = tensor([-0.22, -0.01])
p_t = torch.exp(log_pt)# p_t = tensor([0.80, 0.99])  <-- These are the actual model confidences for correct classes
```

#### Step 4: Applying the Focal Loss Modulation
This is where Focal Loss diverges from standard Cross-Entropy. It calculates a modulating weight $(1 - p_t)^\gamma$ to down-weight easy examples. Let's use **gamma = 2.0**.

```python
focal_weight = (1 - p_t) ** 2.0
```

* Token 1: $(1 - 0.80)^2 = (0.20)^2 = \mathbf{0.04}$ (Slightly down-weighted because it's easy)
* Token 2: $(1 - 0.99)^2 = (0.01)^2 = \mathbf{0.0001}$ (Heavily suppressed because the model is already 99% confident)

Now, multiply this modulating weight by the original negative log-likelihood loss:

```python
loss = -focal_weight * log_pt
# Token 1 loss: -0.04 * -0.22 = 0.0088
# Token 2 loss: -0.0001 * -0.01 = 0.000001
```

#### Step 5: Class Weighting (α)
If you pass an `alpha` tensor (e.g., `tensor([0.1, 1.0, 1.0]`) to give less weight to the frequent class `0`), the code gathers the specific alpha value for each token's true class and scales the loss:

```python
alpha_t = tensor([0.1, 1.0]) # Class 0 gets 0.1 weight, Class 1 gets 1.0 weight
loss = alpha_t * loss
# Token 1 loss: 0.0088 * 0.1 = 0.00088
```
#### Step 6: Reduction
Finally, the remaining token losses are averaged together to output a single scalar value that PyTorch can optimize.

```python
return loss.mean() # Final scalar Loss = (0.00088 + 0.000001) / 2 = 0.00044
```
------------------------------