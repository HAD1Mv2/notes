- [Data Collator](#data-collator)
  - [How to use DataCollator](#how-to-use-datacollator)
    - [Never use with padding](#never-use-with-padding)
      - [1. Why the Error Happens](#1-why-the-error-happens)
      - [2. How to Fix It](#2-how-to-fix-it)
      - [3. Alternative: Manually Pop the Data](#3-alternative-manually-pop-the-data)
  - [All kinds of data collator class](#all-kinds-of-data-collator-class)
    - [Core \& Generic Data Collators](#core--generic-data-collators)
    - [Text \& Language Modeling Data Collators](#text--language-modeling-data-collators)
    - [Task-Specific NLP Data Collators](#task-specific-nlp-data-collators)
    - [Quick Code Example: How to Use One](#quick-code-example-how-to-use-one)


# Data Collator

## How to use DataCollator

### Never use with padding

When using `DataCollatorWithPadding` with a Hugging Face `Trainer`, you must drop `offset_mapping` from your encoded data first. The collator collates and pads raw data types into tensors, which causes an error when encountering lists of tuples. 

#### 1. Why the Error Happens

Your tokenizer generates `offset_mapping` (a tuple of start/end character indices for each token) if you set `return_offsets_mapping=True` (common for NER or Question Answering tasks). However, `DataCollatorWithPadding` does not know how to handle these tuples during batching. It expects standard tensors like `input_ids` and `attention_mask`.

#### 2. How to Fix It
You can remove `offset_mapping` from the dataset before feeding it to the collator. 
Here is how you drop `offset_mapping` from your Hugging Face dataset:

```python
# 1. Map your tokenizer with return_offsets_mappingdef tokenize_and_map(examples):
    return tokenizer(
        examples["text"], 
        return_offsets_mapping=True, 
        truncation=True
    )
tokenized_dataset = dataset.map(tokenize_and_map, batched=True)
# 2. Remove 'offset_mapping' from the dataset to avoid data collator errors
tokenized_dataset = tokenized_dataset.remove_columns("offset_mapping")
```

#### 3. Alternative: Manually Pop the Data
If you need `offset_mapping` later in your pipeline (e.g., to map model predictions back to the original text), pop it out before yielding the batch: 

```python
def collate_fn(batch):
    # Pop 'offset_mapping' out of the features so it's not passed to the collator
    offsets = [example.pop("offset_mapping") for example in batch]
    
    # Let DataCollatorWithPadding handle the rest
    collated_batch = data_collator(batch)
    
    # Store offsets alongside your batched data so you can access them later
    collated_batch["offset_mapping"] = offsets
    
    return collated_batch
```

## All kinds of data collator class

The Hugging Face transformers library provides a wide range of specialized DataCollator classes designed to format, pad, and prepare dataset samples into batches for specific machine learning tasks.
Here is the complete categorized list of available data collators in the library.

------------------------------
### Core & Generic Data Collators
These are foundational collators used for basic batch formatting and structural manipulation.

* **`default_data_collator`**: A basic function (not a class) that packs dictionary-like objects into tensors without adding padding.
* **`DefaultDataCollator`**: The official class equivalent of the default function.
* **`DataCollatorMixin`**: The base mixin class used to build custom data collators with framework routing (PyTorch/NumPy/TensorFlow).
* **`DataCollatorWithPadding`**: Dynamically pads text inputs to the longest sequence in the batch using a specified tokenizer.
* **`DataCollatorWithFlattening`**: Materializes nested structures by flattening inputs before performing batch collation.

### Text & Language Modeling Data Collators
These handle the masking, permuting, and shifting required to train various types of NLP language models. 

* **`DataCollatorForLanguageModeling`**: Handles Masked Language Modeling (MLM like BERT) with random token masking, or Causal Language Modeling (CLM like GPT).
* **`DataCollatorForWholeWordMask`**: Special MLM collator that masks entire words instead of subtoken pieces (improves BERT-style training).
* **`DataCollatorForPermutationLanguageModeling`**: Specifically built for Permutation Language Modeling used to train XLNet models.
* **`DataCollatorForSeq2Seq`**: Dynamically pads both the inputs and the target labels specifically for sequence-to-sequence tasks (e.g., translation, summarization). 

### Task-Specific NLP Data Collators
These are optimized for standard supervised downstream tasks. [1] 

* **`DataCollatorForTokenClassification`**: Dynamically pads inputs and corresponding token-level labels (e.g., NER, POS tagging), masking pad label IDs with -100.
* **`DataCollatorForMultipleChoice`**: Formats inputs for multiple-choice models where each sample contains several text choice variations. 

------------------------------
### Quick Code Example: How to Use One
Data collators are most frequently passed directly into the Hugging Face [Trainer](https://huggingface.co/docs/transformers/main_classes/trainer): 

```python
from transformers import AutoTokenizer, DataCollatorWithPadding, Trainer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
# Sequences will dynamically pad to the longest sequence per batch
data_collator = DataCollatorWithPadding(tokenizer=tokenizer, return_tensors="pt")
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset,
    data_collator=data_collator,  # Passed here
)
```