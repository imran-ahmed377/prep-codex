# LLM Fine-Tuning Guide (Step-by-Step)

## Table of Contents

- [What is LLM Fine-Tuning?](#what-is-llm-fine-tuning)
- [Complete Fine-Tuning Pipeline](#complete-fine-tuning-pipeline)
- [Step 1: Load Dataset](#step-1-load-dataset)
- [Step 2: Install Required Libraries](#step-2-install-required-libraries)
- [Step 3: Check GPU](#step-3-check-gpu)
- [Step 4: Load the Base Model](#step-4-load-the-base-model)
- [Step 5: Maximum Sequence Length](#step-5-maximum-sequence-length)
- [Step 6: Load Model and Tokenizer](#step-6-load-model-and-tokenizer)
- [Step 7: Why Use 4-Bit?](#step-7-why-use-4-bit)
- [Step 8: Format Training Prompts](#step-8-format-training-prompts)
- [Why Add `<|endoftext|>`?](#why-add-endoftext)
- [Step 9: Create Hugging Face Dataset](#step-9-create-hugging-face-dataset)
- [Step 10: Add LoRA Adapters](#step-10-add-lora-adapters)
- [LoRA Rank](#lora-rank)
- [Target Modules](#target-modules)
- [LoRA Alpha](#lora-alpha)
- [Gradient Checkpointing](#gradient-checkpointing)
- [Step 11: Create Trainer](#step-11-create-trainer)
- [Step 12: Batch Size](#step-12-batch-size)
- [Gradient Accumulation](#gradient-accumulation)
- [Epochs](#epochs)
- [Learning Rate](#learning-rate)
- [Optimizer](#optimizer)
- [Step 13: Train the Model](#step-13-train-the-model)
- [Step 14: Switch to Inference Mode](#step-14-switch-to-inference-mode)
- [Step 15: Chat Template](#step-15-chat-template)
- [Step 16: Generate Response](#step-16-generate-response)
- [Step 17: Decode Tokens](#step-17-decode-tokens)
- [Step 18: Save as GGUF](#step-18-save-as-gguf)
- [Step 19: Download Model](#step-19-download-model)
- [Important Concepts](#important-concepts)
- [Complete Fine-Tuning Workflow](#complete-fine-tuning-workflow)
- [Summary](#summary)

This guide explains the complete LLM fine-tuning process using **Unsloth**, **LoRA**, and **Supervised Fine-Tuning (SFT)** in a simple, beginner-friendly way.

---

# What is LLM Fine-Tuning?

Large Language Models (LLMs) like **Phi-3**, **Llama**, **Mistral**, and **Gemma** are already trained on massive amounts of text from books, websites, articles, and code.

These models already know:

- English language
- Programming
- Mathematics
- Reasoning
- General knowledge

However, they don't know your specific task.

Fine-tuning teaches the model a **new specialized skill** using your own dataset.

Example:

**Before Fine-Tuning**

Input

```html
<div>
   <h2>iPhone</h2>
   <span>$999</span>
</div>
```

Output

```
The HTML contains a product.
```

After Fine-Tuning

Input

```html
<div>
   <h2>iPhone</h2>
   <span>$999</span>
</div>
```

Output

```json
{
  "product": "iPhone",
  "price": "$999"
}
```

The model becomes specialized in extracting product information.

---

# Complete Fine-Tuning Pipeline

```
Dataset
    │
    ▼
Load Dataset
    │
    ▼
Format Prompts
    │
    ▼
Load Base Model
    │
    ▼
Load Tokenizer
    │
    ▼
Tokenization
    │
    ▼
Add LoRA Adapters
    │
    ▼
Configure Trainer
    │
    ▼
Train Model
    │
    ▼
Test Model
    │
    ▼
Save Model
```

---

# Step 1: Load Dataset

```python
import json

file = json.load(open("json_extraction_dataset_500.json", "r"))
print(file[1])
```

## What does this do?

This loads your dataset into Python.

Example dataset:

```json
[
  {
    "input": "<div><h2>iPhone</h2></div>",
    "output": {
      "product": "iPhone"
    }
  }
]
```

Now

```python
file
```

contains a list of training examples.

---

## Why is this needed?

The model learns by looking at examples.

Think of a teacher.

Teacher:

> Here is HTML.

Student:

> Here is the correct JSON.

After hundreds of examples, the student learns the pattern.

---

# Step 2: Install Required Libraries

```python
!pip install unsloth trl peft accelerate bitsandbytes
```

These libraries make fine-tuning easier.

### Unsloth

- Faster training
- Lower memory usage
- Optimized for consumer GPUs

---

### PEFT

Parameter Efficient Fine-Tuning.

Instead of training billions of parameters, train only a tiny subset.

---

### TRL

Provides the `SFTTrainer` used for supervised fine-tuning.

---

### Accelerate

Helps efficiently use GPUs.

---

### BitsAndBytes

Allows loading models in 4-bit or 8-bit precision to reduce memory usage.

---

# Step 3: Check GPU

```python
import torch

print(torch.cuda.is_available())
```

If it prints

```
True
```

then training will use the GPU.

Otherwise it uses the CPU, which is much slower.

---

# Step 4: Load the Base Model

```python
model_name = "unsloth/Phi-3-mini-4k-instruct-bnb-4bit"
```

This is the pretrained model.

It already knows:

- English
- Coding
- Reasoning
- Mathematics

You are **not creating a new model**.

You are improving an existing one.

```
Base Model
      +
Your Dataset
      =
Specialized Model
```

---

# Step 5: Maximum Sequence Length

```python
max_seq_length = 2048
```

A language model reads **tokens**, not characters.

Example:

```
I love AI
```

might become

```
[23, 98, 405]
```

Each number is a token.

2048 means the model can process up to 2048 tokens at once.

---

# Step 6: Load Model and Tokenizer

```python
model, tokenizer = FastLanguageModel.from_pretrained(...)
```

This loads two things.

## Model

- The neural network.

- Contains billions of learned weights.

---

## Tokenizer

Converts text into token IDs.

Example

```
Apple
```

becomes

```
[5321]
```

The model only understands numbers.

---

# Step 7: Why Use 4-Bit?

```python
load_in_4bit=True
```

Normally, every weight uses 16 bits.

Example:

```
7 Billion Weights
```

This requires a lot of GPU memory.

Using 4-bit quantization compresses the weights.

Benefits:

- Smaller memory usage
- Faster loading
- Can run on consumer GPUs

---

# Step 8: Format Training Prompts

```python
def format_prompt(example):
```

Transforms the dataset into this format:

```
### Input:
<html>

### Output:
{
   ...
}
```

This teaches the model:

> "When I see HTML, I should produce JSON."

---

# Why Add `<|endoftext|>`?

```
<|endoftext|>
```

This token tells the model:

"The answer ends here."

Without it, the model may continue generating unnecessary text.

---

# Step 9: Create Hugging Face Dataset

```python
Dataset.from_dict(...)
```

The trainer expects a Hugging Face Dataset object.

Instead of

```
Python List
```

convert it into

```
Hugging Face Dataset
```

---

# Step 10: Add LoRA Adapters

```python
FastLanguageModel.get_peft_model(...)
```

This is one of the most important steps.

Without LoRA:

Train

```
3.8 Billion Parameters
```

Requires expensive GPUs.

With LoRA:

Freeze almost all weights.

Only train tiny adapter matrices.

```
Original Model
      │
      ├── Frozen
      │
      ▼
LoRA Adapters
      │
      ▼
Learn New Task
```

Benefits:

- Much lower memory usage
- Faster training
- Keeps original model knowledge

---

# LoRA Rank

```python
r = 64
```

Rank controls the adapter size.

Small Rank

```
r = 8
```

- Lower memory
- Lower learning capacity

Large Rank

```
r = 64
```

- Better learning
- More memory usage

---

# Target Modules

```python
q_proj
k_proj
v_proj
o_proj
```

These are Transformer attention layers.

Instead of modifying every layer,

LoRA only modifies important ones.

---

# LoRA Alpha

```python
lora_alpha = 128
```

Controls how strongly the LoRA adapters affect the model.

Common rule:

```
Alpha ≈ 2 × Rank
```

Since

```
Rank = 64
```

Alpha becomes

```
128
```

---

# Gradient Checkpointing

```python
use_gradient_checkpointing
```

Normally training stores many intermediate values.

This uses lots of memory.

Gradient checkpointing:

- Saves less
- Recomputes later
- Uses much less GPU memory

---

# Step 11: Create Trainer

```python
trainer = SFTTrainer(...)
```

Instead of writing the entire training loop manually,

the trainer automates it.

Without Trainer:

```
Forward Pass

↓

Compute Loss

↓

Backward Pass

↓

Optimizer Step

↓

Repeat
```

Trainer handles everything.

---

# Step 12: Batch Size

```python
per_device_train_batch_size = 2
```

Only 2 training examples are processed at once.

---

# Gradient Accumulation

```python
gradient_accumulation_steps = 4
```

Effective batch size becomes

```
2 × 4 = 8
```

Instead of updating after every 2 samples,

the model updates after 8 samples.

This allows larger effective batch sizes without increasing GPU memory.

---

# Epochs

```python
num_train_epochs = 3
```

One epoch means one complete pass through the dataset.

Example:

Dataset

```
500 Examples
```

Training:

```
Epoch 1

↓

Epoch 2

↓

Epoch 3
```

---

# Learning Rate

```python
learning_rate = 2e-4
```

Controls how much the weights change.

Too High

- Training becomes unstable.

Too Low

- Learning becomes very slow.

---

# Optimizer

```python
adamw_8bit
```

The optimizer decides how the trainable weights should change.

Using 8-bit optimization reduces memory usage.

---

# Step 13: Train the Model

```python
trainer.train()
```

Internally, each iteration looks like this:

```
Input

↓

Tokenizer

↓

Token IDs

↓

Model Prediction

↓

Compare with Correct Output

↓

Calculate Loss

↓

Backpropagation

↓

Update LoRA Weights

↓

Repeat
```

Only the LoRA adapters are updated.

The original model remains frozen.

---

# Step 14: Switch to Inference Mode

```python
FastLanguageModel.for_inference(model)
```

Training is finished.

The model is now optimized for generating responses quickly.

---

# Step 15: Chat Template

```python
tokenizer.apply_chat_template(...)
```

Converts your message into the exact format expected by Phi-3.

Example:

```
User:
Extract Product Information
```

becomes the proper internal prompt format.

---

# Step 16: Generate Response

```python
model.generate(...)
```

The model predicts tokens one by one.

```
Token

↓

Next Token

↓

Next Token

↓

...

↓

Complete JSON
```

---

# Step 17: Decode Tokens

```python
tokenizer.batch_decode(outputs)
```

The model generates numbers.

Example:

```
[12, 84, 152, 921]
```

The tokenizer converts them back into readable text.

---

# Step 18: Save as GGUF

```python
model.save_pretrained_gguf(...)
```

GGUF is a format used for efficient local inference.

It is compatible with tools such as:

- Ollama
- llama.cpp

---

# Step 19: Download Model

```python
files.download(...)
```

Downloads the generated `.gguf` file from Google Colab to your computer.

---

# Important Concepts

| Concept | Explanation |
|----------|-------------|
| LLM | A large neural network trained to predict the next token in text. |
| Token | A small piece of text represented by an integer ID. |
| Tokenizer | Converts text into tokens and tokens back into text. |
| Dataset | Collection of input-output examples used for training. |
| Prompt | Input given to the model. |
| Response | Desired output the model should generate. |
| Fine-Tuning | Continuing training on a task-specific dataset. |
| LoRA | Trains only small adapter layers instead of the full model. |
| QLoRA | Combines 4-bit quantization with LoRA. The base model is loaded in 4-bit precision and frozen, while only the small LoRA adapters are trained. This greatly reduces GPU memory usage while maintaining good performance. |
| PEFT | Techniques that make fine-tuning more memory efficient. |
| Quantization | Compresses model weights (e.g., 4-bit or 8-bit) to reduce memory usage. |
| Epoch | One complete pass through the dataset. |
| Batch Size | Number of samples processed together before gradient accumulation. |
| Gradient Accumulation | Combines gradients from multiple smaller batches before updating weights. |
| Learning Rate | Controls how much trainable weights change after each update. |
| Warmup | A training technique where the learning rate starts small and gradually increases during the first few training steps. This helps stabilize training and prevents large, unstable weight updates at the beginning. |
| Loss | Measures how far the prediction is from the correct answer. |
| Backpropagation | Computes gradients used to update trainable weights. |
| Checkpoint | Saved state of the model during training. |
| Inference | Using the trained model to make predictions on new inputs. |
| GGUF | Efficient model format for local inference with tools like Ollama and llama.cpp. |
---

# Complete Fine-Tuning Workflow

```
Prepare Dataset
        │
        ▼
Load JSON Dataset
        │
        ▼
Format Prompts
        │
        ▼
Load Base LLM
        │
        ▼
Load Tokenizer
        │
        ▼
Convert Text → Tokens
        │
        ▼
Attach LoRA Adapters
        │
        ▼
Configure Training Arguments
        │
        ▼
Train Model
        │
        ├── Forward Pass
        ├── Compute Loss
        ├── Backpropagation
        └── Update LoRA Weights
        │
        ▼
Switch to Inference Mode
        │
        ▼
Test on New Input
        │
        ▼
Generate JSON Output
        │
        ▼
Save Fine-Tuned Model (GGUF)
        │
        ▼
Deploy or Run Locally
```

---

# Summary

This workflow demonstrates a modern, memory-efficient approach to fine-tuning an LLM:

1. Prepare a dataset of input-output examples.
2. Load a pretrained language model and tokenizer.
3. Format the dataset into prompts.
4. Attach LoRA adapters instead of training all model parameters.
5. Configure training using `SFTTrainer`.
6. Fine-tune the model on your dataset.
7. Test the fine-tuned model on new inputs.
8. Export the model in GGUF format for efficient local inference.