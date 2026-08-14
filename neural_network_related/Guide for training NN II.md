- [Warm Up Training](#warm-up-training)
    - [Why is warm-up useful?](#why-is-warm-up-useful)
    - [Typical learning-rate schedule](#typical-learning-rate-schedule)
    - [Is warm-up always necessary?](#is-warm-up-always-necessary)

# Warm Up Training

Warm-up in neural network training means **starting with a small learning rate and gradually increasing it** over the first few training steps/epochs until reaching the target learning rate.

### Why is warm-up useful?

1. **Prevents unstable updates at the beginning**

   At initialization, model weights are essentially random. Large learning-rate updates can cause the model to make very large parameter changes before it has learned anything useful.

   Warm-up gives the optimizer time to make smaller, safer adjustments first.

2. **Helps with large learning rates**

   For large models, especially Transformers, you may want a relatively large learning rate for efficient training. Jumping immediately to that rate can cause:

   * exploding gradients
   * loss spikes
   * unstable training
   * divergence

   Warm-up reduces this initial shock.

3. **Improves optimization stability**

   Suppose your target learning rate is:

   `LR = 0.001`

   Instead of immediately using 0.001:

   ```text
   Step 1       0.00001
   Step 100     0.00010
   Step 500     0.00050
   Step 1000    0.00100
   ```

   The model gradually transitions into the main training regime.

4. **Particularly useful with Adam/AdamW**

   Optimizers such as Adam and AdamW maintain estimates of gradient statistics. At the very beginning, these estimates are still being established. Warm-up can make the interaction between these estimates and the learning rate more stable.

5. **Useful for large-batch training**

   When using very large batch sizes, people often increase the learning rate. This can make the beginning of training more sensitive, so warm-up is commonly used to compensate.

### Typical learning-rate schedule

A common setup is:

```text
Learning rate
    ^
    |             ┌──────────────
    |            /
    |           /
    |          /
    |         /
    |________/__________________> training steps
             ↑
          warm-up
```

After warm-up, you might then use a **cosine decay**, linear decay, or another learning-rate schedule.

### Is warm-up always necessary?

**No.** For small/simple models or carefully tuned learning rates, training may work perfectly well without it.

It's especially common when training:

* Transformers / LLMs
* large neural networks
* models with Adam/AdamW
* large-batch training
* models where training is unstable at the beginning

**In short:** warm-up prevents the model from taking overly aggressive optimization steps before the optimizer and model have settled into a useful training regime.
