# FLAP 0.5x Llama3.1-70b

## Overview

This guide provides a walkthrough of applying **FLAP** (Fluctuation-based Adaptive Structured Pruning) to compress and accelerate the Llama3.1-70b model. FLAP allows for significant reduction in model size and computational requirements without sacrificing performance. Unlike traditional pruning techniques, FLAP requires no retraining and adapts the pruning ratio across different modules and layers, offering an efficient and effective approach for deploying large language models in resource-constrained environments.

## Table of Contents
- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Pruning](#running-the-pruning)
- [Performance Evaluation](#performance-evaluation)
- [Conclusion](#conclusion)

## Introduction

In this example, we'll be utilizing the FLAP technique to prune the Llama3.1-70b model, aiming to reduce its memory footprint and improve inference speed. FLAP's adaptive compression structure and no-training-required approach make it a versatile tool for adapting large models to various hardware configurations.

## Requirements

Before you begin, ensure that you have the following:
- A GPU-enabled environment with CUDA support.
- The Nyuntam repository cloned and set up as per the [Installation Guide](#installation).

## Installation

### Step 1: Clone the Nyuntam Repository

Clone the repository and navigate to the `nyuntam` directory:
```bash
git clone https://github.com/nyunAI/nyuntam.git
cd nyuntam
```

### Step 2: Set Up the Workspace

Create and activate an environment for the FLAP pruning example:
```bash
conda create -n flap_pruning python=3.10 # or use virtualenv if preferred
conda activate flap_pruning
```

Install the required dependencies:
```bash
pip install -r text_generation/pruning/flap/requirements.txt
```

## Configuration

Prepare the YAML configuration file specific to FLAP pruning. Use the following template as a starting point:

```yaml
# flap_pruning.yaml

# Model configuration
MODEL: "meta-llama/Meta-Llama-3.1-70B-Instruct"

# Data configuration
DATASET_NAME: "wikitext"
DATASET_SUBNAME: "wikitext-2-raw-v1"
TEXT_COLUMN: "text"                     
SPLIT: "train"

DATA_PATH:
FORMAT_STRING:

# Quantization configuration
llm:
  FlapPruner:
    dtype: "float16"
    metrics: "WIFV"
    nsamples: 1024
    pruning_ratio: 0.5
    remove_heads: -1
    seed: 0
    start_pruning_layer_idx: 22
    structure: "AL-AM"

    to_finetune: False

# Job configuration
CUDA_ID: "0,1,2,3"
ALGORITHM: "FlapPruner"
JOB_SERVICE: "Kompress"
USER_FOLDER: "user_data"
JOB_ID: "flap_pruning"
CACHE_PATH: "user_data/.cache"
JOB_PATH: "user_data/jobs/flap_pruning"
LOGGING_PATH: "user_data/logs/flap_pruning"
ALGO_TYPE: "llm"
TASK: "llm"
```

## Running the Pruning

With your YAML file configured, initiate the pruning process by running:

```bash
python main.py --yaml_path examples/text-generation/flap_pruning/config.yaml
```

Monitor the process to ensure that the pruning completes successfully.

Once the job starts, you'll find the following directory structure in the `user_data` folder:

```bash
user_data/
├── datasets
│   └── wikitext
├── jobs
│   └── Kompress
│       └── flap_pruning
├── logs
│   └── flap_pruning
└── models
    └── meta-llama
        └── Meta-Llama-3.1-70B-Instruct
```

The pruned model will be saved in the `user_data/jobs/Kompress/flap_pruning` directory.
```bash
user_data/
└── jobs
    └── Kompress
        └── flap_pruning
            ├── config.json
            ├── generation_config.json
            ├── model-00001-of-00019.safetensors
            ├── model-00002-of-00019.safetensors
            ├── model-00003-of-00019.safetensors
            ├── model-00004-of-00019.safetensors
            ├── model-00005-of-00019.safetensors
            ├── model-00006-of-00019.safetensors
            ├── model-00007-of-00019.safetensors
            ├── model-00008-of-00019.safetensors
            ├── model-00009-of-00019.safetensors
            ├── model-00010-of-00019.safetensors
            ├── model-00011-of-00019.safetensors
            ├── model-00012-of-00019.safetensors
            ├── model-00013-of-00019.safetensors
            ├── model-00014-of-00019.safetensors
            ├── model-00015-of-00019.safetensors
            ├── model-00016-of-00019.safetensors
            ├── model-00017-of-00019.safetensors
            ├── model-00018-of-00019.safetensors
            ├── model-00019-of-00019.safetensors
            ├── model.safetensors.index.json
            ├── prune_config.json
            ├── special_tokens_map.json
            ├── tokenizer.json
            └── tokenizer_config.json
```

## Performance Evaluation

After the pruning process, evaluate the performance of the pruned model using the evaluation script provided.

```bash
pip install lm-eval git+https://github.com/PanQiWei/AutoGPTQ.git

python examples/text-generation/flap_pruning/evaluate.py \
  --base_model "meta-llama/Meta-Llama-3.1-70B-Instruct" \
  --pruned_model "user_data/jobs/Kompress/flap_pruning/" \
  --tasks "arc_challenge:25,gptq_wikitext:0" \
  --results "user_data/evals/meta-llama_3.1-70b" \
  --cache_dir "user_data/.cache"

```

Compare the results with the original model to assess the impact of pruning on accuracy and inference speed.


| Model       	| Task                        	| Metric       	| Baseline 	| Pruned   	| Impact 	|
|-------------	|-----------------------------	|--------------	|----------	|----------	|--------	|
| Llama3.1-70b 	| Size (in GB)                	| Model Size ↓ 	| 132.0    	| 85.0     	| -47.0 	|
| Llama3.1-70b 	| Perplexity (wikitext, gptq) 	| Perplexity ↓ 	| 3.79     	| 84146.23 	| > +84k 	|
| Llama3.1-70b 	| ARC Challenge (25 shot)     	| Accuracy ↑   	| 70.48    	| 27.82    	| -42.66 	|

<br>

---

*Author: [Kushwaha, Shubham](https://www.linkedin.com/in/shwoobham/)*

### Additional Examples

- **Pruning - 0.5x Pruned Llama3.1-70b**
- **LMQuant - 4-8-4 Quantisation (w4a8kv4) of Llama3.1-70b**
- **AQLM - Sub 4-bit (w2a16) Quantisation of Llama3.1-70b**
- **TensorRTLLM - Accelerating a 4-bit Quantised Llama3.1-70b**

---