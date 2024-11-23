---
layout: post
title:  "Functions, Tools, and Agents"
date:   2024-11-20
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
```python
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
```python
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
```python
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
```python
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
```python
runnable.invoke({"input": "how did the patriots do yesterday?"})
```
**Output:**
```python
AIMessage(content='', additional_kwargs={'function_call': {'name': 'sports_search', 'arguments': '{"team_name":"New England Patriots"}'}})
```
- `content` is empty because the LLM determined that a function needs to be called in order to generate a response.
- `function_call` containts the following information:
```python
'function_call': {
   'name': 'sports_search', # the custom function that needs to be called 
   'arguments': '{"team_name":"New England Patriots"}' # the arguments to the function.
}
```
The LLM automatically determined that `patriots` in the user query referred to the `New England Patriots` which is a sports team. Interesting!

Now, let's invoke the chain with a weather related question:
```python
runnable.invoke({"input": "what is the weather in sf"})
```
**Output:**
```python
AIMessage(content='', additional_kwargs={'function_call': {'name': 'weather_search', 'arguments': '{"airport_code":"SFO"}'}})
```
This time, the LLM determined that the function `weather_search` needs to be called with the argument `airport_code=SFO` where `SFO` stands for San Francisco!

By default, if none of the defined functions are relevant to the user query, the LLM will not invoke function-calling and generate a normal user (natural language) response. It doesn't forcibly use any function or hallucinate its arguments - until explicitely made to do so.


## Use pydantic to create functions with ease

Pydantic is a Python library for data validation using Python-type annotations. It ensures that the data you work with matches your specified data types, simplifying error handling and data parsing in Python applications.

Thus, Pydantic will help us define LLM functions with ease. Let's look at an example:

```python
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
```python
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
```python
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
```python
model_with_functions.invoke("what is the weather in sf?")
```
**Output:**
```python
AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{"airport_code":"SFO"}'}})
```

Invoking with artist related question:
```python
model_with_functions.invoke("what are three songs by taylor swift?")
```
**Output:**
```python
AIMessage(content='', additional_kwargs={'function_call': {'name': 'ArtistSearch', 'arguments': '{"artist_name":"Taylor Swift","n":3}'}})
```

Default behaviour:
```python
model_with_functions.invoke("hi!")
```
**Output:**
```python
AIMessage(content='Hello! How can I assist you today?')
```


## Extraction

Concept extraction is an NLP task that automatically identifies and extracts specific concepts or entities from unstructured text.

Let's see how function-calling can help us solve this task of extracting all the papers and their respective authors mentioned in an unstructured article with ease.

```python
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
```python
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

# Tools and Routing

1. **Tools:** Functions and services an LLM can utilize to extend its capabilities are named "tools" in LangChain. 

2. **Routing:** Selecting from multiple possible tools is called "routing".

## Tools

Let's create a function `get_current_temperature()` that calls an external weather API to get the temperature forecast for the next day.

Each function can be converted into a tool using the `@tool` decorator. `args_schema` specifies the format of the args for this function.

```python
from langchain.agents import tool

# Define the input schema
class OpenMeteoInput(BaseModel):
    latitude: float = Field(..., description="Latitude of the location to fetch weather data for")
    longitude: float = Field(..., description="Longitude of the location to fetch weather data for")

@tool(args_schema=OpenMeteoInput) # args_schema tells the format of the args for this function
def get_current_temperature(latitude: float, longitude: float) -> dict:
    """Fetch current temperature for given coordinates."""
    
    BASE_URL = "https://api.open-meteo.com/v1/forecast"
    
    # Parameters for the request
    params = {
        'latitude': latitude,
        'longitude': longitude,
        'hourly': 'temperature_2m',
        'forecast_days': 1,
    }

    # Make the request
    response = requests.get(BASE_URL, params=params)
    
    if response.status_code == 200:
        results = response.json()
    else:
        raise Exception(f"API Request failed with status code: {response.status_code}")

    current_utc_time = datetime.datetime.utcnow()
    time_list = [datetime.datetime.fromisoformat(time_str.replace('Z', '+00:00')) for time_str in results['hourly']['time']]
    temperature_list = results['hourly']['temperature_2m']
    
    closest_time_index = min(range(len(time_list)), key=lambda i: abs(time_list[i] - current_utc_time))
    current_temperature = temperature_list[closest_time_index]
    
    return f'The current temperature is {current_temperature}°C'
```

Every tool has the following properties:
```python
get_current_temperature.name
> 'get_current_temperature'

get_current_temperature.description
> 'get_current_temperature(latitude: float, longitude: float) -> dict - Fetch current temperature for given coordinates.'

# followed the defined schema
get_current_temperature.args
> "{'latitude': {'title': 'Latitude',
  'description': 'Latitude of the location to fetch weather data for',
  'type': 'number'},
 'longitude': {'title': 'Longitude',
  'description': 'Longitude of the location to fetch weather data for',
  'type': 'number'}}"
```

You can convert any tool into an openai function format.
```python
from langchain.tools.render import format_tool_to_openai_function

format_tool_to_openai_function(get_current_temperature)
> "{'name': 'get_current_temperature',
 'description': 'get_current_temperature(latitude: float, longitude: float) -> dict - Fetch current temperature for given coordinates.',
 'parameters': {'title': 'OpenMeteoInput',
  'type': 'object',
  'properties': {'latitude': {'title': 'Latitude',
    'description': 'Latitude of the location to fetch weather data for',
    'type': 'number'},
   'longitude': {'title': 'Longitude',
    'description': 'Longitude of the location to fetch weather data for',
    'type': 'number'}},
  'required': ['latitude', 'longitude']}}"
```

## Routing

Let's create another tool `search_wikipedia()`.
```python
import wikipedia
@tool
def search_wikipedia(query: str) -> str:
    """Run Wikipedia search and get page summaries."""
    page_titles = wikipedia.search(query)
    summaries = []
    for page_title in page_titles[: 3]:
        try:
            wiki_page =  wikipedia.page(title=page_title, auto_suggest=False)
            summaries.append(f"Page: {page_title}\nSummary: {wiki_page.summary}")
        except (
            self.wiki_client.exceptions.PageError,
            self.wiki_client.exceptions.DisambiguationError,
        ):
            pass
    if not summaries:
        return "No good Wikipedia Search Result was found"
    return "\n\n".join(summaries)
```

The response of running any tool can be either:
1. **AgentAction**: specifies what the next action should be in order to get the final response, i.e., call a specific function using arguments after analysing the user input query.
2. **AgentFinish**: provides final user content/response.

```python
from langchain.schema.agent import AgentFinish

# create a routing 
def route(result):
    # If the result is AgentFinish, no more action is needed.
    # return the final output to the user
    if isinstance(result, AgentFinish):
        return result.return_values['output']
    else:
        tools = {
            "search_wikipedia": search_wikipedia, 
            "get_current_temperature": get_current_temperature,
        }

        # call the function identified by the LLM with the corresponding args
        return tools[result.tool].run(result.tool_input)
```

```python
# create a chain that 
  # takes a prompt, 
  # identifies if function calling is required, 
  # parses the output, 
  # routes to the required function-call, executes the function to retrieve the final response.
chain = prompt | model | OpenAIFunctionsAgentOutputParser() | route

# This used AgentAction to call `get_current_weather()`
result = chain.invoke({"input": "What is the weather in san francisco right now?"})
> 'The current temperature is 12.1°C'

# This used AgentFinish since no function-call was required
chain.invoke({"input": "hi!"})
> 'Hello! How can I assist you today?'
```

# Conversational Agents

## What are agents?

A combination of LLMs and code. LLMs reason about what steps to take and call for actions.

## Agent Loop

1. Choose a tool to use.
2. Observe the output of the tool.
3. Repeat until a stopping condition is met.
    - LLM determined.
    - Hardcorded rules. 

Give the LLM all intermediate results and their observations for the LLM to identify the next steps. This is done by populating intermediate results in `agent_scratchpad` inside the prompt everytime the LLM is called.

```python
from langchain.schema.runnable import RunnablePassthrough

chain = prompt | model | OpenAIFunctionsAgentOutputParser()
# RunnablePassthrough helps in passing intermediate steps in the prompt everytime the LLM is called
agent_chain = RunnablePassthrough.assign(
    agent_scratchpad= lambda x: format_to_openai_functions(x["intermediate_steps"])
) | chain

def run_agent(user_input):
    intermediate_steps = []
    while True:
        result = agent_chain.invoke({
            "input": user_input, 
            "intermediate_steps": intermediate_steps
        })

        # if final output is received, return 
        if isinstance(result, AgentFinish):
            return result

        tool = {
            "search_wikipedia": search_wikipedia, 
            "get_current_temperature": get_current_temperature,
        }[result.tool]
        
        # get the observation from the respective tool
        observation = tool.run(result.tool_input)

        # populate the tool used and its observation in the prompt for next iteration
        intermediate_steps.append((result, observation))


run_agent("what is the weather is sf?")

> AgentFinish(return_values={'output': 'The current temperature in San Francisco is 11.8°C.'}, log='The current temperature in San Francisco is 11.8°C.')
```

`agent_executor` is a helpful abstraction to create e-2-e agents.
```python
from langchain.agents import AgentExecutor

agent_executor = AgentExecutor(agent=agent_chain, tools=tools, verbose=True)
agent_executor.invoke({"input": "what is langchain?"})
```

**Output:**
```
> Entering new AgentExecutor chain...

Invoking: `search_wikipedia` with `{'query': 'Langchain'}`

Page: LangChain
Summary: LangChain is a software framework that helps facilitate the integration of large language models (LLMs) into applications. As a language model integration framework
...
> Finished chain.

{'input': 'what is langchain?',
 'output': 'LangChain is a software framework that helps facilitate the integration of large language models (LLMs) into applications. It is used for document analysis and summarization, chatbots, and code analysis.'}
```

## Chat History

By default, LLMs don't have history of previous interactions. We can add the chat history using `ConversationBufferMemory`.

```python
from langchain.memory import ConversationBufferMemory

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are helpful but sassy assistant"),
    MessagesPlaceholder(variable_name="chat_history"), # populate the chat history inside prompt here
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad")
])

# keep the history inside memory buffer
memory = ConversationBufferMemory(return_messages=True, memory_key="chat_history")

agent_executor = AgentExecutor(agent=agent_chain, tools=tools, verbose=True, memory=memory)
```

**Output:**
```python
agent_executor.invoke({"input": "my name is bob"})
> Entering new AgentExecutor chain...
Hello Bob! How can I assist you today?

> Finished chain.
{'input': 'my name is bob',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Hello Bob! How can I assist you today?')],
 'output': 'Hello Bob! How can I assist you today?'}

----------

agent_executor.invoke({"input": "whats my name"})

> Entering new AgentExecutor chain...
Your name is Bob. How can I assist you today, Bob?

> Finished chain.
{'input': 'whats my name',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Hello Bob! How can I assist you today?'),
  HumanMessage(content='whats my name'),
  AIMessage(content='Your name is Bob. How can I assist you today, Bob?')],
 'output': 'Your name is Bob. How can I assist you today, Bob?'}
```

## Chatbot

Let's make a conversational bot using everything we learnt!!
Also, I am on a time-constraint right now, that's why I'm just re-iterating what the tutorial taught. But someday (when I have enough time and I'm feeling creative), I'll create a fun chatbot of my own.

```python
import panel as pn  # GUI
pn.extension()
import panel as pn
import param


# A helper class to conbine all concepts learnt above
class cbfs(param.Parameterized):
    
    def __init__(self, tools, **params):
        super(cbfs, self).__init__( **params)
        self.panels = []
        self.functions = [format_tool_to_openai_function(f) for f in tools]
        self.model = ChatOpenAI(temperature=0).bind(functions=self.functions)
        self.memory = ConversationBufferMemory(return_messages=True,memory_key="chat_history")
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", "You are helpful but sassy assistant"),
            MessagesPlaceholder(variable_name="chat_history"),
            ("user", "{input}"),
            MessagesPlaceholder(variable_name="agent_scratchpad")
        ])
        self.chain = RunnablePassthrough.assign(
            agent_scratchpad = lambda x: format_to_openai_functions(x["intermediate_steps"])
        ) | self.prompt | self.model | OpenAIFunctionsAgentOutputParser()
        self.qa = AgentExecutor(agent=self.chain, tools=tools, verbose=False, memory=self.memory)
    
    def convchain(self, query):
        if not query:
            return
        inp.value = ''
        result = self.qa.invoke({"input": query})
        self.answer = result['output'] 
        self.panels.extend([
            pn.Row('User:', pn.pane.Markdown(query, width=450)),
            pn.Row('ChatBot:', pn.pane.Markdown(self.answer, width=450, styles={'background-color': '#F6F6F6'}))
        ])
        return pn.WidgetBox(*self.panels, scroll=True)


    def clr_history(self,count=0):
        self.chat_history = []
        return 
```

Putting it all together in a nice simple UI.
```python
cb = cbfs(tools)
inp = pn.widgets.TextInput( placeholder='Enter text here…')
conversation = pn.bind(cb.convchain, inp) 

tab1 = pn.Column(
    pn.Row(inp),
    pn.layout.Divider(),
    pn.panel(conversation,  loading_indicator=True, height=400),
    pn.layout.Divider(),
)

dashboard = pn.Column(
    pn.Row(pn.pane.Markdown('# QnA_Bot')),
    pn.Tabs(('Conversation', tab1))
)
dashboard
```

<div class="row justify-content-sm-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/chatbot.png" title="image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

---------------

That's all folks! I have a few other agent concepts to learn in the pipeline. Let the learning continue 😄.
