---
layout: post
title:  "Privacy Preserving AI"
date:   2024-11-22
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

It's Day 2 of the [#30DaysOfFLCode](https://info.openmined.org/30daysofflcode) Challenge by [OpenMined](https://openmined.org/) 🥳!

Today I went through the video: [Privacy Preserving AI by Andrew Trask: MIT Deep Learning Series](https://www.youtube.com/watch?v=4zrU54VIK6k).

Below are my notes from the talk ⬇️.

# Tools for Remote Data Science 📉

> Can we answer questions using data we cannot see?

There are multiple tools for being able to use the data we donot have to answer important questions.

## Tool 1: Remote Execution

[PySyft](https://github.com/OpenMined/PySyft) is a library that helps to coordinate with data and computations remotely. It is a wrapper over PyTorch and has the ability to instantiate a Virtual Worker where all the computation will happen.

## Tool 2: Search and Example Data

[PyGrid](https://blog.openmined.org/what-is-pygrid-demo/) is a platform that:
- helps to search and access snippets of actual data, 
- answer relevant statistical questions regarding the data distribution, 

such that it will help later to perform pre-processing locally. This enables individual parties to do basic feature engineering with sample data.

However, there is a possibility of stealing/reconstructing sensitive data using the `.get()` method (sample data).

## Tool 3: Differential Privacy

Differential Privacy is a framework that ensures an algorithm's output does not reveal whether any specific individual’s data was included in the input dataset. Formally, a mechanism is $(\epsilon, \delta)$-differentially private if, for any two datasets $D_1$ and $D_2$ differing in at most one record, and for all possible outputs $S$:

$$[
P[\mathcal{M}(D_1) \in S] \leq e^\epsilon \cdot P[\mathcal{M}(D_2) \in S] + \delta
]
$$

This guarantees that the output remains statistically similar regardless of an individual’s participation, ensuring privacy.

### Sensitivity:
Sensitivity quantifies the maximum change in an algorithm's output due to the inclusion or exclusion of a single data point in the dataset. It is defined as:

$$
[
\Delta f = \max_{D_1, D_2} ||f(D_1) - f(D_2)||
]
$$

where $f$ is the query or function, and $D_1, D_2$ are datasets differing in one record. Lower sensitivity functions make it easier to achieve differential privacy.

### Epsilon $\epsilon$ (Privacy Budget):
Epsilon is a parameter that measures the privacy loss in a differentially private mechanism. Smaller values of $\epsilon$ provide stronger privacy guarantees but reduce the accuracy of the results. Conversely, larger $\epsilon$ allows better accuracy but weaker privacy. 

The "privacy budget" refers to the cumulative $\epsilon$ used across multiple queries; once it's exhausted, the mechanism risks breaching privacy guarantees.

### Types of Differential Privacy:

#### Local Differential Privacy (LDP):
LDP ensures privacy at the individual data level, meaning each data contributor perturbs their model weights before sharing it with the server. The server never sees raw data. This approach requires:

- Trust in the data contributors.
- Strong privacy guarantees since no raw data is exposed.

**Best use case:** Scenarios where users want to ensure absolute privacy without trusting the server, such as in surveys or telemetry data collection.

**Limitation:** Results may be noisier compared to global DP due to individual perturbation.

#### Global Differential Privacy (Centralized DP):
In global DP, raw data is collected centrally, and noise is added at the aggregate level (e.g., during analysis). The server is assumed to be trusted not to misuse raw data.

**Advantages:**
- Produces more accurate results since perturbation happens after aggregation.
- Better for large-scale analysis, like training machine learning models.

**Risk:** Requires trusting the server, which could potentially compromise the raw data.

**Best Way to Use Local DP vs Global DP:**
- **Local DP:** Use when trust in the server is low, or contributors demand high privacy.
- **Global DP:** Use when high accuracy is critical, and there is sufficient trust in the central server's security.

> "Multiple companies' entire business model is buy anonymized datasets, merge them together to de-anonymize them, and sell market intelligence to insurance companies". - Andrew Trask

The idea of the tools is to allow a set no. of queries that do not exceed the epsilon (privacy budget). This will give privacy guarantees for the sensitive dataset.

### Who sets the privacy budget?

- Not the data scientist because thats a conflict of interest.

- Potentially data owners (e.g. hospitals), but this still leads back to the same issue; 
    - 1 hospital releases 0.1 eps of data,
    - another releases 0.1 eps of separate data,
    - total publicly available data of a common user = 0.2.
    - imagine if eps budget is 0.15, this is already exceeded --> privacy is lost!

- Ideally individuals should be allowed to set the budget. However, it's too far a goal for now.

However, a few issues with DP are:
- we cannot do join computations anymore inherently due to the statistically added noise.
- the data is safe but the model can be corrupted by individual parties.
- need to trust the remote server for not corrupting the model by doing something malicious.

## Tool 4: Secure Multi-Party Computation

Its a cryptographic protocol that allows multiple parties to jointly compute a function over their inputs while keeping those inputs private. None of the parties learns anything about the other participants' inputs except what can be inferred from the output of the computation.

In SMPC, each participant's data is split into encrypted or obfuscated shares and distributed among the parties. The computation is then carried out on these shares without revealing the underlying data. After computation, the final result is reconstructed, ensuring:

- **Privacy**: No party learns anything about the private inputs except the output.
- **Correctness**: The result of the computation is accurate and consistent with the function.
- **Security**: Colluding subsets of participants (below a certain threshold) cannot learn private information.

> The implication is that multiple people can share ownership of a number. And models and datasets are just large collections of numbers which we can encrypt.

This provides 2 benefits:
1. **Encryptions:** Neither knows the hidden value.
2. **Shared Governance**: The number can only be decrypted till all of the shareholders agree.

#### Advantages

- Models can  be encrypted during training
- We can do joins/ functions across data owners.

#### Disadvantages
- It is computationally expensive.


# AI, Privacy, Society

3 broad non-exhaustive categories are important for the future of AI.

## 1. Open Data for Science

> "Everyone's gonna work on models, but if you look historically, the biggest jumps in progress have happened when we had new big datasets - or the ability to process them." - Phil Blunsom

## 2. Single-Use Accountability

Build systems that are able to answer questions you really need to answer and nothing else. This makes it privacy preserving.

E.g., Using raw data to answer what causes cancer
- while not knowing who all have it
- while not being able to answer any other query other than the objective

is privacy-preserving.

## 3. End-to-End Encrypted Services

Provide services (model computations and forecasts) without the need for knowing individual data is privacy-preserving.

- Models can be encrypted.
- Data can be encrypted.

-----

That is all for today folks! Blogging + learning is a long process haha, but I'm glad I learnt a bunch today!
See you tomorrow :D


# References

[1] [https://www.youtube.com/watch?v=4zrU54VIK6k](https://www.youtube.com/watch?v=4zrU54VIK6k)
