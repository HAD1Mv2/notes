- [How to use DataCollator](#how-to-use-datacollator)
  - [Never use with padding](#never-use-with-padding)
    - [1. Why the Error Happens](#1-why-the-error-happens)
    - [2. How to Fix It](#2-how-to-fix-it)
    - [3. Alternative: Manually Pop the Data](#3-alternative-manually-pop-the-data)


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
