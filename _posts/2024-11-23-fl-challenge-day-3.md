---
layout: post
title:  "Federated Fine-Tuning of LLMs with Private Data"
date:   2024-11-23
description: ""
tag:
- opensource
- ai
- federated-learning
- openmined
- challenge
- data-privacy
- trustworthy-ai
thumbnail: https://miro.medium.com/v2/resize:fit:1200/1*Hnm8Txe9NPX3TylnmwzQfw.png
categories: open-source
giscus_comments: true
---

It's Day 3 of the [#30DaysOfFLCode](https://info.openmined.org/30daysofflcode) Challenge by [OpenMined](https://openmined.org/) 🥳!

Today I went through a tutorial: [Federated Fine-Tuning of LLMs with Private Data](https://learn.deeplearning.ai/courses/intro-to-federated-learning-c2) by [DeepLearning.AI](https://learn.deeplearning.ai/) and [Flower Labs](https://flower.ai/).

Let's quickly learn a few concepts ⬇️.

# Introduction

<add image>

A lot of private data was able to be extracted and reconstructed from multiple open source LLMs' pre-training dataset. Federated Learning is the missing piece of LLMs and private data. This is an alternative to conventional LLM fine-tuning, allowing you to avoid:
- using closed fune-tuning APIs.
- copying into a 3rd party cloud resources.

Key strengths of Federated LLM fine-tuning are:
- Improved privacy of data
- Wider range of data sources
- Tolerable communication overhead.


# Centralized LLM Fine-Tuning

We would be working with a medical private data scenario, but the concepts can be extended to any industry with private datasets. The idea is that different hospitals have different sets of private data which would be cenrally collected. This entire data will then be used to fine-tune an LLM and the final model will be shared with all the participating hospitals.

We are going to use `medAlpaca` <add link> - an open source collection of medical conversational AI training data and models. It has the following properties:
- 50k training examples
- includes Q&A pairs
- variety of medical domain knowledge

peft + lora will be used for fine-tuning
<explain config>
```
dataset:
  name: medalpaca/medical_meadow_medical_flashcards
model:
  name: EleutherAI/pythia-70m
  quantization: 4
  gradient_checkpointing: true
  use_fast_tokenizer: true
  lora:
    peft_lora_r: 16
    peft_lora_alpha: 64
    target_modules: null
train:
  save_every_round: 5
  seq_length: 512
  padding_side: left
  training_arguments:
    learning_rate: 0.0005
    per_device_train_batch_size: 2
    gradient_accumulation_steps: 1
    logging_steps: 1
    max_steps: 5
    report_to: null
    save_steps: 200
    save_total_limit: 10
    gradient_checkpointing: true
    lr_scheduler_type: cosine
```

Generic base LLMs imrpove on domain-specific prompts after fine-tuning with private data.

mistral-7b fine-tuned with medalpaca is significantly better under medical prompts

However, this finetuning is performed centrally. None of the key challenges of private data have been addressed. 
- privacy
- regulation
- data volume

federeated LLM fine-tuning is the answer.


# Federated LLM Fine-Tuning

<all images>

<img about analysis>

<img communication>



# References

[1] [https://learn.deeplearning.ai/courses/intro-to-federated-learning-c2](https://learn.deeplearning.ai/courses/intro-to-federated-learning-c2)
