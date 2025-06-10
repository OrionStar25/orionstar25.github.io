---
layout: post
title:  "Agentic RAG with LlamaIndex"
date:   2025-05-21
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

## Router Engine

Given a query, the router will pick 1 of several engines to answer the query. Provides dynamic query understanding capabilities.

E.g., RAG over a single document that does QA as well as summarization.

```python
from llama_index.core import SummaryIndex, VectorStoreIndex

summary_index = SummaryIndex(nodes)
vector_index = VectorStoreIndex(nodes)

summary_query_engine = summary_index.as_query_engine(
    response_mode="tree_summarize",
    use_async=True,
)
vector_query_engine = vector_index.as_query_engine()
```

```python
from llama_index.core.tools import QueryEngineTool


summary_tool = QueryEngineTool.from_defaults(
    query_engine=summary_query_engine,
    description=(
        "Useful for summarization questions related to MetaGPT"
    ),
)

vector_tool = QueryEngineTool.from_defaults(
    query_engine=vector_query_engine,
    description=(
        "Useful for retrieving specific context from the MetaGPT paper."
    ),
)
```

```python
from llama_index.core.query_engine.router_query_engine import RouterQueryEngine
from llama_index.core.selectors import LLMSingleSelector


query_engine = RouterQueryEngine(
    selector=LLMSingleSelector.from_defaults() # Can use pydantic selectors too,
    query_engine_tools=[
        summary_tool,
        vector_tool,
    ],
    verbose=True
)
```


## Build an Agent Reasoning Loop

### Agent Runner
- overall task dispatcher - creating task 
- orchestrating runs of agent workers on top of the given task
- return final response to the user
- the whole interface is behind the agent runner

### Agent Worker
- executing the next step of the agent

```python
from llama_index.core.agent import FunctionCallingAgentWorker
from llama_index.core.agent import AgentRunner

agent_worker = FunctionCallingAgentWorker.from_tools(
    [vector_tool, summary_tool], 
    llm=llm, 
    verbose=True
)
agent = AgentRunner(agent_worker)
```

```python
response = agent.query(
    "Tell me about the agent roles in MetaGPT, "
    "and then how they communicate with each other."
)
```

Response:

> === Calling Function ===
Calling function: summary_tool_metagpt with args: {"input": "agent roles in MetaGPT"}

> === Function Output ===
The agent roles in MetaGPT include Product Manager, Architect, Project Manager, Engineer, and QA Engineer. Each role has specific responsibilities and expertise tailored to different aspects of the software development process, such as analyzing user requirements, translating requirements into system design components, handling task distribution, executing code, formulating test cases, and ensuring code quality. These roles work together in a sequential manner following Standard Operating Procedures (SOPs) to efficiently complete complex software development tasks.

> === Calling Function ===
Calling function: summary_tool_metagpt with args: {"input": "communication between agent roles in MetaGPT"}

> === Function Output ===
Communication between agent roles in MetaGPT is structured and facilitated through mechanisms such as message pools, subscriptions, documents, and diagrams. Agents publish structured messages in a shared message pool and can subscribe to relevant messages based on their profiles. This approach ensures efficient exchange of information among agents, allowing for transparent access to task-related information while avoiding distractions from irrelevant details. The communication process involves clear handoffs between roles, starting from the Product Manager creating a detailed Product Requirement Document (PRD) to the Engineers working on completing development tasks. Additionally, MetaGPT implements natural language programming capabilities for interactive communication between agents, enabling effective collaboration and transparency in interactions.

> === LLM Response ===
In MetaGPT, the agent roles include Product Manager, Architect, Project Manager, Engineer, and QA Engineer. These roles have specific responsibilities tailored to different aspects of the software development process. They work together in a structured manner following Standard Operating Procedures (SOPs) to efficiently complete tasks.
Communication between agent roles in MetaGPT is facilitated through mechanisms like message pools, subscriptions, documents, and diagrams. Agents publish messages in a shared pool and subscribe to relevant ones based on their profiles. This structured approach ensures efficient information exchange, transparent access to task-related details, and clear handoffs between roles. Additionally, natural language programming capabilities enable interactive communication, fostering effective collaboration and transparency.



### Conversation history

- use `agent.chat()` instead of `agent.query()`.

### Agent Control

1. Decoupling of Task Creation and Execution:
    - Users gain the flexibility to schedule task execution according to their needs.

2. Enhanced debuggability:
    - Offers deeper insights into each step of the execution process.

3. Steerability:
    - Allows users to directly modify intermediate steps and incorporate human feedback for refined control.

```python
task = agent.create_task(
    "Tell me about the agent roles in MetaGPT, "
    "and then how they communicate with each other."
)

step_output = agent.run_step(task.task_id) ## run the first task

upcoming_steps = agent.get_upcoming_steps(task.task_id)
print(f"Num upcoming steps for task {task.task_id}: {len(upcoming_steps)}") ## 1


step_output = agent.run_step(
    task.task_id, input="What about how agents share information?"
)
```

> Added user message to memory: What about how agents share information?
=== Calling Function ===
Calling function: summary_tool_metagpt with args: {"input": "how agents share information in MetaGPT"}
=== Function Output ===
Agents in MetaGPT share information through a structured communication protocol that includes a shared message pool for publishing and accessing structured messages. They can also subscribe to relevant messages based on their profiles, ensuring they receive task-related information. Additionally, agents review previous feedback to adjust constraint prompts before each project, summarizing received feedback to improve the multi-agent system. The workflow involves sharing information through a Product Requirement Document, system design by the Architect, task assignment by the Project Manager, and code review by the QA Engineer. This structured process ensures efficient information flow among different agents in MetaGPT.

```python
response = agent.finalize_response(task.task_id)
```

> assistant: Agents in MetaGPT share information through a structured communication protocol that includes a shared message pool for publishing and accessing structured messages. They can also subscribe to relevant messages based on their profiles, ensuring they receive task-related information. Additionally, agents review previous feedback to adjust constraint prompts before each project, summarizing received feedback to improve the multi-agent system. The workflow involves sharing information through a Product Requirement Document, system design by the Architect, task assignment by the Project Manager, and code review by the QA Engineer. This structured process ensures efficient information flow among different agents in MetaGPT.


## Building multi-document agent

When you have a large no. of documents,
    - first retrieve a small set of relevant tools
    - feed the relevant tools to the agent reasoning prompt
    - get answer


```python
from utils import get_doc_tools
from pathlib import Path

paper_to_tools_dict = {}
for paper in papers:
    print(f"Getting tools for paper: {paper}")
    vector_tool, summary_tool = get_doc_tools(paper, Path(paper).stem)
    paper_to_tools_dict[paper] = [vector_tool, summary_tool]
```

```python
all_tools = [t for paper in papers for t in paper_to_tools_dict[paper]]

# define an "object" index and retriever over these tools
from llama_index.core import VectorStoreIndex
from llama_index.core.objects import ObjectIndex

obj_index = ObjectIndex.from_objects(
    all_tools,
    index_cls=VectorStoreIndex,
)

obj_retriever = obj_index.as_retriever(similarity_top_k=3)

tools = obj_retriever.retrieve(
    "Tell me about the eval dataset used in MetaGPT and SWE-Bench"
)
```

```python
from llama_index.core.agent import FunctionCallingAgentWorker
from llama_index.core.agent import AgentRunner

agent_worker = FunctionCallingAgentWorker.from_tools(
    tool_retriever=obj_retriever,
    llm=llm, 
    system_prompt=""" \
You are an agent designed to answer queries over a set of given papers.
Please always use the tools provided to answer a question. Do not rely on prior knowledge.\

""",
    verbose=True
)
agent = AgentRunner(agent_worker)
```

```python
# Queries across multiple docs
response = agent.query(
    "Compare and contrast the LoRA papers (LongLoRA, LoftQ). "
    "Analyze the approach in each paper first. "
)
```

