---
layout: post
title:  "[WIP] The RAG Triad - Metrics to evaluate Advanced RAG"
date:   2024-06-25
description: "Evaluate advanced RAG using meaningful metrics such as answer relevance, context relevance, and groundedness."
tag:
- rag
- evaluation
- opensource
- mistral
- generative ai
thumbnail: assets/img/triad.png
categories: artificial-intelligence
giscus_comments: true
---

https://docs.google.com/presentation/d/14zE7RMad5OAp3IknnjqGdfdmKwiJObo_Wp6ikSwPxCo/edit?usp=sharing
[Recording](https://x.com/GDGCloudMumbai/status/1804489370801676375), [Slides](https://docs.google.com/presentation/d/14zE7RMad5OAp3IknnjqGdfdmKwiJObo_Wp6ikSwPxCo/edit?usp=sharing)
https://github.com/OrionStar25/Build-and-Evaluate-RAGs

https://huggingface.co/learn/cookbook/en/rag_evaluation
https://python.langchain.com/v0.1/docs/integrations/chat/google_vertex_ai_palm/
https://cloud.google.com/vertex-ai/docs/tutorials/jupyter-notebooks#vertex-ai-workbench

start with:

1. what is rag?
    - how is it different than fie-tuning?
    - ways of evaluating rag
        - humans
        - static scores
            - precision
            - recall
            - bleu, rouge
        - use another llm
            - since broad capabilities
            - varied human preferences


2. Steps in evaluating RAG
    - Build RAG
    - Build a evaluation dataset
        - manually create dataset
        - use an llm to create dataset
        - use another llm to filter out relevant questions
    - Use another LLM as a judge to critic on evaluation set

3. lots of things to tweak in the RAG pipeline - but there should be a proper way to evaluate its impact

4. dataset used is huggingface documentation
    - text + source

5. synthetic evaluation dataset creation
    - input: randomised context from knowledge base
        - text+metadata
        - chunk documents into equal-sized chunks with overlap for continuity

    - output: 
        - possible question from the context asked by a user
            - force llm to not mention anything about using a context to generate this question (user never had a context)
        - answer to the given question supported by the context.

    - use LLM A (Mixtral 7B-instruct) to create this set.
        - The Mixtral-8x7B Large Language Model (LLM) is a pretrained generative Sparse Mixture of Experts (8 experts).
        - The Mixtral-8x7B outperforms Llama 2 70B on most benchmarks.

    - generate at least > 200 QA pairs since half of these will get filtered out + evaluation set should be at least ~100 questions.

    - critique agent:
        - rate each question on several criteria
        - evaluation criteria:
            - groundedness of question within context
            - relevance of questions wrt to expected task
            - stand-alone of questions without any context
        - give score 1 to 5 and remove all questions that have low score for any criteria
        - use same LLM A to critique

6. Build RAG
    - split documents using recursive split
    - embed documents
        - model used: thenlper/gte-small
    - retrieve relevant chunks - like a search engine
        - FAISS index stores embeddings for quick retrievals of similar chunks
    - use retrieved contexts + query to formulate answer
        - rerank retrieved context
            - retrieve 30 docs
            - rerank and output best 7
            - model used: colbert-ir/colbertv2.0
            - reevaluates and reorders the documents retrieved by the initial search based on their relevance to the query
        - answer using llm
            - LLM B (zephyr-7b-beta)
            - a fine-tuned version of (mistralai/Mistral-7B-v0.1) trained to act as helpful assistants.

7. Evaluate RAG
    - use answer correctness as a metric
        - use a scoring rubric to ground the evaluation
    - use LLM C (chatvertexai/palm2)
    - should use other metrics such as context relevance, faithfulness (groundedness)
    https://docs.ragas.io/en/latest/concepts/metrics/index.html





RAG Triads:
- answer relevance
    - is the answer relevant to the question
- context relevance
    - is the retrieved context relevant to the question asked
- groundedness
    - is the answer grounded in the context retrieved
