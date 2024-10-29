
# LangGraph Chatbot with Groq and LangChain

This project showcases an interactive chatbot built using [LangGraph](https://github.com/langgraph/langgraph), [LangChain](https://github.com/hwchase17/langchain), and [Groq](https://groq.com/). The chatbot leverages the `gemma2-9b-It` language model to answer user questions and interact in a natural language format. The design employs a `StateGraph` in LangGraph for state management, enabling dynamic conversation flow.

## Project Overview

* **Frameworks** : LangGraph for graph-based state management, LangChain for natural language processing, and Groq as the language model backend.
* **Goal** : Implement a chatbot that processes user queries, generates responses, and maintains conversation state effectively.
* **Visualization** : This README includes instructions for visualizing the LangGraph structure.

---

### Table of Contents

1. [Installation](#installation)
2. [Environment Setup](#environment-setup)
3. [Code Walkthrough](#code-walkthrough)
4. [LangGraph Overview](#langgraph-overview)
5. [Running the Chatbot](#running-the-chatbot)
6. [Graph Visualization](#graph-visualization)

---

## Installation

Before you begin, make sure you have Python installed. Then, install the required libraries:

<pre class="!overflow-visible"><div class="contain-inline-size rounded-md border-[0.5px] border-token-border-medium relative bg-token-sidebar-surface-primary dark:bg-gray-950"><div class="flex items-center text-token-text-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md h-9 bg-token-sidebar-surface-primary dark:bg-token-main-surface-secondary">bash</div><div class="sticky top-9 md:top-[5.75rem]"><div class="absolute bottom-0 right-2 flex h-9 items-center"><div class="flex items-center rounded bg-token-sidebar-surface-primary px-2 font-sans text-xs text-token-text-secondary dark:bg-token-main-surface-secondary"><span class="" data-state="closed"><button class="flex gap-1 items-center py-1"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M7 5C7 3.34315 8.34315 2 10 2H19C20.6569 2 22 3.34315 22 5V14C22 15.6569 20.6569 17 19 17H17V19C17 20.6569 15.6569 22 14 22H5C3.34315 22 2 20.6569 2 19V10C2 8.34315 3.34315 7 5 7H7V5ZM9 7H14C15.6569 7 17 8.34315 17 10V15H19C19.5523 15 20 14.5523 20 14V5C20 4.44772 19.5523 4 19 4H10C9.44772 4 9 4.44772 9 5V7ZM5 9C4.44772 9 4 9.44772 4 10V19C4 19.5523 4.44772 20 5 20H14C14.5523 20 15 19.5523 15 19V10C15 9.44772 14.5523 9 14 9H5Z" fill="currentColor"></path></svg>Copy code</button></span></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="!whitespace-pre hljs language-bash">!pip install -q langgraph langsmith langchain langchain_groq langchain_community
</code></div></div></pre>

## Environment Setup

You'll need to set up environment variables for your API keys:

```bash
from google.colab import userdata  # for Google Colab, replace as needed for local setup

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "LearningLanggraph"
os.environ["GROQ_API_KEY"] = userdata.get('GROK_LLAMA3')  # replace with actual key retrieval
os.environ["LANGCHAIN_API_KEY"] = userdata.get('Langsmith_api_key')  # replace with actual key retrieval
```




## Code Walkthrough

### 1. Initialize the Groq Language Model

The `ChatGroq` model is used here, specifying the `gemma2-9b-It` model:

```from
llm = ChatGroq(model_name="gemma2-9b-It")
```

### 2. Set Up the LangGraph `StateGraph`

We use `StateGraph` to manage chatbot state transitions, where each node represents a function or conversational step.

#### Define the State

The state contains the `messages` list, holding the chat history.

```from
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]
```


#### Create the Graph

Initialize the `StateGraph` and define the chatbot function:

```graph_builder
def chatbot(state: State):
    return {"messages": llm.invoke(state['messages'])}

graph_builder.add_node(chatbot)
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)

graph = graph_builder.compile()
```


### 3. Visualize the LangGraph

To understand the flow of this chatbot’s architecture, you can visualize the graph using `Mermaid` (for Colab users, an inline display):

```from
try:
    display(Image(graph.get_graph().draw_mermaid_png()))
except Exception:
    pass
```

The visualization will show nodes connected from `START` to `chatbot` to `END`, representing the sequential flow of the chatbot process.

---

## LangGraph Overview

### Key Concepts in LangGraph

* **StateGraph** : Manages nodes and edges representing conversational steps.
* **Nodes** : Each node can contain functions handling specific tasks.
* **Edges** : Define directional flow, enabling structured conversation paths.
* **START and END** : Default entry and exit points for graph-based flows.

### Chatbot Node

The main node, `chatbot`, processes incoming messages by invoking the language model (`llm`). It updates the `messages` list in the state, ensuring the chatbot maintains context throughout the session.

## Running the Chatbot

Launch the chatbot in a loop that allows user inputs and produces outputs based on the graph’s configuration:

```while
    user_input = input("User: ")
    if user_input.lower() in ["quit", "q"]:
        print("Goodbye")
        break
    for event in graph.stream({"messages": ("user", user_input)}):
        for value in event.values():
            print("Assistant:", value["messages"].content)
```

In this loop:

1. The user provides an input.
2. The chatbot processes the input using the `ChatGroq` model.
3. The chatbot responds based on the LangGraph flow, utilizing previous inputs and context stored in `messages`.

## Graph Visualization

LangGraph provides tools to visualize the conversational flow, which can help in debugging and understanding the chatbot’s structure.

To visualize, use:

```
from IPython.display import Image, display 
try:
    display(Image(graph.get_graph().draw_mermaid_png()))
except Exception:
    print("Graph visualization is not supported in this environment.")


```

This generates a flowchart from `START` -> `chatbot` -> `END`, giving insight into each node and transition.
