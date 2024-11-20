---
layout: post
title:  "Functions, Tools, and Agents"
date:   2024-11-01
description: "Understand the capabilities of LLMs to the fullest."
tag:
- opensource
- ai
- github
thumbnail: https://theblue.ai/wp-content/uploads/2024/06/LLM-Agent-LLM-Agents-AI-Agent.png
categories: open-source
giscus_comments: true
---

# Function Calling

Function calling in large language models (LLMs) is a technique that allows LLMs to interact with external tools and APIs by: 

1. **Detecting when a function is needed:** LLMs are fine-tuned to identify when a function needs to be called based on a user's prompt.

2. **Generating a structured response:** If applicable, LLMs generate a JSON object that contains the function name and arguments.

3. **Executing the function:** The developer's code executes the function using the extracted arguments and to get an output.

4. **Using the function output:** The LLM uses the function output to generate a final response in natural language to the user.

In this post, I will explain the common structure of function-calling, tools, and agents using LangChain and OpenAI models.


## LangChain Expression Language (LCEL)

LangChain is an open-source framework that allows developers to build applications using large language models (LLMs). The LangChain Expression Language (LCEL) helps to compose a chain of LangChain components with minimal code.

Typically, a chain contains the following components seperated by the `|` operator:
> Prompt | LLM | OutputParser

Below is a code example to use LCEL and an LLM to generate a chain.
```
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StrOutputParser

# Create a prompt
prompt = ChatPromptTemplate.from_template(
    "tell me a short joke about {topic}"
)

# Instantiate an LLM
model = ChatOpenAI()

# Instantiate a parser that converts model output to string
output_parser = StrOutputParser()

# Create a langChain "chain" of components using LCEL
chain = prompt | model | output_parser

# Invoke the chain to generate a response from the LLM
chain.invoke({"topic": "bears"})
```
**Output:**
```
"Why do bears have hairy coats?\n\nBecause they don't like to shave!"
```

1. The user creates a string prompt. 
2. This prompt is sent to the LLM to generate an output.
3. The LLM output object is properly parsed to generate a string response for the user.


## Bind custom functions to LLMs

The purpose of function-calling is:
1. To leverage an LLM to analyse a a user query in order to determine whether a function-call is required to generate the desired response.
2. If a function call is required, the LLM needs to figure out the correct function to call along with the required function argument values.

This process helps the developer to then execute the relevant function with the arguments to generate the desired response. A function could be a custom function written by the developer or an API call that generates an output based on given arguments.

OpenAI models expect the functions to be defined in the following format:
```
functions = [
   {
      "name": <function_name>,
      "description": <function description>,
      "parameters": {
        "type": "object",
        "properties": {
          <argument_name>: {
            "type": <argument_type>,
            "description": <argument_description>
          },
        },
        "required": [<list of required arguments>]
      }
   },
...
]
```

Let's take an example with 2 custom defined functions `weather_search` and `sports_search`:
```
# Define the functions
functions = [
    {
      "name": "weather_search",
      "description": "Search for weather given an airport code",
      "parameters": {
        "type": "object",
        "properties": {
          "airport_code": {
            "type": "string",
            "description": "The airport code to get the weather for"
          },
        },
        "required": ["airport_code"]
      }
    },
        {
      "name": "sports_search",
      "description": "Search for news of recent sport events",
      "parameters": {
        "type": "object",
        "properties": {
          "team_name": {
            "type": "string",
            "description": "The sports team to search for"
          },
        },
        "required": ["team_name"]
      }
    }
  ]

# Create a prompt
prompt = ChatPromptTemplate.from_messages(
    [
        ("human", "{input}")
    ]
)

# Instantiate an LLM and "bind" the defined functions with the LLM.
# This tells the LLM the list of available functions to choose from in case function-calling is required.
model = ChatOpenAI(temperature=0).bind(functions=functions)

# Create a chain
runnable = prompt | model
```

Invoke the chain with a sports-related question:
```
runnable.invoke({"input": "how did the patriots do yesterday?"})
```
**Output:**
```
AIMessage(content='', additional_kwargs={'function_call': {'name': 'sports_search', 'arguments': '{"team_name":"New England Patriots"}'}})
```
- `content` is empty because the LLM determined that a function needs to be called in order to generate a response.
- `function_call` containts the following information:
```
'function_call': {
   'name': 'sports_search', # the custom function that needs to be called 
   'arguments': '{"team_name":"New England Patriots"}' # the arguments to the function.
}
```
The LLM automatically determined that `patriots` in the user query referred to the `New England Patriots` which is a sports team. Interesting!

Now, let's invoke the chain with a weather related question:
```
runnable.invoke({"input": "what is the weather in sf"})
```
**Output:**
```
AIMessage(content='', additional_kwargs={'function_call': {'name': 'weather_search', 'arguments': '{"airport_code":"SFO"}'}})
```
This time, the LLM determined that the function `weather_search` needs to be called with the argument `airport_code=SFO` where `SFO` stands for San Francisco!

By default, if none of the defined functions are relevant to the user query, the LLM will not invoke function-calling and generate a normal user (natural language) response. It doesn't forcibly use any function or hallucinate its arguments - until explicitely made to do so.


## Use pydantic to create functions with ease

Pydantic is a Python library for data validation using Python-type annotations. It ensures that the data you work with matches your specified data types, simplifying error handling and data parsing in Python applications.

Thus, Pydantic will help us define LLM functions with ease. Let's look at an example:

```
# Define a Pydantic class which refers to a function
# The fields of the class refer to function arguments
# The docstring used to describe the class is mandatory as it helps the LLM understand what the purpose of the function is. 

class WeatherSearch(BaseModel):
    """Call this with an airport code to get the weather at that airport"""
    airport_code: str = Field(description="airport code to get weather for")

from langchain.utils.openai_functions import convert_pydantic_to_openai_function
convert_pydantic_to_openai_function(WeatherSearch)
```
**Output:**
```
{'name': 'WeatherSearch', # name of the class
 'description': 'Call this with an airport code to get the weather at that airport', # Pulled from the docstring used to describe the class.
 'parameters': {'title': 'WeatherSearch',
  'description': 'Call this with an airport code to get the weather at that airport',
  'type': 'object',
  'properties': {'airport_code': {'title': 'Airport Code',
    'description': 'airport code to get weather for',
    'type': 'string'}},
  'required': ['airport_code']}}
```

Let's create multiple functions using Pydantic!
```
# Custom function with 2 args
class ArtistSearch(BaseModel):
    """Call this to get the names of songs by a particular artist"""
    artist_name: str = Field(description="name of artist to look up")
    n: int = Field(description="number of results")

functions = [
    convert_pydantic_to_openai_function(WeatherSearch),
    convert_pydantic_to_openai_function(ArtistSearch),
]

# Bind the functions to the LLM
model_with_functions = model.bind(functions=functions)
```

Invoking with weather related question:
```
model_with_functions.invoke("what is the weather in sf?")
```
**Output:**
```
AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{"airport_code":"SFO"}'}})
```

Invoking with artist related question:
```
model_with_functions.invoke("what are three songs by taylor swift?")
```
**Output:**
```
AIMessage(content='', additional_kwargs={'function_call': {'name': 'ArtistSearch', 'arguments': '{"artist_name":"Taylor Swift","n":3}'}})
```

Default behaviour:
```
model_with_functions.invoke("hi!")
```
**Output:**
```
AIMessage(content='Hello! How can I assist you today?')
```


## Extraction

Concept extraction is an NLP task that automatically identifies and extracts specific concepts or entities from unstructured text.

Let's see how function-calling can help us solve this task of extracting all the papers and their respective authors mentioned in an unstructured article with ease.

```
# Load unstructured text from the internet
from langchain.document_loaders import WebBaseLoader
loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
documents = loader.load()
page_content = documents[0].page_content[:10000]

# Create a prompt
template = """A article will be passed to you. Extract from it all papers that are mentioned by this article followed by its author. 

Do not extract the name of the article itself. If no papers are mentioned that's fine - you don't need to extract any! Just return an empty list.

Do not make up or guess ANY extra information. Only extract what exactly is in the text."""

prompt = ChatPromptTemplate.from_messages([
    ("system", template),
    ("human", "{input}")
])

# Create custom function definitions
class Paper(BaseModel):
    """Information about papers mentioned."""
    title: str
    author: Optional[str]

class Info(BaseModel):
    """Information to extract"""
    papers: List[Paper]

paper_extraction_function = [
    convert_pydantic_to_openai_function(Info)
]

# Bind the function to the LLM
extraction_model = model.bind(
    functions=paper_extraction_function, # functions present for binding
    function_call={"name":"Info"} # Force the LLM to use the function "Info" everytime
)

# Create a chain and invoke it with the article
extraction_chain = prompt | extraction_model | JsonKeyOutputFunctionsParser(key_name="papers")
extraction_chain.invoke({"input": page_content})
```

**Output:**
```
[{'title': 'Chain of thought (CoT; Wei et al. 2022)', 'author': 'Wei et al. 2022'},
 {'title': 'Tree of Thoughts (Yao et al. 2023)', 'author': 'Yao et al. 2023'},
 {'title': 'LLM+P (Liu et al. 2023)', 'author': 'Liu et al. 2023'},
 {'title': 'ReAct (Yao et al. 2023)', 'author': 'Yao et al. 2023'},
 {'title': 'Reflexion (Shinn & Labash 2023)', 'author': 'Shinn & Labash 2023'},
 {'title': 'Chain of Hindsight (CoH; Liu et al. 2023)', 'author': 'Liu et al. 2023'},
 {'title': 'Algorithm Distillation (AD; Laskin et al. 2023)', 'author': 'Laskin et al. 2023'}]
```

The chain has successfully extracted all the papers that were mentioned in the article along with its authors in a structured JSON format.

That's all for function-calling (for now)!

# Tools 

# Agents








