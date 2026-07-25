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
- [Hugginface Dataset](#hugginface-dataset)
  - [Notes about `IterableDataset`](#notes-about-iterabledataset)
    - [Common Reasons for Unexpected RAM Growth](#common-reasons-for-unexpected-ram-growth)
      - [1. Large Shuffle Buffers](#1-large-shuffle-buffers)
      - [2. Accumulating Python Statistics or Objects](#2-accumulating-python-statistics-or-objects)
      - [3. PyTorch DataLoader Workers (num\_workers \> 0) \[12\]](#3-pytorch-dataloader-workers-num_workers--0-12)
      - [4. Heavy Data Formats (Audio and Images)](#4-heavy-data-formats-audio-and-images)
      - [Alternative: Standard Dataset with Memory Mapping](#alternative-standard-dataset-with-memory-mapping)
  - [Notes about `Dataset`](#notes-about-dataset)
    - [How Memory Mapping Works](#how-memory-mapping-works)
    - [When Local Files Will Use RAM](#when-local-files-will-use-ram)
  - [Processing data on the fly with Dataset obj](#processing-data-on-the-fly-with-dataset-obj)
    - [Option 1: Use `dataset.with_transform()` (Recommended)](#option-1-use-datasetwith_transform-recommended)
      - [How to use it:](#how-to-use-it)
      - [Why it works well:](#why-it-works-well)
      - [The Catch:](#the-catch)
    - [Option 2: Preprocess inside the PyTorch `DataLoader` collate\_fn](#option-2-preprocess-inside-the-pytorch-dataloader-collate_fn)
      - [How to use it:](#how-to-use-it-1)
      - [Why it works well:](#why-it-works-well-1)
    - [Comparison Matrix](#comparison-matrix)
  - [Create custom\_collate\_fn that include DataCollater functionality.](#create-custom_collate_fn-that-include-datacollater-functionality)
    - [Method 1: Functional Wrapper (Best for standard PyTorch DataLoader)](#method-1-functional-wrapper-best-for-standard-pytorch-dataloader)
    - [Method 2: Custom Class Wrapper (Best for clean code \& HF Trainer)](#method-2-custom-class-wrapper-best-for-clean-code--hf-trainer)
  - [⚠️ Critical Warning for On-The-Fly Tokenization](#️-critical-warning-for-on-the-fly-tokenization)


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

*Important Note: datacollator will automatically convert the encodings into pytorch tensor, so when we built dataset, we don't necessarily need to convert the encoding into 'pt' first.*

# Hugginface Dataset

## Notes about `IterableDataset`

Question: Is huggingface iterable dataset not using full data loaded into ram ?

Yes, the Hugging Face IterableDataset does not load the full dataset into RAM. Instead, it lazily loads and streams data sequentially (on-the-fly) from local storage or remote repositories, processing only a small fraction of rows at any given time. [1, 2, 3] 
However, if you are noticing your RAM fills up anyway while using an IterableDataset, it is usually due to a few specific data-handling pitfalls. [4] 

------------------------------
### Common Reasons for Unexpected RAM Growth
If you notice RAM scaling up despite using streaming=True, look out for these factors: [2] 
#### 1. Large Shuffle Buffers

* An IterableDataset cannot truly randomize a dataset because it does not have random access to the data.
* When you call .shuffle(buffer_size=...), Hugging Face pre-loads exactly buffer_size items into RAM to shuffle them dynamically.
* The Fix: Keep your buffer_size reasonable (e.g., 1,000 to 10,000 rows) instead of setting it to the size of the whole dataset. [2, 5, 6, 7] 

#### 2. Accumulating Python Statistics or Objects

* If you iterate through your IterableDataset using a simple loop and accidentally append features, lists, or metrics to a standard Python list, Python keeps those references active. [1, 8, 9] 
* The Fix: Avoid storing full-resolution rows in tracking variables. Clear variables or explicitly invoke the garbage collector (import gc; gc.collect()) if doing heavy manipulation. [1, 8, 9, 10, 11] 

#### 3. PyTorch DataLoader Workers (num_workers > 0) [12] 

* When using a PyTorch DataLoader with multiple workers, each worker instantiates its own copy of the dataset iterator and data buffers.
* High worker counts multiply your baseline streaming memory consumption.
* The Fix: Reduce num_workers to a smaller value (like 2 or 4) if you observe RAM creeping higher during training loops. [13, 14, 15, 16] 

#### 4. Heavy Data Formats (Audio and Images)

* Features like uncompressed audio waves or high-resolution images take significant overhead to unpack in memory during a .map() transformation.
* The Fix: Apply .map() with a smaller batched=True configuration and set a low batch_size to force early memory recycling. [17, 18, 19, 20] 

------------------------------
#### Alternative: Standard Dataset with Memory Mapping
If your dataset is stored locally on disk, you often do not even need an IterableDataset. [21] 

* Hugging Face’s standard [Dataset object](https://huggingface.co/docs/datasets/about_mapstyle_vs_iterable) utilizes Apache Arrow memory-mapping (mmap) under the hood.
* It leaves the file on your disk and loads rows into RAM strictly on-demand when requested by a training batch, providing fast random access without consuming huge amounts of memory. [6, 22, 23] 

To isolate why your setup is taking up unexpected RAM, could you tell me:

* Are you shuffling the data, and if so, what is your buffer size?
* What kind of data are you working with (e.g., text, audio, images)?
* Are you using multiple workers in a PyTorch DataLoader?

I can give you the exact lines of code needed to fix the memory leakage.

[1] [https://huggingface.co](https://huggingface.co/docs/datasets/about_mapstyle_vs_iterable)
[2] [https://huggingface.co](https://huggingface.co/docs/datasets/access)
[3] [https://huggingface.co](https://huggingface.co/docs/datasets/package_reference/loading_methods)
[4] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/using-dataset-in-streaming-mode-causing-increasing-in-ram/172651)
[5] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/best-practices-for-a-large-dataset/137632)
[6] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/limitations-of-iterable-datasets/16794)
[7] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/what-s-the-definiation-of-lazy-loading-is-iterabledataset-also-faster-than-dataset-when-loading-locally/168304)
[8] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/expected-memory-usage-of-dataset/45648)
[9] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/expected-memory-usage-of-dataset/45648/2)
[10] [https://blog.heaphero.io](https://blog.heaphero.io/handling-large-datasets-without-running-out-of-memory/)
[11] [https://www.kaggle.com](https://www.kaggle.com/code/yuliagm/how-to-work-with-big-datasets-on-16g-ram-dask)
[12] [https://github.com](https://github.com/huggingface/datasets/issues/5506)
[13] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/prevent-iterable-dataset-from-consuming-all-the-rams/160276)
[14] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/a-streaming-datasets-memory-footprint-continually-grows/159404/3)
[15] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/how-to-handle-iterabledataset-with-huggingface-trainer-and-num-workers-in-ddp-setup/78459)
[16] [https://github.com](https://github.com/huggingface/datasets/issues/3423)
[17] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/running-out-of-ram-using-iterable-datasets/169790)
[18] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/running-out-of-ram-using-iterable-datasets/169790/2)
[19] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/running-out-of-memory-processing-dataset/34796)
[20] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/using-a-generator-for-the-map-function-in-an-iterable-dataset/67354)
[21] [https://huggingface.co](https://huggingface.co/docs/datasets/access)
[22] [https://discuss.huggingface.co](https://discuss.huggingface.co/t/big-text-dataset-loading-for-training/153576)
[23] [https://huggingface.co](https://huggingface.co/learn/llm-course/chapter5/4)

## Notes about `Dataset`

Question: Oh, so in case dataset object, if I use load data from local file, its not loaded data into RAM ?

Correct, the standard Dataset object does not load the entire dataset into RAM when you load it from local files (like CSV, JSON, Parquet, or Arrow). Instead, Hugging Face zero-loads the data into memory by using a technology called Apache Arrow memory-mapping (mmap). [1, 2] 
Here is exactly how it manages memory:
### How Memory Mapping Works

* Disk-Based Reading: The data stays on your hard drive.
* Virtual Address Mapping: Hugging Face maps the file directly to virtual memory addresses.
* Just-In-Time Loading: Rows are only copied into your actual RAM at the exact millisecond you access them (e.g., when a training batch is actively grabbed).
* Instant Eviction: As soon as the training step finishes with that batch, the RAM is freed up or recycled.

### When Local Files Will Use RAM
While the core dataset remains on disk, RAM will scale up slightly in these specific scenarios:

* In-Memory Modifications: If you use .map(), .filter(), or .shuffle() without specifying a cache_file_name, Hugging Face may process those updates in RAM. (Tip: Use keep_in_memory=False in your map functions to force it to write updates to a disk cache instead of RAM).
* Massive Index Arrays: Operations like .shuffle() create a list of randomized row indexes. While the rows stay on disk, the list of row integer positions resides in RAM (though this uses very little memory).
* Collate Functions: When your data loaders grab a batch and format it into PyTorch/TensorFlow tensors, that specific active batch resides fully in RAM/GPU memory.

If you want to optimize your local setup, let me know:

* What file format is your local dataset in (e.g., JSON, CSV, Arrow)?
* Are you running heavy preprocessing transformations using .map()?

I can show you how to configure your loading script to ensure your RAM usage stays near zero.

[1] [https://chayansraj.medium.com](https://chayansraj.medium.com/data-ingestion-with-copy-into-in-snowflake-local-file-aws-azure-gcp-6da65a2f058f)
[2] [https://slingdata.io](https://slingdata.io/articles/export-load-local-parquet-sqlite-sling/)

## Processing data on the fly with Dataset obj

Question: what if I want to keep using Dataset object but preprocessing data still on the fly ?

To process data **on the fly** while keeping a standard `Dataset` object, you have two primary options depending on your training framework. [1] 


------------------------------
### Option 1: Use `dataset.with_transform()` (Recommended)

The `.with_transform()` method replaces `.map()` for on-the-fly execution. It takes your preprocessing function and wraps the dataset, applying the transformation lazily only when rows are explicitly indexed (e.g., during batching in your data loader).

#### How to use it:
```python
# 1. Define your function to handle a batch dictionary
def preprocess_on_the_fly(batch):
    # This runs lazily during training, NOT upfront
    inputs = tokenizer(batch["text"], padding="max_length", truncation=True)
    return inputs

# 2. Apply the transform (this happens instantly with 0 RAM/disk overhead)
dataset.with_transform(preprocess_on_the_fly)
```
#### Why it works well:

* **Zero Disk Cache**: It does not write any new files to your hard drive.
* **Keeps Dataset Features**: It remains a standard Dataset object, meaning you can still use `.select()`, `.shuffle()`, and check its exact length `len(dataset)`.
* **Dynamic**: If you change your preprocessing logic or augmentations, it applies instantly without needing to clear old caches. [2] 

#### The Catch:

* Repeated Computation: It does not save the results. If you train for 5 epochs, the exact same preprocessing function will run 5 times on every single row.

------------------------------
### Option 2: Preprocess inside the PyTorch `DataLoader` collate_fn

Instead of modifying the dataset object at all, you can pass your raw text/images directly to the PyTorch `DataLoader` and handle the preprocessing inside a **custom collate function**.

#### How to use it:

``` python
from torch.utils.data import DataLoader
# 1. Define a collate function that processes a list of raw rowsdef custom_collate_fn(batch):
    texts = [item["text"] for item in batch]
    labels = [item["label"] for item in batch]
    
    # Preprocess/tokenize the batch on the fly
    tokenized = tokenizer(texts, padding=True, truncation=True, return_tensors="pt")
    tokenized["labels"] = torch.tensor(labels)
    
    return tokenized

# 2. Pass your raw, untouched Hugging Face dataset straight to the DataLoader
train_loader = DataLoader(dataset, batch_size=32, collate_fn=custom_collate_fn)
```

#### Why it works well:

* **Multiprocessing Power**: PyTorch standard `DataLoader` uses `num_workers` to parallelize this collate function on the fly across multiple CPU cores right before passing it to the GPU.
* **Standard practice**: This is the native, industry-standard PyTorch approach to data pipelines. [3] 

------------------------------
### Comparison Matrix

| Choice | Best Used For... | Keeps len(dataset)? | Multiprocessing |
|---|---|---|---|
| `with_transform() `| Quick experiments, Hugging Face Trainer integration. | Yes | Uses HF internal loading. |
| `collate_fn` | Native PyTorch loops, complex tensor manipulation. | Yes | Highly optimized via PyTorch num_workers. |

Are you using the Hugging Face **Trainer** or a custom **PyTorch training loop**? I can give you the exact code template tailored to your specific training script setup.

[1] [https://medium.com](https://medium.com/neural-nomad/an-end-to-end-machine-learning-project-guide-af339d364f7c)
[2] [https://www.reddit.com](https://www.reddit.com/r/deeplearning/comments/1lamwur/data_augmentation_is_not_necessarily_about/)
[3] [https://www.codecademy.com](https://www.codecademy.com/article/how-to-use-pytorch-dataloader-custom-datasets-transformations-and-efficient-techniques)

## Create custom_collate_fn that include DataCollater functionality.

Question: how to combine custom_collate_fn function with DataCollator object ?

You can combine a custom collate function with a Hugging Face `DataCollator` object by wrapping the data collator inside your own function.

A `DataCollator` is just a callable object that accepts a list of dictionary samples and returns a dictionary of tensors. By passing your raw data through your custom logic first and then feeding the result into the data collator, you achieve both custom on-the-fly preprocessing and optimal batch padding/tensor conversion. [1] 

Here is how to do it using two different methods depending on your workflow.

------------------------------
### Method 1: Functional Wrapper (Best for standard PyTorch DataLoader)

In this approach, your custom function extracts and tokenizes the raw text on the fly, maps the keys correctly, and then calls the `data_collator `manually to handle the padding and PyTorch tensor conversion.

```python
import torch
from torch.utils.data import DataLoader
from transformers import AutoTokenizer, DataCollatorWithPadding

#### 1. Initialize tokenizer and the standard HF data collator
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
base_collator = DataCollatorWithPadding(tokenizer=tokenizer)

#### 2. Create your combined collate function
def custom_collate_fn(batch):
    # Step A: Extract raw strings from the dataset batch
    texts = [item["text"] for item in batch]
    labels = [item["label"] for item in batch]
    
    # Step B: Tokenize on the fly (do NOT pad here, let the collator handle it dynamically)
    tokenized_features = tokenizer(texts, truncation=True, max_length=512)
    
    # Step C: Reconstruct a list of dicts that the DataCollator expects
    features_list = []
    for i in range(len(batch)):
        feature = {
            "input_ids": tokenized_features["input_ids"][i],
            "attention_mask": tokenized_features["attention_mask"][i],
            "label": labels[i]
        }
        features_list.append(feature)
        
    # Step D: Pass to HF DataCollator to pad dynamically and convert to torch.Tensors
    return base_collator(features_list)

# 3. Pass your combined function straight to your DataLoader
train_loader = DataLoader(dataset, batch_size=32, collate_fn=custom_collate_fn)
```

------------------------------
### Method 2: Custom Class Wrapper (Best for clean code & HF Trainer)

If you are using the Hugging Face `Trainer`, it expects a single object passed to `data_collator=`. Writing a custom class that inherits from your chosen data collator allows you to inject your preprocessing right before the padding step occurs.

```python
from transformers import DataCollatorWithPadding

class OnTheFlyProcessingCollator(DataCollatorWithPadding):
    def __call__(self, features):
        # 'features' is a list of raw rows directly from your untouched Dataset object
        
        # Step A: Extract and tokenize the raw text on the fly
        texts = [item["text"] for item in features]
        tokenized = self.tokenizer(texts, truncation=True, max_length=512)
        
        # Step B: Rebuild the features list with tokenized outputs and original labels
        processed_features = []
        for i in range(len(features)):
            processed_features.append({
                "input_ids": tokenized["input_ids"][i],
                "attention_mask": tokenized["attention_mask"][i],
                "labels": features[i]["label"] # DataCollator expects 'labels' (plural) for loss calculation
            })
            
        # Step C: Call the parent class (DataCollatorWithPadding) to execute dynamic padding
        return super().__call__(processed_features)

# How to initialize it
custom_collator = OnTheFlyProcessingCollator(tokenizer=tokenizer)

# Use it in standard PyTorch:
# train_loader = DataLoader(dataset, batch_size=32, collate_fn=custom_collator)

# Or pass it straight to the Hugging Face Trainer:
# trainer = Trainer(..., data_collator=custom_collator, ...)
```

------------------------------

## ⚠️ Critical Warning for On-The-Fly Tokenization

When tokenizing inside a collate function, never set `padding=True` inside the tokenizer call.

If you do, the tokenizer will pad each batch to your maximum possible length (e.g., 512 tokens), completely breaking the main benefit of the `DataCollator` (which is padding batches dynamically to the length of the longest item in that specific batch to save GPU memory). Always let `super().__call__()` or `base_collator()` handle the padding. [2] 

[1] [https://huggingface.co](https://huggingface.co/learn/llm-course/chapter3/2)
[2] [https://medium.com](https://medium.com/@sujathamudadla1213/what-is-datacollatorwithpadding-in-hugging-face-transformers-12c2b3b2f612)

