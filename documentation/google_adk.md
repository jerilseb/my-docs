# Installing ADK

## Create & activate virtual environment

We recommend creating a virtual Python environment using venv :

```
python -m venv .venv
source .venv/bin/activate
```

### Install ADK

```
pip install google-adk
```


## Next steps

## 1. Create Agent Project

### Project structure

You will need to create the following project structure:

```
parent_folder/
    multi_tool_agent/
        __init__.py
        agent.py
        .env
```

Create the folder `multi_tool_agent` :

```
mkdir multi_tool_agent/
```

### __init__.py

Now create an `__init__.py` file in the folder:

```
echo "from . import agent" > multi_tool_agent/__init__.py
```

Your `__init__.py` should now look like this:

multi_tool_agent/__init__.py
```
from . import agent
```

### agent.py

Create an `agent.py` file in the same folder:

```
touch multi_tool_agent/agent.py
```

Copy and paste the following code into `agent.py` :

multi_tool_agent/agent.py
```
import datetime
from zoneinfo import ZoneInfo
from google.adk.agents import Agent

def get_weather(city: str) -> dict:
    """Retrieves the current weather report for a specified city.

    Args:
        city (str): The name of the city for which to retrieve the weather report.

    Returns:
        dict: status and result or error msg.
    """
    if city.lower() == "new york":
        return {
            "status": "success",
            "report": (
                "The weather in New York is sunny with a temperature of 25 degrees"
                " Celsius (77 degrees Fahrenheit)."
            ),
        }
    else:
        return {
            "status": "error",
            "error_message": f"Weather information for '{city}' is not available.",
        }

def get_current_time(city: str) -> dict:
    """Returns the current time in a specified city.

    Args:
        city (str): The name of the city for which to retrieve the current time.

    Returns:
        dict: status and result or error msg.
    """

    if city.lower() == "new york":
        tz_identifier = "America/New_York"
    else:
        return {
            "status": "error",
            "error_message": (
                f"Sorry, I don't have timezone information for {city}."
            ),
        }

    tz = ZoneInfo(tz_identifier)
    now = datetime.datetime.now(tz)
    report = (
        f'The current time in {city} is {now.strftime("%Y-%m-%d %H:%M:%S %Z%z")}'
    )
    return {"status": "success", "report": report}

root_agent = Agent(
    name="weather_time_agent",
    model="gemini-2.5-flash",
    description=(
        "Agent to answer questions about the time and weather in a city."
    ),
    instruction=(
        "You are a helpful agent who can answer user questions about the time and weather in a city."
    ),
    tools=[get_weather, get_current_time],
)
```

### .env

Create a `.env` file in the same folder:

```
touch multi_tool_agent/.env
```

More instructions about this file are described in the next section on Set up the model .

## 2. Set up the model

Your agent's ability to understand user requests and generate responses is
powered by a Large Language Model (LLM). Your agent needs to make secure calls
to this external LLM service, which **requires authentication credentials** . Without
valid authentication, the LLM service will deny the agent's requests, and the
agent will be unable to function.

Model Authentication guide

For a detailed guide on authenticating to different models, see the Authentication guide .
This is a critical step to ensure your agent can make calls to the LLM service.

1. Get an API key from Google AI Studio .
2. When using Python, open the **`.env`** file located inside ( `multi_tool_agent/` )
and copy-paste the following code.

multi_tool_agent/.env
```
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_API_KEY_HERE
```

When using Java, define environment variables:

terminal
```
export GOOGLE_GENAI_USE_VERTEXAI=FALSE
export GOOGLE_API_KEY=PASTE_YOUR_ACTUAL_API_KEY_HERE
```

3. Replace `PASTE_YOUR_ACTUAL_API_KEY_HERE` with your actual `API KEY` .

## 3. Run Your Agent

Using the terminal, navigate to the parent directory of your agent project
(e.g. using `cd ..` ):

```
parent_folder/      <-- navigate to this directory
    multi_tool_agent/
        __init__.py
        agent.py
        .env
```

There are multiple ways to interact with your agent:
Run the following command to launch the **dev UI** .

```
adk web
```

Note for Windows users

When hitting the `_make_subprocess_transport NotImplementedError` , consider using `adk web --no-reload` instead.

**Step 1:** Open the URL provided (usually `http://localhost:8000` or `http://127.0.0.1:8000` ) directly in your browser.

**Step 2.** In the top-left corner of the UI, you can select your agent in
the dropdown. Select "multi_tool_agent".

Troubleshooting

If you do not see "multi_tool_agent" in the dropdown menu, make sure you
are running `adk web` in the **parent folder** of your agent folder
(i.e. the parent folder of multi_tool_agent).

**Step 3.** Now you can chat with your agent using the textbox:

**Step 4.** By using the `Events` tab at the left, you can inspect
individual function calls, responses and model responses by clicking on the
actions:

On the `Events` tab, you can also click the `Trace` button to see the trace logs for each event that shows the latency of each function calls:

**Step 5.** You can also enable your microphone and talk to your agent:

Model support for voice/video streaming

In order to use voice/video streaming in ADK, you will need to use Gemini models that support the Live API. You can find the **model ID(s)** that supports the Gemini Live API in the documentation:

- Google AI Studio: Gemini Live API
- Vertex AI: Gemini Live API

You can then replace the `model` string in `root_agent` in the `agent.py` file you created earlier ( jump to section ). Your code should look something like:

```
root_agent = Agent(
    name="weather_time_agent",
    model="replace-me-with-model-id", #e.g. gemini-2.5-flash-live-001
    ...
```

### 📝 Example prompts to try

- What is the weather in New York?
- What is the time in New York?
- What is the weather in Paris?
- What is the time in Paris?

-----------------

# Agents

In the Agent Development Kit (ADK), an **Agent** is a self-contained execution unit designed to act autonomously to achieve specific goals. Agents can perform tasks, interact with users, utilize external tools, and coordinate with other agents.

The foundation for all agents in ADK is the `BaseAgent` class. It serves as the fundamental blueprint. To create functional agents, you typically extend `BaseAgent` in one of three main ways, catering to different needs – from intelligent reasoning to structured process control.

## Core Agent Categories

ADK provides distinct agent categories to build sophisticated applications:

1. **LLM Agents ( `LlmAgent` , `Agent` )** : These agents utilize Large Language Models (LLMs) as their core engine to understand natural language, reason, plan, generate responses, and dynamically decide how to proceed or which tools to use, making them ideal for flexible, language-centric tasks. Learn more about LLM Agents...
2. **Workflow Agents ( `SequentialAgent` , `ParallelAgent` , `LoopAgent` )** : These specialized agents control the execution flow of other agents in predefined, deterministic patterns (sequence, parallel, or loop) without using an LLM for the flow control itself, perfect for structured processes needing predictable execution. Explore Workflow Agents...
3. **Custom Agents** : Created by extending `BaseAgent` directly, these agents allow you to implement unique operational logic, specific control flows, or specialized integrations not covered by the standard types, catering to highly tailored application requirements. Discover how to build Custom Agents...

## Choosing the Right Agent Type

The following table provides a high-level comparison to help distinguish between the agent types. As you explore each type in more detail in the subsequent sections, these distinctions will become clearer.

| Feature | LLM Agent (LlmAgent) | Workflow Agent | Custom Agent (BaseAgent subclass) |
| --- | --- | --- | --- |
| Primary Function | Reasoning, Generation, Tool Use | Controlling Agent Execution Flow | Implementing Unique Logic/Integrations |
| Core Engine | Large Language Model (LLM) | Predefined Logic (Sequence, Parallel, Loop) | Custom Code |
| Determinism | Non-deterministic (Flexible) | Deterministic (Predictable) | Can be either, based on implementation |
| Primary Use | Language tasks, Dynamic decisions | Structured processes, Orchestration | Tailored requirements, Specific workflows |

## Agents Working Together: Multi-Agent Systems

While each agent type serves a distinct purpose, the true power often comes from combining them. Complex applications frequently employ multi-agent architectures where:

- **LLM Agents** handle intelligent, language-based task execution.
- **Workflow Agents** manage the overall process flow using standard patterns.
- **Custom Agents** provide specialized capabilities or rules needed for unique integrations.

Understanding these core types is the first step toward building sophisticated, capable AI applications with ADK.


-----------------

# LLM Agent

The `LlmAgent` (often aliased simply as `Agent` ) is a core component in ADK,
acting as the "thinking" part of your application. It leverages the power of a
Large Language Model (LLM) for reasoning, understanding natural language, making
decisions, generating responses, and interacting with tools.

Unlike deterministic Workflow Agents that follow
predefined execution paths, `LlmAgent` behavior is non-deterministic. It uses
the LLM to interpret instructions and context, deciding dynamically how to
proceed, which tools to use (if any), or whether to transfer control to another
agent.

Building an effective `LlmAgent` involves defining its identity, clearly guiding
its behavior through instructions, and equipping it with the necessary tools and
capabilities.

## Defining the Agent's Identity and Purpose

First, you need to establish what the agent *is* and what it's *for* .

- **`name` (Required):** Every agent needs a unique string identifier. This `name` is crucial for internal operations, especially in multi-agent systems
  where agents need to refer to or delegate tasks to each other. Choose a
  descriptive name that reflects the agent's function (e.g., `customer_support_router` , `billing_inquiry_agent` ). Avoid reserved names like `user` .

- **`description` (Optional, Recommended for Multi-Agent):** Provide a concise
  summary of the agent's capabilities. This description is primarily used by *other* LLM agents to determine if they should route a task to this agent.
  Make it specific enough to differentiate it from peers (e.g., "Handles
  inquiries about current billing statements," not just "Billing agent").

- **`model` (Required):** Specify the underlying LLM that will power this
  agent's reasoning. This is a string identifier like `"gemini-2.5-flash"` . The
  choice of model impacts the agent's capabilities, cost, and performance. See
  the Models page for available options and considerations.

```
# Example: Defining the basic identity
capital_agent = LlmAgent(
    model="gemini-2.5-flash",
    name="capital_agent",
    description="Answers user questions about the capital city of a given country."
    # instruction and tools will be added next
)
```

## Guiding the Agent: Instructions (instruction)

The `instruction` parameter is arguably the most critical for shaping an `LlmAgent` 's behavior. It's a string (or a function returning a string) that
tells the agent:

- Its core task or goal.
- Its personality or persona (e.g., "You are a helpful assistant," "You are a witty pirate").
- Constraints on its behavior (e.g., "Only answer questions about X," "Never reveal Y").
- How and when to use its `tools` . You should explain the purpose of each tool and the circumstances under which it should be called, supplementing any descriptions within the tool itself.
- The desired format for its output (e.g., "Respond in JSON," "Provide a bulleted list").

**Tips for Effective Instructions:**

- **Be Clear and Specific:** Avoid ambiguity. Clearly state the desired actions and outcomes.
- **Use Markdown:** Improve readability for complex instructions using headings, lists, etc.
- **Provide Examples (Few-Shot):** For complex tasks or specific output formats, include examples directly in the instruction.
- **Guide Tool Use:** Don't just list tools; explain *when* and *why* the agent should use them.

**State:**

- The instruction is a string template, you can use the `{var}` syntax to insert dynamic values into the instruction.
- `{var}` is used to insert the value of the state variable named var.
- `{artifact.var}` is used to insert the text content of the artifact named var.
- If the state variable or artifact does not exist, the agent will raise an error. If you want to ignore the error, you can append a `?` to the variable name as in `{var?}` .

```
# Example: Adding instructions
capital_agent = LlmAgent(
    model="gemini-2.5-flash",
    name="capital_agent",
    description="Answers user questions about the capital city of a given country.",
    instruction="""You are an agent that provides the capital city of a country.
When a user asks for the capital of a country:
1. Identify the country name from the user's query.
2. Use the `get_capital_city` tool to find the capital.
3. Respond clearly to the user, stating the capital city.
Example Query: "What's the capital of {country}?"
Example Response: "The capital of France is Paris."
""",
    # tools will be added next
)
```

*(Note: For instructions that apply to* all *agents in a system, consider using `global_instruction` on the root agent, detailed further in the Multi-Agents section.)*

## Equipping the Agent: Tools (tools)

Tools give your `LlmAgent` capabilities beyond the LLM's built-in knowledge or
reasoning. They allow the agent to interact with the outside world, perform
calculations, fetch real-time data, or execute specific actions.

- **`tools` (Optional):** Provide a list of tools the agent can use. Each item in the list can be:
- A native function or method (wrapped as a `FunctionTool` ). Python ADK automatically wraps the native function into a `FuntionTool` whereas, you must explicitly wrap your Java methods using `FunctionTool.create(...)`
- An instance of a class inheriting from `BaseTool` .
- An instance of another agent ( `AgentTool` , enabling agent-to-agent delegation - see Multi-Agents ).

The LLM uses the function/tool names, descriptions (from docstrings or the `description` field), and parameter schemas to decide which tool to call based
on the conversation and its instructions.

```
# Define a tool function
def get_capital_city(country: str) -> str:
  """Retrieves the capital city for a given country."""
  # Replace with actual logic (e.g., API call, database lookup)
  capitals = {"france": "Paris", "japan": "Tokyo", "canada": "Ottawa"}
  return capitals.get(country.lower(), f"Sorry, I don't know the capital of {country}.")

# Add the tool to the agent
capital_agent = LlmAgent(
    model="gemini-2.5-flash",
    name="capital_agent",
    description="Answers user questions about the capital city of a given country.",
    instruction="""You are an agent that provides the capital city of a country... (previous instruction text)""",
    tools=[get_capital_city] # Provide the function directly
)
```

Learn more about Tools in the Tools section.

## Advanced Configuration & Control

Beyond the core parameters, `LlmAgent` offers several options for finer control:

### Configuring LLM Generation (generate_content_config)

You can adjust how the underlying LLM generates responses using `generate_content_config` .

- **`generate_content_config` (Optional):** Pass an instance of `google.genai.types.GenerateContentConfig` to control parameters like `temperature` (randomness), `max_output_tokens` (response length), `top_p` , `top_k` , and safety settings.

```
from google.genai import types

agent = LlmAgent(
    # ... other params
    generate_content_config=types.GenerateContentConfig(
        temperature=0.2, # More deterministic output
        max_output_tokens=,
        safety_settings=[
            types.SafetySetting(
                category=types.HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
                threshold=types.HarmBlockThreshold.BLOCK_LOW_AND_ABOVE,
            )
        ]
    )
)
```

### Structuring Data (input_schema, output_schema, output_key)

For scenarios requiring structured data exchange with an `LLM Agent` , the ADK provides mechanisms to define expected input and desired output formats using schema definitions.

- **`input_schema` (Optional):** Define a schema representing the expected input structure. If set, the user message content passed to this agent *must* be a JSON string conforming to this schema. Your instructions should guide the user or preceding agent accordingly.
- **`output_schema` (Optional):** Define a schema representing the desired output structure. If set, the agent's final response *must* be a JSON string conforming to this schema.
- **`output_key` (Optional):** Provide a string key. If set, the text content of the agent's *final* response will be automatically saved to the session's state dictionary under this key. This is useful for passing results between agents or steps in a workflow.

- In Python, this might look like: `session.state[output_key] = agent_response_text`
- In Java: `session.state().put(outputKey, agentResponseText)`

The input and output schema is typically a `Pydantic` BaseModel.

```
from pydantic import BaseModel, Field

class CapitalOutput(BaseModel):
    capital: str = Field(description="The capital of the country.")

structured_capital_agent = LlmAgent(
    # ... name, model, description
    instruction="""You are a Capital Information Agent. Given a country, respond ONLY with a JSON object containing the capital. Format: {"capital": "capital_name"}""",
    output_schema=CapitalOutput, # Enforce JSON output
    output_key="found_capital"  # Store result in state['found_capital']
    # Cannot use tools=[get_capital_city] effectively here
)
```

### Managing Context (include_contents)

Control whether the agent receives the prior conversation history.

- **`include_contents` (Optional, Default: `'default'` ):** Determines if the `contents` (history) are sent to the LLM.
- `'default'` : The agent receives the relevant conversation history.
- `'none'` : The agent receives no prior `contents` . It operates based solely on its current instruction and any input provided in the *current* turn (useful for stateless tasks or enforcing specific contexts).

```
stateless_agent = LlmAgent(
    # ... other params
    include_contents='none'
)
```

### Planner

**`planner` (Optional):** Assign a `BasePlanner` instance to enable multi-step reasoning and planning before execution. There are two main planners:

- **`BuiltInPlanner` :** Leverages the model's built-in planning capabilities (e.g., Gemini's thinking feature). See Gemini Thinking for details and examples.

Here, the `thinking_budget` parameter guides the model on the number of thinking tokens to use when generating a response. The `include_thoughts` parameter controls whether the model should include its raw thoughts and internal reasoning process in the response.

```
from google.adk import Agent
from google.adk.planners import BuiltInPlanner
from google.genai import types

my_agent = Agent(
    model="gemini-2.5-flash",
    planner=BuiltInPlanner(
        thinking_config=types.ThinkingConfig(
            include_thoughts=True,
            thinking_budget=,
        )
    ),
    # ... your tools here
)
```

- **`PlanReActPlanner` :** This planner instructs the model to follow a specific structure in its output: first create a plan, then execute actions (like calling tools), and provide reasoning for its steps. *It's particularly useful for models that don't have a built-in "thinking" feature* .

```
from google.adk import Agent
from google.adk.planners import PlanReActPlanner

my_agent = Agent(
    model="gemini-2.5-flash",
    planner=PlanReActPlanner(),
    # ... your tools here
)
```

The agent's response will follow a structured format:

```
[user]: ai news
[google_search_agent]: /*PLANNING*/
1. Perform a Google search for "latest AI news" to get current updates and headlines related to artificial intelligence.
2. Synthesize the information from the search results to provide a summary of recent AI news.

/*ACTION*/
/*REASONING*/
The search results provide a comprehensive overview of recent AI news, covering various aspects like company developments, research breakthroughs, and applications. I have enough information to answer the user's request.

/*FINAL_ANSWER*/
Here's a summary of recent AI news:
....
```

### Code Execution

- **`code_executor` (Optional):** Provide a `BaseCodeExecutor` instance to allow the agent to execute code blocks found in the LLM's response. ( See Tools/Built-in tools ).

Example for using built-in-planner:

```
from dotenv import load_dotenv

import asyncio
import os

from google.genai import types
from google.adk.agents.llm_agent import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.artifacts.in_memory_artifact_service import InMemoryArtifactService # Optional
from google.adk.planners import BasePlanner, BuiltInPlanner, PlanReActPlanner
from google.adk.models import LlmRequest

from google.genai.types import ThinkingConfig
from google.genai.types import GenerateContentConfig

import datetime
from zoneinfo import ZoneInfo

APP_NAME = "weather_app"
USER_ID = "1234"
SESSION_ID = "session1234"

def get_weather(city: str) -> dict:
    """Retrieves the current weather report for a specified city.

    Args:
        city (str): The name of the city for which to retrieve the weather report.

    Returns:
        dict: status and result or error msg.
    """
    if city.lower() == "new york":
        return {
            "status": "success",
            "report": (
                "The weather in New York is sunny with a temperature of 25 degrees"
                " Celsius (77 degrees Fahrenheit)."
            ),
        }
    else:
        return {
            "status": "error",
            "error_message": f"Weather information for '{city}' is not available.",
        }

def get_current_time(city: str) -> dict:
    """Returns the current time in a specified city.

    Args:
        city (str): The name of the city for which to retrieve the current time.

    Returns:
        dict: status and result or error msg.
    """

    if city.lower() == "new york":
        tz_identifier = "America/New_York"
    else:
        return {
            "status": "error",
            "error_message": (
                f"Sorry, I don't have timezone information for {city}."
            ),
        }

    tz = ZoneInfo(tz_identifier)
    now = datetime.datetime.now(tz)
    report = (
        f'The current time in {city} is {now.strftime("%Y-%m-%d %H:%M:%S %Z%z")}'
    )
    return {"status": "success", "report": report}

# Step 1: Create a ThinkingConfig
thinking_config = ThinkingConfig(
    include_thoughts=True,   # Ask the model to include its thoughts in the response
    thinking_budget=      # Limit the 'thinking' to 256 tokens (adjust as needed)
)
print("ThinkingConfig:", thinking_config)

# Step 2: Instantiate BuiltInPlanner
planner = BuiltInPlanner(
    thinking_config=thinking_config
)
print("BuiltInPlanner created.")

# Step 3: Wrap the planner in an LlmAgent
agent = LlmAgent(
    model="gemini-2.5-pro-preview-03-25",  # Set your model name
    name="weather_and_time_agent",
    instruction="You are an agent that returns time and weather",
    planner=planner,
    tools=[get_weather, get_current_time]
)

# Session and Runner
session_service = InMemorySessionService()
session = session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
runner = Runner(agent=agent, app_name=APP_NAME, session_service=session_service)

# Agent Interaction
def call_agent(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    events = runner.run(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    for event in events:
        print(f"\nDEBUG EVENT: {event}\n")
        if event.is_final_response() and event.content:
            final_answer = event.content.parts[0].text.strip()
            print("\n🟢 FINAL ANSWER\n", final_answer, "\n")

call_agent("If it's raining in New York right now, what is the current temperature?")
```

## Putting It Together: Example

Code
Here's the complete basic `capital_agent` :

```

# --- Full example code demonstrating LlmAgent with Tools vs. Output Schema ---
import json # Needed for pretty printing dicts
import asyncio

from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types
from pydantic import BaseModel, Field

# --- 1. Define Constants ---
APP_NAME = "agent_comparison_app"
USER_ID = "test_user_456"
SESSION_ID_TOOL_AGENT = "session_tool_agent_xyz"
SESSION_ID_SCHEMA_AGENT = "session_schema_agent_xyz"
MODEL_NAME = "gemini-2.5-flash"

# --- 2. Define Schemas ---

# Input schema used by both agents
class CountryInput(BaseModel):
    country: str = Field(description="The country to get information about.")

# Output schema ONLY for the second agent
class CapitalInfoOutput(BaseModel):
    capital: str = Field(description="The capital city of the country.")
    # Note: Population is illustrative; the LLM will infer or estimate this
    # as it cannot use tools when output_schema is set.
    population_estimate: str = Field(description="An estimated population of the capital city.")

# --- 3. Define the Tool (Only for the first agent) ---
def get_capital_city(country: str) -> str:
    """Retrieves the capital city of a given country."""
    print(f"\n-- Tool Call: get_capital_city(country='{country}') --")
    country_capitals = {
        "united states": "Washington, D.C.",
        "canada": "Ottawa",
        "france": "Paris",
        "japan": "Tokyo",
    }
    result = country_capitals.get(country.lower(), f"Sorry, I couldn't find the capital for {country}.")
    print(f"-- Tool Result: '{result}' --")
    return result

# --- 4. Configure Agents ---

# Agent 1: Uses a tool and output_key
capital_agent_with_tool = LlmAgent(
    model=MODEL_NAME,
    name="capital_agent_tool",
    description="Retrieves the capital city using a specific tool.",
    instruction="""You are a helpful agent that provides the capital city of a country using a tool.
The user will provide the country name in a JSON format like {"country": "country_name"}.
1. Extract the country name.
2. Use the `get_capital_city` tool to find the capital.
3. Respond clearly to the user, stating the capital city found by the tool.
""",
    tools=[get_capital_city],
    input_schema=CountryInput,
    output_key="capital_tool_result", # Store final text response
)

# Agent 2: Uses output_schema (NO tools possible)
structured_info_agent_schema = LlmAgent(
    model=MODEL_NAME,
    name="structured_info_agent_schema",
    description="Provides capital and estimated population in a specific JSON format.",
    instruction=f"""You are an agent that provides country information.
The user will provide the country name in a JSON format like {{"country": "country_name"}}.
Respond ONLY with a JSON object matching this exact schema:
{json.dumps(CapitalInfoOutput.model_json_schema(), indent=)}
Use your knowledge to determine the capital and estimate the population. Do not use any tools.
""",
    # *** NO tools parameter here - using output_schema prevents tool use ***
    input_schema=CountryInput,
    output_schema=CapitalInfoOutput, # Enforce JSON output structure
    output_key="structured_info_result", # Store final JSON response
)

# --- 5. Set up Session Management and Runners ---
session_service = InMemorySessionService()

# Create a runner for EACH agent
capital_runner = Runner(
    agent=capital_agent_with_tool,
    app_name=APP_NAME,
    session_service=session_service
)
structured_runner = Runner(
    agent=structured_info_agent_schema,
    app_name=APP_NAME,
    session_service=session_service
)

# --- 6. Define Agent Interaction Logic ---
async def call_agent_and_print(
    runner_instance: Runner,
    agent_instance: LlmAgent,
    session_id: str,
    query_json: str
):
    """Sends a query to the specified agent/runner and prints results."""
    print(f"\n>>> Calling Agent: '{agent_instance.name}' | Query: {query_json}")

    user_content = types.Content(role='user', parts=[types.Part(text=query_json)])

    final_response_content = "No final response received."
    async for event in runner_instance.run_async(user_id=USER_ID, session_id=session_id, new_message=user_content):
        # print(f"Event: {event.type}, Author: {event.author}") # Uncomment for detailed logging
        if event.is_final_response() and event.content and event.content.parts:
            # For output_schema, the content is the JSON string itself
            final_response_content = event.content.parts[0].text

    print(f"<<< Agent '{agent_instance.name}' Response: {final_response_content}")

    current_session = await session_service.get_session(app_name=APP_NAME,
                                                  user_id=USER_ID,
                                                  session_id=session_id)
    stored_output = current_session.state.get(agent_instance.output_key)

    # Pretty print if the stored output looks like JSON (likely from output_schema)
    print(f"--- Session State ['{agent_instance.output_key}']: ", end="")
    try:
        # Attempt to parse and pretty print if it's JSON
        parsed_output = json.loads(stored_output)
        print(json.dumps(parsed_output, indent=2))
    except (json.JSONDecodeError, TypeError):
         # Otherwise, print as string
        print(stored_output)
    print("-" * 30)

# --- 7. Run Interactions ---
async def main():
    # Create separate sessions for clarity, though not strictly necessary if context is managed
    print("--- Creating Sessions ---")
    await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID_TOOL_AGENT)
    await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID_SCHEMA_AGENT)

    print("--- Testing Agent with Tool ---")
    await call_agent_and_print(capital_runner, capital_agent_with_tool, SESSION_ID_TOOL_AGENT, '{"country": "France"}')
    await call_agent_and_print(capital_runner, capital_agent_with_tool, SESSION_ID_TOOL_AGENT, '{"country": "Canada"}')

    print("\n\n--- Testing Agent with Output Schema (No Tool Use) ---")
    await call_agent_and_print(structured_runner, structured_info_agent_schema, SESSION_ID_SCHEMA_AGENT, '{"country": "France"}')
    await call_agent_and_print(structured_runner, structured_info_agent_schema, SESSION_ID_SCHEMA_AGENT, '{"country": "Japan"}')

# --- Run the Agent ---
# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
if __name__ == "__main__":
    asyncio.run(main())
```

-----------------

# Workflow Agents

This section introduces " *workflow agents* " - **specialized agents that control the execution flow of its sub-agents** .

Workflow agents are specialized components in ADK designed purely for **orchestrating the execution flow of sub-agents** . Their primary role is to manage *how* and *when* other agents run, defining the control flow of a process.

Unlike LLM Agents , which use Large Language Models for dynamic reasoning and decision-making, Workflow Agents operate based on **predefined logic** . They determine the execution sequence according to their type (e.g., sequential, parallel, loop) without consulting an LLM for the orchestration itself. This results in **deterministic and predictable execution patterns** .

ADK provides three core workflow agent types, each implementing a distinct execution pattern:

- **Sequential Agents**

---

Executes sub-agents one after another, in **sequence** .

Learn more

- **Loop Agents**

---

**Repeatedly** executes its sub-agents until a specific termination condition is met.

Learn more

- **Parallel Agents**

---

Executes multiple sub-agents in **parallel** .

Learn more

## Why Use Workflow Agents?

Workflow agents are essential when you need explicit control over how a series of tasks or agents are executed. They provide:

- **Predictability:** The flow of execution is guaranteed based on the agent type and configuration.
- **Reliability:** Ensures tasks run in the required order or pattern consistently.
- **Structure:** Allows you to build complex processes by composing agents within clear control structures.

While the workflow agent manages the control flow deterministically, the sub-agents it orchestrates can themselves be any type of agent, including intelligent LLM Agent instances. This allows you to combine structured process control with flexible, LLM-powered task execution.

-----------------

# Sequential agents

## The SequentialAgent

The `SequentialAgent` is a workflow agent that executes its sub-agents in the order they are specified in the list.

Use the `SequentialAgent` when you want the execution to occur in a fixed, strict order.

### Example

- You want to build an agent that can summarize any webpage, using two tools: `Get Page Contents` and `Summarize Page` . Because the agent must always call `Get Page Contents` before calling `Summarize Page` (you can't summarize from nothing!), you should build your agent using a `SequentialAgent` .

As with other workflow agents , the `SequentialAgent` is not powered by an LLM, and is thus deterministic in how it executes. That being said, workflow agents are concerned only with their execution (i.e. in sequence), and not their internal logic; the tools or sub-agents of a workflow agent may or may not utilize LLMs.

### How it works

When the `SequentialAgent` 's `Run Async` method is called, it performs the following actions:

1. **Iteration:** It iterates through the sub agents list in the order they were provided.
2. **Sub-Agent Execution:** For each sub-agent in the list, it calls the sub-agent's `Run Async` method.

### Full Example: Code Development Pipeline

Consider a simplified code development pipeline:

- **Code Writer Agent:** An LLM Agent that generates initial code based on a specification.
- **Code Reviewer Agent:** An LLM Agent that reviews the generated code for errors, style issues, and adherence to best practices.  It receives the output of the Code Writer Agent.
- **Code Refactorer Agent:** An LLM Agent that takes the reviewed code (and the reviewer's comments) and refactors it to improve quality and address issues.

A `SequentialAgent` is perfect for this:

```
SequentialAgent(sub_agents=[CodeWriterAgent, CodeReviewerAgent, CodeRefactorerAgent])
```

This ensures the code is written, *then* reviewed, and *finally* refactored, in a strict, dependable order. **The output from each sub-agent is passed to the next by storing them in state via Output Key** .

Shared Invocation Context

The `SequentialAgent` passes the same `InvocationContext` to each of its sub-agents. This means they all share the same session state, including the temporary ( `temp:` ) namespace, making it easy to pass data between steps within a single turn.

Code
```
# Part of agent.py --> Follow https://google.github.io/adk-docs/get-started/quickstart/ to learn the setup

# --- 1. Define Sub-Agents for Each Pipeline Stage ---

# Code Writer Agent

# Takes the initial specification (from user query) and writes code.

code_writer_agent = LlmAgent(

    name="CodeWriterAgent",

    model=GEMINI_MODEL,

    # Change 3: Improved instruction

    instruction="""You are a Python Code Generator.

Based *only* on the user's request, write Python code that fulfills the requirement.

Output *only* the complete Python code block, enclosed in triple backticks (
```python ...
```).

Do not add any other text before or after the code block.

""",

    description="Writes initial Python code based on a specification.",

    output_key="generated_code" # Stores output in state['generated_code']

)

# Code Reviewer Agent

# Takes the code generated by the previous agent (read from state) and provides feedback.

code_reviewer_agent = LlmAgent(

    name="CodeReviewerAgent",

    model=GEMINI_MODEL,

    # Change 3: Improved instruction, correctly using state key injection

    instruction="""You are an expert Python Code Reviewer.

    Your task is to provide constructive feedback on the provided code.

    **Code to Review:**


```python

    {generated_code}


```

**Review Criteria:**

1.  **Correctness:** Does the code work as intended? Are there logic errors?

2.  **Readability:** Is the code clear and easy to understand? Follows PEP 8 style guidelines?

3.  **Efficiency:** Is the code reasonably efficient? Any obvious performance bottlenecks?

4.  **Edge Cases:** Does the code handle potential edge cases or invalid inputs gracefully?

5.  **Best Practices:** Does the code follow common Python best practices?

**Output:**

Provide your feedback as a concise, bulleted list. Focus on the most important points for improvement.

If the code is excellent and requires no changes, simply state: "No major issues found."

Output *only* the review comments or the "No major issues" statement.

""",

    description="Reviews code and provides feedback.",

    output_key="review_comments", # Stores output in state['review_comments']

)

# Code Refactorer Agent

# Takes the original code and the review comments (read from state) and refactors the code.

code_refactorer_agent = LlmAgent(

    name="CodeRefactorerAgent",

    model=GEMINI_MODEL,

    # Change 3: Improved instruction, correctly using state key injection

    instruction="""You are a Python Code Refactoring AI.

Your goal is to improve the given Python code based on the provided review comments.

  **Original Code:**


```python

  {generated_code}


```

  **Review Comments:**

  {review_comments}

**Task:**

Carefully apply the suggestions from the review comments to refactor the original code.

If the review comments state "No major issues found," return the original code unchanged.

Ensure the final code is complete, functional, and includes necessary imports and docstrings.

**Output:**

Output *only* the final, refactored Python code block, enclosed in triple backticks (
```python ...
```).

Do not add any other text before or after the code block.

""",

    description="Refactors code based on review comments.",

    output_key="refactored_code", # Stores output in state['refactored_code']

)

# --- 2. Create the SequentialAgent ---

# This agent orchestrates the pipeline by running the sub_agents in order.

code_pipeline_agent = SequentialAgent(

    name="CodePipelineAgent",

    sub_agents=[code_writer_agent, code_reviewer_agent, code_refactorer_agent],

    description="Executes a sequence of code writing, reviewing, and refactoring.",

    # The agents will run in the order provided: Writer -> Reviewer -> Refactorer

)

# For ADK tools compatibility, the root agent must be named `root_agent`

root_agent = code_pipeline_agent
```

-----------------

# Loop agents

## The LoopAgent

The `LoopAgent` is a workflow agent that executes its sub-agents in a loop (i.e. iteratively). It ***repeatedly runs* a sequence of agents** for a specified number of iterations or until a termination condition is met.

Use the `LoopAgent` when your workflow involves repetition or iterative refinement, such as revising code.

### Example

- You want to build an agent that can generate images of food, but sometimes when you want to generate a specific number of items (e.g. 5 bananas), it generates a different number of those items in the image (e.g. an image of 7 bananas). You have two tools: `Generate Image` , `Count Food Items` . Because you want to keep generating images until it either correctly generates the specified number of items, or after a certain number of iterations, you should build your agent using a `LoopAgent` .

As with other workflow agents , the `LoopAgent` is not powered by an LLM, and is thus deterministic in how it executes. That being said, workflow agents are only concerned only with their execution (i.e. in a loop), and not their internal logic; the tools or sub-agents of a workflow agent may or may not utilize LLMs.

### How it Works

When the `LoopAgent` 's `Run Async` method is called, it performs the following actions:

1. **Sub-Agent Execution:** It iterates through the Sub Agents list *in order* . For *each* sub-agent, it calls the agent's `Run Async` method.
2. **Termination Check:**

*Crucially* , the `LoopAgent` itself does *not* inherently decide when to stop looping. You *must* implement a termination mechanism to prevent infinite loops.  Common strategies include:

- **Max Iterations** : Set a maximum number of iterations in the `LoopAgent` . **The loop will terminate after that many iterations** .
- **Escalation from sub-agent** : Design one or more sub-agents to evaluate a condition (e.g., "Is the document quality good enough?", "Has a consensus been reached?").  If the condition is met, the sub-agent can signal termination (e.g., by raising a custom event, setting a flag in a shared context, or returning a specific value).

### Full Example: Iterative Document Improvement

Imagine a scenario where you want to iteratively improve a document:

- **Writer Agent:** An `LlmAgent` that generates or refines a draft on a topic.
- **Critic Agent:** An `LlmAgent` that critiques the draft, identifying areas for improvement.

```
LoopAgent(sub_agents=[WriterAgent, CriticAgent], max_iterations=)
```

In this setup, the `LoopAgent` would manage the iterative process.  The `CriticAgent` could be **designed to return a "STOP" signal when the document reaches a satisfactory quality level** , preventing further iterations. Alternatively, the `max iterations` parameter could be used to limit the process to a fixed number of cycles, or external logic could be implemented to make stop decisions. The **loop would run at most five times** , ensuring the iterative refinement doesn't continue indefinitely.

Full Code
```
# Part of agent.py --> Follow https://google.github.io/adk-docs/get-started/quickstart/ to learn the setup

import asyncio

import os

from google.adk.agents import LoopAgent, LlmAgent, BaseAgent, SequentialAgent

from google.genai import types

from google.adk.runners import InMemoryRunner

from google.adk.agents.invocation_context import InvocationContext

from google.adk.tools.tool_context import ToolContext

from typing import AsyncGenerator, Optional

from google.adk.events import Event, EventActions

# --- Constants ---

APP_NAME = "doc_writing_app_v3" # New App Name

USER_ID = "dev_user_01"

SESSION_ID_BASE = "loop_exit_tool_session" # New Base Session ID

GEMINI_MODEL = "gemini-2.5-flash"

STATE_INITIAL_TOPIC = "initial_topic"

# --- State Keys ---

STATE_CURRENT_DOC = "current_document"

STATE_CRITICISM = "criticism"

# Define the exact phrase the Critic should use to signal completion

COMPLETION_PHRASE = "No major issues found."

# --- Tool Definition ---

def exit_loop(tool_context: ToolContext):

  """Call this function ONLY when the critique indicates no further changes are needed, signaling the iterative process should end."""

  print(f"  [Tool Call] exit_loop triggered by {tool_context.agent_name}")

  tool_context.actions.escalate = True

  # Return empty dict as tools should typically return JSON-serializable output

  return {}

# --- Agent Definitions ---

# STEP 1: Initial Writer Agent (Runs ONCE at the beginning)

initial_writer_agent = LlmAgent(

    name="InitialWriterAgent",

    model=GEMINI_MODEL,

    include_contents='none',

    # MODIFIED Instruction: Ask for a slightly more developed start

    instruction=f"""You are a Creative Writing Assistant tasked with starting a story.

    Write the *first draft* of a short story (aim for 2-4 sentences).

    Base the content *only* on the topic provided below. Try to introduce a specific element (like a character, a setting detail, or a starting action) to make it engaging.

    Topic: {{initial_topic}}

    Output *only* the story/document text. Do not add introductions or explanations.

""",

    description="Writes the initial document draft based on the topic, aiming for some initial substance.",

    output_key=STATE_CURRENT_DOC

)

# STEP 2a: Critic Agent (Inside the Refinement Loop)

critic_agent_in_loop = LlmAgent(

    name="CriticAgent",

    model=GEMINI_MODEL,

    include_contents='none',

    # MODIFIED Instruction: More nuanced completion criteria, look for clear improvement paths.

    instruction=f"""You are a Constructive Critic AI reviewing a short document draft (typically 2-6 sentences). Your goal is balanced feedback.

    **Document to Review:**


```

    {{current_document}}


```

    **Task:**

    Review the document for clarity, engagement, and basic coherence according to the initial topic (if known).

    IF you identify 1-2 *clear and actionable* ways the document could be improved to better capture the topic or enhance reader engagement (e.g., "Needs a stronger opening sentence", "Clarify the character's goal"):

    Provide these specific suggestions concisely. Output *only* the critique text.

    ELSE IF the document is coherent, addresses the topic adequately for its length, and has no glaring errors or obvious omissions:

    Respond *exactly* with the phrase "{COMPLETION_PHRASE}" and nothing else. It doesn't need to be perfect, just functionally complete for this stage. Avoid suggesting purely subjective stylistic preferences if the core is sound.

    Do not add explanations. Output only the critique OR the exact completion phrase.

""",

    description="Reviews the current draft, providing critique if clear improvements are needed, otherwise signals completion.",

    output_key=STATE_CRITICISM

)

# STEP 2b: Refiner/Exiter Agent (Inside the Refinement Loop)

refiner_agent_in_loop = LlmAgent(

    name="RefinerAgent",

    model=GEMINI_MODEL,

    # Relies solely on state via placeholders

    include_contents='none',

    instruction=f"""You are a Creative Writing Assistant refining a document based on feedback OR exiting the process.

    **Current Document:**


```

    {{current_document}}


```

    **Critique/Suggestions:**

    {{criticism}}

    **Task:**

    Analyze the 'Critique/Suggestions'.

    IF the critique is *exactly* "{COMPLETION_PHRASE}":

    You MUST call the 'exit_loop' function. Do not output any text.

    ELSE (the critique contains actionable feedback):

    Carefully apply the suggestions to improve the 'Current Document'. Output *only* the refined document text.

    Do not add explanations. Either output the refined document OR call the exit_loop function.

""",

    description="Refines the document based on critique, or calls exit_loop if critique indicates completion.",

    tools=[exit_loop], # Provide the exit_loop tool

    output_key=STATE_CURRENT_DOC # Overwrites state['current_document'] with the refined version

)

# STEP 2: Refinement Loop Agent

refinement_loop = LoopAgent(

    name="RefinementLoop",

    # Agent order is crucial: Critique first, then Refine/Exit

    sub_agents=[

        critic_agent_in_loop,

        refiner_agent_in_loop,

    ],

    max_iterations= # Limit loops

)

# STEP 3: Overall Sequential Pipeline

# For ADK tools compatibility, the root agent must be named `root_agent`

root_agent = SequentialAgent(

    name="IterativeWritingPipeline",

    sub_agents=[

        initial_writer_agent, # Run first to create initial doc

        refinement_loop       # Then run the critique/refine loop

    ],

    description="Writes an initial document and then iteratively refines it with critique using an exit tool."

)
```

-----------------

# Parallel agents

The `ParallelAgent` is a workflow agent that executes its sub-agents *concurrently* . This dramatically speeds up workflows where tasks can be performed independently.

Use `ParallelAgent` when: For scenarios prioritizing speed and involving independent, resource-intensive tasks, a `ParallelAgent` facilitates efficient parallel execution. **When sub-agents operate without dependencies, their tasks can be performed concurrently** , significantly reducing overall processing time.

As with other workflow agents , the `ParallelAgent` is not powered by an LLM, and is thus deterministic in how it executes. That being said, workflow agents are only concerned with their execution (i.e. executing sub-agents in parallel), and not their internal logic; the tools or sub-agents of a workflow agent may or may not utilize LLMs.

### Example

This approach is particularly beneficial for operations like multi-source data retrieval or heavy computations, where parallelization yields substantial performance gains. Importantly, this strategy assumes no inherent need for shared state or direct information exchange between the concurrently executing agents.

### How it works

When the `ParallelAgent` 's `run_async()` method is called:

1. **Concurrent Execution:** It initiates the `run_async()` method of *each* sub-agent present in the `sub_agents` list *concurrently* .  This means all the agents start running at (approximately) the same time.
2. **Independent Branches:** Each sub-agent operates in its own execution branch.  There is ***no* automatic sharing of conversation history or state between these branches** during execution.
3. **Result Collection:** The `ParallelAgent` manages the parallel execution and, typically, provides a way to access the results from each sub-agent after they have completed (e.g., through a list of results or events). The order of results may not be deterministic.

### Independent Execution and State Management

It's *crucial* to understand that sub-agents within a `ParallelAgent` run independently.  If you *need* communication or data sharing between these agents, you must implement it explicitly.  Possible approaches include:

- **Shared `InvocationContext` :** You could pass a shared `InvocationContext` object to each sub-agent.  This object could act as a shared data store.  However, you'd need to manage concurrent access to this shared context carefully (e.g., using locks) to avoid race conditions.
- **External State Management:** Use an external database, message queue, or other mechanism to manage shared state and facilitate communication between agents.
- **Post-Processing:** Collect results from each branch, and then implement logic to coordinate data afterwards.

### Full Example: Parallel Web Research

Imagine researching multiple topics simultaneously:

1. **Researcher Agent 1:** An `LlmAgent` that researches "renewable energy sources."
2. **Researcher Agent 2:** An `LlmAgent` that researches "electric vehicle technology."
3. **Researcher Agent 3:** An `LlmAgent` that researches "carbon capture methods."

```
ParallelAgent(sub_agents=[ResearcherAgent1, ResearcherAgent2, ResearcherAgent3])
```

These research tasks are independent.  Using a `ParallelAgent` allows them to run concurrently, potentially reducing the total research time significantly compared to running them sequentially. The results from each agent would be collected separately after they finish.

Full Code
```
# Part of agent.py --> Follow https://google.github.io/adk-docs/get-started/quickstart/ to learn the setup

 # --- 1. Define Researcher Sub-Agents (to run in parallel) ---

 # Researcher 1: Renewable Energy

 researcher_agent_1 = LlmAgent(

     name="RenewableEnergyResearcher",

     model=GEMINI_MODEL,

     instruction="""You are an AI Research Assistant specializing in energy.

 Research the latest advancements in 'renewable energy sources'.

 Use the Google Search tool provided.

 Summarize your key findings concisely (1-2 sentences).

 Output *only* the summary.

 """,

     description="Researches renewable energy sources.",

     tools=[google_search],

     # Store result in state for the merger agent

     output_key="renewable_energy_result"

 )

 # Researcher 2: Electric Vehicles

 researcher_agent_2 = LlmAgent(

     name="EVResearcher",

     model=GEMINI_MODEL,

     instruction="""You are an AI Research Assistant specializing in transportation.

 Research the latest developments in 'electric vehicle technology'.

 Use the Google Search tool provided.

 Summarize your key findings concisely (1-2 sentences).

 Output *only* the summary.

 """,

     description="Researches electric vehicle technology.",

     tools=[google_search],

     # Store result in state for the merger agent

     output_key="ev_technology_result"

 )

 # Researcher 3: Carbon Capture

 researcher_agent_3 = LlmAgent(

     name="CarbonCaptureResearcher",

     model=GEMINI_MODEL,

     instruction="""You are an AI Research Assistant specializing in climate solutions.

 Research the current state of 'carbon capture methods'.

 Use the Google Search tool provided.

 Summarize your key findings concisely (1-2 sentences).

 Output *only* the summary.

 """,

     description="Researches carbon capture methods.",

     tools=[google_search],

     # Store result in state for the merger agent

     output_key="carbon_capture_result"

 )

 # --- 2. Create the ParallelAgent (Runs researchers concurrently) ---

 # This agent orchestrates the concurrent execution of the researchers.

 # It finishes once all researchers have completed and stored their results in state.

 parallel_research_agent = ParallelAgent(

     name="ParallelWebResearchAgent",

     sub_agents=[researcher_agent_1, researcher_agent_2, researcher_agent_3],

     description="Runs multiple research agents in parallel to gather information."

 )

 # --- 3. Define the Merger Agent (Runs *after* the parallel agents) ---

 # This agent takes the results stored in the session state by the parallel agents

 # and synthesizes them into a single, structured response with attributions.

 merger_agent = LlmAgent(

     name="SynthesisAgent",

     model=GEMINI_MODEL,  # Or potentially a more powerful model if needed for synthesis

     instruction="""You are an AI Assistant responsible for combining research findings into a structured report.

 Your primary task is to synthesize the following research summaries, clearly attributing findings to their source areas. Structure your response using headings for each topic. Ensure the report is coherent and integrates the key points smoothly.

 **Crucially: Your entire response MUST be grounded *exclusively* on the information provided in the 'Input Summaries' below. Do NOT add any external knowledge, facts, or details not present in these specific summaries.**

 **Input Summaries:**

 *   **Renewable Energy:**

     {renewable_energy_result}

 *   **Electric Vehicles:**

     {ev_technology_result}

 *   **Carbon Capture:**

     {carbon_capture_result}

 **Output Format:**

 ## Summary of Recent Sustainable Technology Advancements

 ### Renewable Energy Findings

 (Based on RenewableEnergyResearcher's findings)

 [Synthesize and elaborate *only* on the renewable energy input summary provided above.]

 ### Electric Vehicle Findings

 (Based on EVResearcher's findings)

 [Synthesize and elaborate *only* on the EV input summary provided above.]

 ### Carbon Capture Findings

 (Based on CarbonCaptureResearcher's findings)

 [Synthesize and elaborate *only* on the carbon capture input summary provided above.]

 ### Overall Conclusion

 [Provide a brief (1-2 sentence) concluding statement that connects *only* the findings presented above.]

 Output *only* the structured report following this format. Do not include introductory or concluding phrases outside this structure, and strictly adhere to using only the provided input summary content.

 """,

     description="Combines research findings from parallel agents into a structured, cited report, strictly grounded on provided inputs.",

     # No tools needed for merging

     # No output_key needed here, as its direct response is the final output of the sequence

 )

 # --- 4. Create the SequentialAgent (Orchestrates the overall flow) ---

 # This is the main agent that will be run. It first executes the ParallelAgent

 # to populate the state, and then executes the MergerAgent to produce the final output.

 sequential_pipeline_agent = SequentialAgent(

     name="ResearchAndSynthesisPipeline",

     # Run parallel research first, then merge

     sub_agents=[parallel_research_agent, merger_agent],

     description="Coordinates parallel research and synthesizes the results."

 )

 root_agent = sequential_pipeline_agent
```

-----------------

# Multi-Agent Systems in ADK

As agentic applications grow in complexity, structuring them as a single, monolithic agent can become challenging to develop, maintain, and reason about. The Agent Development Kit (ADK) supports building sophisticated applications by composing multiple, distinct `BaseAgent` instances into a **Multi-Agent System (MAS)** .

In ADK, a multi-agent system is an application where different agents, often forming a hierarchy, collaborate or coordinate to achieve a larger goal. Structuring your application this way offers significant advantages, including enhanced modularity, specialization, reusability, maintainability, and the ability to define structured control flows using dedicated workflow agents.

You can compose various types of agents derived from `BaseAgent` to build these systems:

- **LLM Agents:** Agents powered by large language models. (See LLM Agents )
- **Workflow Agents:** Specialized agents ( `SequentialAgent` , `ParallelAgent` , `LoopAgent` ) designed to manage the execution flow of their sub-agents. (See Workflow Agents )
- **Custom agents:** Your own agents inheriting from `BaseAgent` with specialized, non-LLM logic. (See Custom Agents )

The following sections detail the core ADK primitives—such as agent hierarchy, workflow agents, and interaction mechanisms—that enable you to construct and manage these multi-agent systems effectively.

## 1. ADK Primitives for Agent Composition

ADK provides core building blocks—primitives—that enable you to structure and manage interactions within your multi-agent system.

Note

The specific parameters or method names for the primitives may vary slightly by SDK language (e.g., `sub_agents` in Python, `subAgents` in Java). Refer to the language-specific API documentation for details.

### 1.1. Agent Hierarchy (Parent agent, Sub Agents)

The foundation for structuring multi-agent systems is the parent-child relationship defined in `BaseAgent` .

- **Establishing Hierarchy:** You create a tree structure by passing a list of agent instances to the `sub_agents` argument when initializing a parent agent. ADK automatically sets the `parent_agent` attribute on each child agent during initialization.
- **Single Parent Rule:** An agent instance can only be added as a sub-agent once. Attempting to assign a second parent will result in a `ValueError` .
- **Importance:** This hierarchy defines the scope for Workflow Agents and influences the potential targets for LLM-Driven Delegation. You can navigate the hierarchy using `agent.parent_agent` or find descendants using `agent.find_agent(name)` .

```
# Conceptual Example: Defining Hierarchy
from google.adk.agents import LlmAgent, BaseAgent

# Define individual agents
greeter = LlmAgent(name="Greeter", model="gemini-2.5-flash")
task_doer = BaseAgent(name="TaskExecutor") # Custom non-LLM agent

# Create parent agent and assign children via sub_agents
coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.5-flash",
    description="I coordinate greetings and tasks.",
    sub_agents=[ # Assign sub_agents here
        greeter,
        task_doer
    ]
)

# Framework automatically sets:
# assert greeter.parent_agent == coordinator
# assert task_doer.parent_agent == coordinator
```

### 1.2. Workflow Agents as Orchestrators

ADK includes specialized agents derived from `BaseAgent` that don't perform tasks themselves but orchestrate the execution flow of their `sub_agents` .

- **`SequentialAgent` :** Executes its `sub_agents` one after another in the order they are listed.
- **Context:** Passes the *same*  `InvocationContext` sequentially, allowing agents to easily pass results via shared state.

```
# Conceptual Example: Sequential Pipeline
from google.adk.agents import SequentialAgent, LlmAgent

step1 = LlmAgent(name="Step1_Fetch", output_key="data") # Saves output to state['data']
step2 = LlmAgent(name="Step2_Process", instruction="Process data from {data}.")

pipeline = SequentialAgent(name="MyPipeline", sub_agents=[step1, step2])
# When pipeline runs, Step2 can access the state['data'] set by Step1.
```

- **`ParallelAgent` :** Executes its `sub_agents` in parallel. Events from sub-agents may be interleaved.
- **Context:** Modifies the `InvocationContext.branch` for each child agent (e.g., `ParentBranch.ChildName` ), providing a distinct contextual path which can be useful for isolating history in some memory implementations.
- **State:** Despite different branches, all parallel children access the *same shared*  `session.state` , enabling them to read initial state and write results (use distinct keys to avoid race conditions).

```
# Conceptual Example: Parallel Execution
from google.adk.agents import ParallelAgent, LlmAgent

fetch_weather = LlmAgent(name="WeatherFetcher", output_key="weather")
fetch_news = LlmAgent(name="NewsFetcher", output_key="news")

gatherer = ParallelAgent(name="InfoGatherer", sub_agents=[fetch_weather, fetch_news])
# When gatherer runs, WeatherFetcher and NewsFetcher run concurrently.
# A subsequent agent could read state['weather'] and state['news'].
```

- **`LoopAgent` :** Executes its `sub_agents` sequentially in a loop.
- **Termination:** The loop stops if the optional `max_iterations` is reached, or if any sub-agent returns an `Event` with `escalate=True` in it's Event Actions.
- **Context & State:** Passes the *same*  `InvocationContext` in each iteration, allowing state changes (e.g., counters, flags) to persist across loops.

```
# Conceptual Example: Loop with Condition
from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
from google.adk.events import Event, EventActions
from google.adk.agents.invocation_context import InvocationContext
from typing import AsyncGenerator

class CheckCondition(BaseAgent): # Custom agent to check state
    async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
        status = ctx.session.state.get("status", "pending")
        is_done = (status == "completed")
        yield Event(author=self.name, actions=EventActions(escalate=is_done)) # Escalate if done

process_step = LlmAgent(name="ProcessingStep") # Agent that might update state['status']

poller = LoopAgent(
    name="StatusPoller",
    max_iterations=,
    sub_agents=[process_step, CheckCondition(name="Checker")]
)
# When poller runs, it executes process_step then Checker repeatedly
# until Checker escalates (state['status'] == 'completed') or 10 iterations pass.
```

### 1.3. Interaction & Communication Mechanisms

Agents within a system often need to exchange data or trigger actions in one another. ADK facilitates this through:

#### a) Shared Session State (session.state)

The most fundamental way for agents operating within the same invocation (and thus sharing the same `Session` object via the `InvocationContext` ) to communicate passively.

- **Mechanism:** One agent (or its tool/callback) writes a value ( `context.state['data_key'] = processed_data` ), and a subsequent agent reads it ( `data = context.state.get('data_key')` ). State changes are tracked via `CallbackContext` .
- **Convenience:** The `output_key` property on `LlmAgent` automatically saves the agent's final response text (or structured output) to the specified state key.
- **Nature:** Asynchronous, passive communication. Ideal for pipelines orchestrated by `SequentialAgent` or passing data across `LoopAgent` iterations.
- **See Also:**  State Management

Invocation Context and `temp:` State

When a parent agent invokes a sub-agent, it passes the same `InvocationContext` . This means they share the same temporary ( `temp:` ) state, which is ideal for passing data that is only relevant for the current turn.

```
# Conceptual Example: Using output_key and reading state
from google.adk.agents import LlmAgent, SequentialAgent

agent_A = LlmAgent(name="AgentA", instruction="Find the capital of France.", output_key="capital_city")
agent_B = LlmAgent(name="AgentB", instruction="Tell me about the city stored in {capital_city}.")

pipeline = SequentialAgent(name="CityInfo", sub_agents=[agent_A, agent_B])
# AgentA runs, saves "Paris" to state['capital_city'].
# AgentB runs, its instruction processor reads state['capital_city'] to get "Paris".
```

#### b) LLM-Driven Delegation (Agent Transfer)

Leverages an `LlmAgent` 's understanding to dynamically route tasks to other suitable agents within the hierarchy.

- **Mechanism:** The agent's LLM generates a specific function call: `transfer_to_agent(agent_name='target_agent_name')` .
- **Handling:** The `AutoFlow` , used by default when sub-agents are present or transfer isn't disallowed, intercepts this call. It identifies the target agent using `root_agent.find_agent()` and updates the `InvocationContext` to switch execution focus.
- **Requires:** The calling `LlmAgent` needs clear `instructions` on when to transfer, and potential target agents need distinct `description` s for the LLM to make informed decisions. Transfer scope (parent, sub-agent, siblings) can be configured on the `LlmAgent` .
- **Nature:** Dynamic, flexible routing based on LLM interpretation.

```
# Conceptual Setup: LLM Transfer
from google.adk.agents import LlmAgent

booking_agent = LlmAgent(name="Booker", description="Handles flight and hotel bookings.")
info_agent = LlmAgent(name="Info", description="Provides general information and answers questions.")

coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.5-flash",
    instruction="You are an assistant. Delegate booking tasks to Booker and info requests to Info.",
    description="Main coordinator.",
    # AutoFlow is typically used implicitly here
    sub_agents=[booking_agent, info_agent]
)
# If coordinator receives "Book a flight", its LLM should generate:
# FunctionCall(name='transfer_to_agent', args={'agent_name': 'Booker'})
# ADK framework then routes execution to booking_agent.
```

#### c) Explicit Invocation (AgentTool)

Allows an `LlmAgent` to treat another `BaseAgent` instance as a callable function or Tool .

- **Mechanism:** Wrap the target agent instance in `AgentTool` and include it in the parent `LlmAgent` 's `tools` list. `AgentTool` generates a corresponding function declaration for the LLM.
- **Handling:** When the parent LLM generates a function call targeting the `AgentTool` , the framework executes `AgentTool.run_async` . This method runs the target agent, captures its final response, forwards any state/artifact changes back to the parent's context, and returns the response as the tool's result.
- **Nature:** Synchronous (within the parent's flow), explicit, controlled invocation like any other tool.
- **(Note:**  `AgentTool` needs to be imported and used explicitly).

```
# Conceptual Setup: Agent as a Tool
from google.adk.agents import LlmAgent, BaseAgent
from google.adk.tools import agent_tool
from pydantic import BaseModel

# Define a target agent (could be LlmAgent or custom BaseAgent)
class ImageGeneratorAgent(BaseAgent): # Example custom agent
    name: str = "ImageGen"
    description: str = "Generates an image based on a prompt."
    # ... internal logic ...
    async def _run_async_impl(self, ctx): # Simplified run logic
        prompt = ctx.session.state.get("image_prompt", "default prompt")
        # ... generate image bytes ...
        image_bytes = b"..."
        yield Event(author=self.name, content=types.Content(parts=[types.Part.from_bytes(image_bytes, "image/png")]))

image_agent = ImageGeneratorAgent()
image_tool = agent_tool.AgentTool(agent=image_agent) # Wrap the agent

# Parent agent uses the AgentTool
artist_agent = LlmAgent(
    name="Artist",
    model="gemini-2.5-flash",
    instruction="Create a prompt and use the ImageGen tool to generate the image.",
    tools=[image_tool] # Include the AgentTool
)
# Artist LLM generates a prompt, then calls:
# FunctionCall(name='ImageGen', args={'image_prompt': 'a cat wearing a hat'})
# Framework calls image_tool.run_async(...), which runs ImageGeneratorAgent.
# The resulting image Part is returned to the Artist agent as the tool result.
```

These primitives provide the flexibility to design multi-agent interactions ranging from tightly coupled sequential workflows to dynamic, LLM-driven delegation networks.

## 2. Common Multi-Agent Patterns using ADK Primitives

By combining ADK's composition primitives, you can implement various established patterns for multi-agent collaboration.

### Coordinator/Dispatcher Pattern

- **Structure:** A central `LlmAgent` (Coordinator) manages several specialized `sub_agents` .
- **Goal:** Route incoming requests to the appropriate specialist agent.
- **ADK Primitives Used:**
- **Hierarchy:** Coordinator has specialists listed in `sub_agents` .
- **Interaction:** Primarily uses **LLM-Driven Delegation** (requires clear `description` s on sub-agents and appropriate `instruction` on Coordinator) or **Explicit Invocation ( `AgentTool` )** (Coordinator includes `AgentTool` -wrapped specialists in its `tools` ).

```
# Conceptual Code: Coordinator using LLM Transfer
from google.adk.agents import LlmAgent

billing_agent = LlmAgent(name="Billing", description="Handles billing inquiries.")
support_agent = LlmAgent(name="Support", description="Handles technical support requests.")

coordinator = LlmAgent(
    name="HelpDeskCoordinator",
    model="gemini-2.5-flash",
    instruction="Route user requests: Use Billing agent for payment issues, Support agent for technical problems.",
    description="Main help desk router.",
    # allow_transfer=True is often implicit with sub_agents in AutoFlow
    sub_agents=[billing_agent, support_agent]
)
# User asks "My payment failed" -> Coordinator's LLM should call transfer_to_agent(agent_name='Billing')
# User asks "I can't log in" -> Coordinator's LLM should call transfer_to_agent(agent_name='Support')
```

### Sequential Pipeline Pattern

- **Structure:** A `SequentialAgent` contains `sub_agents` executed in a fixed order.
- **Goal:** Implement a multi-step process where the output of one step feeds into the next.
- **ADK Primitives Used:**
- **Workflow:**  `SequentialAgent` defines the order.
- **Communication:** Primarily uses **Shared Session State** . Earlier agents write results (often via `output_key` ), later agents read those results from `context.state` .

```
# Conceptual Code: Sequential Data Pipeline
from google.adk.agents import SequentialAgent, LlmAgent

validator = LlmAgent(name="ValidateInput", instruction="Validate the input.", output_key="validation_status")
processor = LlmAgent(name="ProcessData", instruction="Process data if {validation_status} is 'valid'.", output_key="result")
reporter = LlmAgent(name="ReportResult", instruction="Report the result from {result}.")

data_pipeline = SequentialAgent(
    name="DataPipeline",
    sub_agents=[validator, processor, reporter]
)
# validator runs -> saves to state['validation_status']
# processor runs -> reads state['validation_status'], saves to state['result']
# reporter runs -> reads state['result']
```

### Parallel Fan-Out/Gather Pattern

- **Structure:** A `ParallelAgent` runs multiple `sub_agents` concurrently, often followed by a later agent (in a `SequentialAgent` ) that aggregates results.
- **Goal:** Execute independent tasks simultaneously to reduce latency, then combine their outputs.
- **ADK Primitives Used:**
- **Workflow:**  `ParallelAgent` for concurrent execution (Fan-Out). Often nested within a `SequentialAgent` to handle the subsequent aggregation step (Gather).
- **Communication:** Sub-agents write results to distinct keys in **Shared Session State** . The subsequent "Gather" agent reads multiple state keys.

```
# Conceptual Code: Parallel Information Gathering
from google.adk.agents import SequentialAgent, ParallelAgent, LlmAgent

fetch_api1 = LlmAgent(name="API1Fetcher", instruction="Fetch data from API 1.", output_key="api1_data")
fetch_api2 = LlmAgent(name="API2Fetcher", instruction="Fetch data from API 2.", output_key="api2_data")

gather_concurrently = ParallelAgent(
    name="ConcurrentFetch",
    sub_agents=[fetch_api1, fetch_api2]
)

synthesizer = LlmAgent(
    name="Synthesizer",
    instruction="Combine results from {api1_data} and {api2_data}."
)

overall_workflow = SequentialAgent(
    name="FetchAndSynthesize",
    sub_agents=[gather_concurrently, synthesizer] # Run parallel fetch, then synthesize
)
# fetch_api1 and fetch_api2 run concurrently, saving to state.
# synthesizer runs afterwards, reading state['api1_data'] and state['api2_data'].
```

### Hierarchical Task Decomposition

- **Structure:** A multi-level tree of agents where higher-level agents break down complex goals and delegate sub-tasks to lower-level agents.
- **Goal:** Solve complex problems by recursively breaking them down into simpler, executable steps.
- **ADK Primitives Used:**
- **Hierarchy:** Multi-level `parent_agent` / `sub_agents` structure.
- **Interaction:** Primarily **LLM-Driven Delegation** or **Explicit Invocation ( `AgentTool` )** used by parent agents to assign tasks to subagents. Results are returned up the hierarchy (via tool responses or state).

```
# Conceptual Code: Hierarchical Research Task
from google.adk.agents import LlmAgent
from google.adk.tools import agent_tool

# Low-level tool-like agents
web_searcher = LlmAgent(name="WebSearch", description="Performs web searches for facts.")
summarizer = LlmAgent(name="Summarizer", description="Summarizes text.")

# Mid-level agent combining tools
research_assistant = LlmAgent(
    name="ResearchAssistant",
    model="gemini-2.5-flash",
    description="Finds and summarizes information on a topic.",
    tools=[agent_tool.AgentTool(agent=web_searcher), agent_tool.AgentTool(agent=summarizer)]
)

# High-level agent delegating research
report_writer = LlmAgent(
    name="ReportWriter",
    model="gemini-2.5-flash",
    instruction="Write a report on topic X. Use the ResearchAssistant to gather information.",
    tools=[agent_tool.AgentTool(agent=research_assistant)]
    # Alternatively, could use LLM Transfer if research_assistant is a sub_agent
)
# User interacts with ReportWriter.
# ReportWriter calls ResearchAssistant tool.
# ResearchAssistant calls WebSearch and Summarizer tools.
# Results flow back up.
```

### Review/Critique Pattern (Generator-Critic)

- **Structure:** Typically involves two agents within a `SequentialAgent` : a Generator and a Critic/Reviewer.
- **Goal:** Improve the quality or validity of generated output by having a dedicated agent review it.
- **ADK Primitives Used:**
- **Workflow:**  `SequentialAgent` ensures generation happens before review.
- **Communication:**  **Shared Session State** (Generator uses `output_key` to save output; Reviewer reads that state key). The Reviewer might save its feedback to another state key for subsequent steps.

```
# Conceptual Code: Generator-Critic
from google.adk.agents import SequentialAgent, LlmAgent

generator = LlmAgent(
    name="DraftWriter",
    instruction="Write a short paragraph about subject X.",
    output_key="draft_text"
)

reviewer = LlmAgent(
    name="FactChecker",
    instruction="Review the text in {draft_text} for factual accuracy. Output 'valid' or 'invalid' with reasons.",
    output_key="review_status"
)

# Optional: Further steps based on review_status

review_pipeline = SequentialAgent(
    name="WriteAndReview",
    sub_agents=[generator, reviewer]
)
# generator runs -> saves draft to state['draft_text']
# reviewer runs -> reads state['draft_text'], saves status to state['review_status']
```

### Iterative Refinement Pattern

- **Structure:** Uses a `LoopAgent` containing one or more agents that work on a task over multiple iterations.
- **Goal:** Progressively improve a result (e.g., code, text, plan) stored in the session state until a quality threshold is met or a maximum number of iterations is reached.
- **ADK Primitives Used:**
- **Workflow:**  `LoopAgent` manages the repetition.
- **Communication:**  **Shared Session State** is essential for agents to read the previous iteration's output and save the refined version.
- **Termination:** The loop typically ends based on `max_iterations` or a dedicated checking agent setting `escalate=True` in the `Event Actions` when the result is satisfactory.

```
# Conceptual Code: Iterative Code Refinement
from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
from google.adk.events import Event, EventActions
from google.adk.agents.invocation_context import InvocationContext
from typing import AsyncGenerator

# Agent to generate/refine code based on state['current_code'] and state['requirements']
code_refiner = LlmAgent(
    name="CodeRefiner",
    instruction="Read state['current_code'] (if exists) and state['requirements']. Generate/refine Python code to meet requirements. Save to state['current_code'].",
    output_key="current_code" # Overwrites previous code in state
)

# Agent to check if the code meets quality standards
quality_checker = LlmAgent(
    name="QualityChecker",
    instruction="Evaluate the code in state['current_code'] against state['requirements']. Output 'pass' or 'fail'.",
    output_key="quality_status"
)

# Custom agent to check the status and escalate if 'pass'
class CheckStatusAndEscalate(BaseAgent):
    async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]:
        status = ctx.session.state.get("quality_status", "fail")
        should_stop = (status == "pass")
        yield Event(author=self.name, actions=EventActions(escalate=should_stop))

refinement_loop = LoopAgent(
    name="CodeRefinementLoop",
    max_iterations=,
    sub_agents=[code_refiner, quality_checker, CheckStatusAndEscalate(name="StopChecker")]
)
# Loop runs: Refiner -> Checker -> StopChecker
# State['current_code'] is updated each iteration.
# Loop stops if QualityChecker outputs 'pass' (leading to StopChecker escalating) or after 5 iterations.
```

### Human-in-the-Loop Pattern

- **Structure:** Integrates human intervention points within an agent workflow.
- **Goal:** Allow for human oversight, approval, correction, or tasks that AI cannot perform.
- **ADK Primitives Used (Conceptual):**
- **Interaction:** Can be implemented using a custom **Tool** that pauses execution and sends a request to an external system (e.g., a UI, ticketing system) waiting for human input. The tool then returns the human's response to the agent.
- **Workflow:** Could use **LLM-Driven Delegation** ( `transfer_to_agent` ) targeting a conceptual "Human Agent" that triggers the external workflow, or use the custom tool within an `LlmAgent` .
- **State/Callbacks:** State can hold task details for the human; callbacks can manage the interaction flow.
- **Note:** ADK doesn't have a built-in "Human Agent" type, so this requires custom integration.

```
# Conceptual Code: Using a Tool for Human Approval
from google.adk.agents import LlmAgent, SequentialAgent
from google.adk.tools import FunctionTool

# --- Assume external_approval_tool exists ---
# This tool would:
# 1. Take details (e.g., request_id, amount, reason).
# 2. Send these details to a human review system (e.g., via API).
# 3. Poll or wait for the human response (approved/rejected).
# 4. Return the human's decision.
# async def external_approval_tool(amount: float, reason: str) -> str: ...
approval_tool = FunctionTool(func=external_approval_tool)

# Agent that prepares the request
prepare_request = LlmAgent(
    name="PrepareApproval",
    instruction="Prepare the approval request details based on user input. Store amount and reason in state.",
    # ... likely sets state['approval_amount'] and state['approval_reason'] ...
)

# Agent that calls the human approval tool
request_approval = LlmAgent(
    name="RequestHumanApproval",
    instruction="Use the external_approval_tool with amount from state['approval_amount'] and reason from state['approval_reason'].",
    tools=[approval_tool],
    output_key="human_decision"
)

# Agent that proceeds based on human decision
process_decision = LlmAgent(
    name="ProcessDecision",
    instruction="Check {human_decision}. If 'approved', proceed. If 'rejected', inform user."
)

approval_workflow = SequentialAgent(
    name="HumanApprovalWorkflow",
    sub_agents=[prepare_request, request_approval, process_decision]
)
```

These patterns provide starting points for structuring your multi-agent systems. You can mix and match them as needed to create the most effective architecture for your specific application.

-----------------

# Build agents with Agent Config

The ADK Agent Config feature lets you build an ADK workflow without writing
code. An Agent Config uses a YAML format text file with a brief description of
the agent, allowing just about anyone to assemble and run an ADK agent. The
following is a simple example of an basic Agent Config definition:

```
name: assistant_agent
model: gemini-2.5-flash
description: A helper agent that can answer users' questions.
instruction: You are an agent to help answer users' various questions.
```

You can use Agent Config files to build more complex agents which can
incorporate Functions, Tools, Sub-Agents, and more. This page describes how to
build and run ADK workflows with the Agent Config feature. For detailed
information on the syntax and settings supported by the Agent Config format,
see the Agent Config syntax reference .

Experimental

The Agent Config feature is experimental and has some known limitations . We welcome your feedback !

## Get started

This section describes how to set up and start building agents with the ADK and
the Agent Config feature, including installation setup, building an agent, and
running your agent.

### Setup

You need to install the Google Agent Development Kit libraries, and provide an
access key for a generative AI model such as Gemini API. This section provides
details on what you must install and configure before you can run agents with
the Agent Config files.

Note

The Agent Config feature currently only supports Gemini models. For more
information about additional; functional restrictions, see Known limitations .

To setup ADK for use with Agent Config:

1. Install the ADK Python libraries by following the Installation instructions. *Python is currently required.* For more information, see the Known limitations .
2. Verify that ADK is installed by running the following command in your
    terminal:

```
adk --version
```

This command should show the ADK version you have installed.

Tip

If the `adk` command fails to run and the version is not listed in step 2, make
sure your Python environment is active. Execute `source .venv/bin/activate` in
your terminal on Mac and Linux. For other platform commands, see the Installation page.

### Build an agent

You build an agent with Agent Config using the `adk create` command to create
the project files for an agent, and then editing the `root_agent.yaml` file it
generates for you.

To create an ADK project for use with Agent Config:

1. In your terminal window, run the following command to create a
    config-based agent:

```
adk create --type=config my_agent
```

This command generates a `my_agent/` folder, containing a `root_agent.yaml` file and an `.env` file.

2. In the `my_agent/.env` file, set environment variables for your agent to
    access generative AI models and other services:

1. For Gemini model access through Google API, add a line to the
    file with your API key:

```
GOOGLE_GENAI_USE_VERTEXAI=0
GOOGLE_API_KEY=<your-Google-Gemini-API-key>
```

You can get an API key from the Google AI Studio API Keys page.

2. For Gemini model access through Google Cloud, add these lines to the file:

```
GOOGLE_GENAI_USE_VERTEXAI=1
GOOGLE_CLOUD_PROJECT=<your_gcp_project>
GOOGLE_CLOUD_LOCATION=us-central1
```

For information on creating a Cloud Project, see the Google Cloud docs
for Creating and managing projects .

3. Using text editor, edit the Agent Config file `my_agent/root_agent.yaml` , as shown below:
```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json

name: assistant_agent

model: gemini-2.5-flash

description: A helper agent that can answer users' questions.

instruction: You are an agent to help answer users' various questions.
```

You can discover more configuration options for your `root_agent.yaml` agent
configuration file by referring to the ADK samples repository or the Agent Config syntax reference.

### Run the agent

Once you have completed editing your Agent Config, you can run your agent using
the web interface, command line terminal execution, or API server mode.

To run your Agent Config-defined agent:

1. In your terminal, navigate to the `my_agent/` directory containing the `root_agent.yaml` file.
2. Type one of the following commands to run your agent:
- `adk web` - Run web UI interface for your agent.
- `adk run` - Run your agent in the terminal without a user
    interface.

- `adk api_server` - Run your agent as a service that can be
    used by other applications.

For more information on the ways to run your agent, see the *Run Your Agent* topic in the Quickstart .
For more information about the ADK command line options, see the ADK CLI reference .

## Example configs

This section shows examples of Agent Config files to get you started building
agents. For additional and more complete examples, see the ADK samples repository .

### Built-in tool example

The following example uses a built-in ADK tool function for using google search
to provide functionality to the agent. This agent automatically uses the search
tool to reply to user requests.

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json

name: search_agent

model: gemini-2.5-flash

description: 'an agent whose job it is to perform Google search queries and answer questions about the results.'

instruction: You are an agent whose job is to perform Google search queries and answer questions about the results.

tools:

  - name: google_search
```

For more details, see the full code for this sample in the ADK sample repository .

### Custom tool example

The following example uses a custom tool built with Python code and listed in
the `tools:` section of the config file. The agent uses this tool to check if a
list of numbers provided by the user are prime numbers.

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json

agent_class: LlmAgent

model: gemini-2.5-flash

name: prime_agent

description: Handles checking if numbers are prime.

instruction: |

  You are responsible for checking whether numbers are prime.

  When asked to check primes, you must call the check_prime tool with a list of integers.

  Never attempt to determine prime numbers manually.

  Return the prime number results to the root agent.

tools:

  - name: ma_llm.check_prime
```

For more details, see the full code for this sample in the ADK sample repository .

### Sub-agents example

The following example shows an agent defined with two sub-agents in the `sub_agents:` section, and an example tool in the `tools:` section of the config
file. This agent determines what the user wants, and delegates to one of the
sub-agents to resolve the request. The sub-agents are defined using Agent Config
YAML files.

```
# yaml-language-server: $schema=https://raw.githubusercontent.com/google/adk-python/refs/heads/main/src/google/adk/agents/config_schemas/AgentConfig.json

agent_class: LlmAgent

model: gemini-2.5-flash

name: root_agent

description: Learning assistant that provides tutoring in code and math.

instruction: |

  You are a learning assistant that helps students with coding and math questions.

  You delegate coding questions to the code_tutor_agent and math questions to the math_tutor_agent.

  Follow these steps:

  1. If the user asks about programming or coding, delegate to the code_tutor_agent.

  2. If the user asks about math concepts or problems, delegate to the math_tutor_agent.

  3. Always provide clear explanations and encourage learning.

sub_agents:

  - config_path: code_tutor_agent.yaml

  - config_path: math_tutor_agent.yaml
```

For more details, see the full code for this sample in the ADK sample repository .

## Deploy agent configs

You can deploy Agent Config agents with Cloud Run and Agent Engine ,
using the same procedure as code-based agents. For more information on how
to prepare and deploy Agent Config-based agents, see the Cloud Run and Agent Engine deployment guides.

## Known limitations

The Agent Config feature is experimental and includes the following
limitations:

- **Model support:** Only Gemini models are currently supported.
    Integration with third-party models is in progress.

- **Programming language:** The Agent Config feature currently supports
    only Python code for tools and other functionality requiring programming code.

- **ADK Tool support:** The following ADK tools are supported by the Agent
    Config feature, but *not all tools are fully supported* :
- `google_search`
- `load_artifacts`
- `url_context`
- `exit_loop`
- `preload_memory`
- `get_user_choice`
- `enterprise_web_search`
- `load_web_page` : Requires a fully-qualified path to access web
    pages.

- **Agent Type Support:** The `LangGraphAgent` and `A2aAgent` types are
    not yet supported.
- `AgentTool`
- `LongRunningFunctionTool`
- `VertexAiSearchTool`
- `MCPToolset`
- `CrewaiTool`
- `LangchainTool`
- `ExampleTool`

## Next steps

For ideas on how and what to build with ADK Agent Configs, see the yaml-based
agent definitions in the ADK adk-samples repository. For detailed information on the syntax and settings supported by
the Agent Config format, see the Agent Config syntax reference .

-----------------

# Tools

## What is a Tool?

In the context of ADK, a Tool represents a specific
capability provided to an AI agent, enabling it to perform actions and interact
with the world beyond its core text generation and reasoning abilities. What
distinguishes capable agents from basic language models is often their effective
use of tools.

Technically, a tool is typically a modular code component— **like a Python/ Java
function** , a class method, or even another specialized agent—designed to
execute a distinct, predefined task. These tasks often involve interacting with
external systems or data.

### Key Characteristics

**Action-Oriented:** Tools perform specific actions, such as:

- Querying databases
- Making API requests (e.g., fetching weather data, booking systems)
- Searching the web
- Executing code snippets
- Retrieving information from documents (RAG)
- Interacting with other software or services

**Extends Agent capabilities:** They empower agents to access real-time information, affect external systems, and overcome the knowledge limitations inherent in their training data.

**Execute predefined logic:** Crucially, tools execute specific, developer-defined logic. They do not possess their own independent reasoning capabilities like the agent's core Large Language Model (LLM). The LLM reasons about which tool to use, when, and with what inputs, but the tool itself just executes its designated function.

## How Agents Use Tools

Agents leverage tools dynamically through mechanisms often involving function calling. The process generally follows these steps:

1. **Reasoning:** The agent's LLM analyzes its system instruction, conversation history, and user request.
2. **Selection:** Based on the analysis, the LLM decides on which tool, if any, to execute, based on the tools available to the agent and the docstrings that describes each tool.
3. **Invocation:** The LLM generates the required arguments (inputs) for the selected tool and triggers its execution.
4. **Observation:** The agent receives the output (result) returned by the tool.
5. **Finalization:** The agent incorporates the tool's output into its ongoing reasoning process to formulate the next response, decide the subsequent step, or determine if the goal has been achieved.

Think of the tools as a specialized toolkit that the agent's intelligent core (the LLM) can access and utilize as needed to accomplish complex tasks.

## Tool Types in ADK

ADK offers flexibility by supporting several types of tools:

1. **Function Tools :** Tools created by you, tailored to your specific application's needs.
- **Functions/Methods :** Define standard synchronous functions or methods in your code (e.g., Python def).
- **Agents-as-Tools :** Use another, potentially specialized, agent as a tool for a parent agent.
- **Long Running Function Tools :** Support for tools that perform asynchronous operations or take significant time to complete.

2. **Built-in Tools :** Ready-to-use tools provided by the framework for common tasks.
        Examples: Google Search, Code Execution, Retrieval-Augmented Generation (RAG).

3. **Third-Party Tools :** Integrate tools seamlessly from popular external libraries.
        Examples: LangChain Tools, CrewAI Tools.

Navigate to the respective documentation pages linked above for detailed information and examples for each tool type.

## Referencing Tool in Agent’s Instructions

Within an agent's instructions, you can directly reference a tool by using its **function name.** If the tool's **function name** and **docstring** are sufficiently descriptive, your instructions can primarily focus on **when the Large Language Model (LLM) should utilize the tool** . This promotes clarity and helps the model understand the intended use of each tool.

It is **crucial to clearly instruct the agent on how to handle different return values** that a tool might produce. For example, if a tool returns an error message, your instructions should specify whether the agent should retry the operation, give up on the task, or request additional information from the user.

Furthermore, ADK supports the sequential use of tools, where the output of one tool can serve as the input for another. When implementing such workflows, it's important to **describe the intended sequence of tool usage** within the agent's instructions to guide the model through the necessary steps.

### Example

The following example showcases how an agent can use tools by **referencing their function names in its instructions** . It also demonstrates how to guide the agent to **handle different return values from tools** , such as success or error messages, and how to orchestrate the **sequential use of multiple tools** to accomplish a task.

```

import asyncio
from google.adk.agents import Agent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

APP_NAME="weather_sentiment_agent"
USER_ID="user1234"
SESSION_ID="1234"
MODEL_ID="gemini-2.5-flash"

# Tool 1
def get_weather_report(city: str) -> dict:
    """Retrieves the current weather report for a specified city.

    Returns:
        dict: A dictionary containing the weather information with a 'status' key ('success' or 'error') and a 'report' key with the weather details if successful, or an 'error_message' if an error occurred.
    """
    if city.lower() == "london":
        return {"status": "success", "report": "The current weather in London is cloudy with a temperature of 18 degrees Celsius and a chance of rain."}
    elif city.lower() == "paris":
        return {"status": "success", "report": "The weather in Paris is sunny with a temperature of 25 degrees Celsius."}
    else:
        return {"status": "error", "error_message": f"Weather information for '{city}' is not available."}

weather_tool = FunctionTool(func=get_weather_report)

# Tool 2
def analyze_sentiment(text: str) -> dict:
    """Analyzes the sentiment of the given text.

    Returns:
        dict: A dictionary with 'sentiment' ('positive', 'negative', or 'neutral') and a 'confidence' score.
    """
    if "good" in text.lower() or "sunny" in text.lower():
        return {"sentiment": "positive", "confidence": 0.8}
    elif "rain" in text.lower() or "bad" in text.lower():
        return {"sentiment": "negative", "confidence": 0.7}
    else:
        return {"sentiment": "neutral", "confidence": 0.6}

sentiment_tool = FunctionTool(func=analyze_sentiment)

# Agent
weather_sentiment_agent = Agent(
    model=MODEL_ID,
    name='weather_sentiment_agent',
    instruction="""You are a helpful assistant that provides weather information and analyzes the sentiment of user feedback.
**If the user asks about the weather in a specific city, use the 'get_weather_report' tool to retrieve the weather details.**
**If the 'get_weather_report' tool returns a 'success' status, provide the weather report to the user.**
**If the 'get_weather_report' tool returns an 'error' status, inform the user that the weather information for the specified city is not available and ask if they have another city in mind.**
**After providing a weather report, if the user gives feedback on the weather (e.g., 'That's good' or 'I don't like rain'), use the 'analyze_sentiment' tool to understand their sentiment.** Then, briefly acknowledge their sentiment.
You can handle these tasks sequentially if needed.""",
    tools=[weather_tool, sentiment_tool]
)

async def main():
    """Main function to run the agent asynchronously."""
    # Session and Runner Setup
    session_service = InMemorySessionService()
    # Use 'await' to correctly create the session
    await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)

    runner = Runner(agent=weather_sentiment_agent, app_name=APP_NAME, session_service=session_service)

    # Agent Interaction
    query = "weather in london?"
    print(f"User Query: {query}")
    content = types.Content(role='user', parts=[types.Part(text=query)])

    # The runner's run method handles the async loop internally
    events = runner.run(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    for event in events:
        if event.is_final_response():
            final_response = event.content.parts[].text
            print("Agent Response:", final_response)

# Standard way to run the main async function
if __name__ == "__main__":
    asyncio.run(main())
```

## Tool Context

For more advanced scenarios, ADK allows you to access additional contextual information within your tool function by including the special parameter `tool_context: ToolContext` . By including this in the function signature, ADK will **automatically** provide an **instance of the ToolContext** class when your tool is called during agent execution.

The **ToolContext** provides access to several key pieces of information and control levers:

- `state: State` : Read and modify the current session's state. Changes made here are tracked and persisted.
- `actions: EventActions` : Influence the agent's subsequent actions after the tool runs (e.g., skip summarization, transfer to another agent).
- `function_call_id: str` : The unique identifier assigned by the framework to this specific invocation of the tool. Useful for tracking and correlating with authentication responses. This can also be helpful when multiple tools are called within a single model response.
- `function_call_event_id: str` : This attribute provides the unique identifier of the **event** that triggered the current tool call. This can be useful for tracking and logging purposes.
- `auth_response: Any` : Contains the authentication response/credentials if an authentication flow was completed before this tool call.
- Access to Services: Methods to interact with configured services like Artifacts and Memory.

Note that you shouldn't include the `tool_context` parameter in the tool function docstring. Since `ToolContext` is automatically injected by the ADK framework *after* the LLM decides to call the tool function, it is not relevant for the LLM's decision-making and including it can confuse the LLM.

### State Management

The `tool_context.state` attribute provides direct read and write access to the state associated with the current session. It behaves like a dictionary but ensures that any modifications are tracked as deltas and persisted by the session service. This enables tools to maintain and share information across different interactions and agent steps.

- **Reading State** : Use standard dictionary access ( `tool_context.state['my_key']` ) or the `.get()` method ( `tool_context.state.get('my_key', default_value)` ).
- **Writing State** : Assign values directly ( `tool_context.state['new_key'] = 'new_value'` ). These changes are recorded in the state_delta of the resulting event.
- **State Prefixes** : Remember the standard state prefixes:

- `app:*` : Shared across all users of the application.
- `user:*` : Specific to the current user across all their sessions.
- (No prefix): Specific to the current session.
- `temp:*` : Temporary, not persisted across invocations (useful for passing data within a single run call but generally less useful inside a tool context which operates between LLM calls).

```
from google.adk.tools import ToolContext, FunctionTool

def update_user_preference(preference: str, value: str, tool_context: ToolContext):
    """Updates a user-specific preference."""
    user_prefs_key = "user:preferences"
    # Get current preferences or initialize if none exist
    preferences = tool_context.state.get(user_prefs_key, {})
    preferences[preference] = value
    # Write the updated dictionary back to the state
    tool_context.state[user_prefs_key] = preferences
    print(f"Tool: Updated user preference '{preference}' to '{value}'")
    return {"status": "success", "updated_preference": preference}

pref_tool = FunctionTool(func=update_user_preference)

# In an Agent:
# my_agent = Agent(..., tools=[pref_tool])

# When the LLM calls update_user_preference(preference='theme', value='dark', ...):
# The tool_context.state will be updated, and the change will be part of the
# resulting tool response event's actions.state_delta.
```

### Controlling Agent Flow

The `tool_context.actions` attribute ( `ToolContext.actions()` in Java) holds an **EventActions** object. Modifying attributes on this object allows your tool to influence what the agent or framework does after the tool finishes execution.

- **`skip_summarization: bool`** : (Default: False) If set to True, instructs the ADK to bypass the LLM call that typically summarizes the tool's output. This is useful if your tool's return value is already a user-ready message.
- **`transfer_to_agent: str`** : Set this to the name of another agent. The framework will halt the current agent's execution and **transfer control of the conversation to the specified agent** . This allows tools to dynamically hand off tasks to more specialized agents.
- **`escalate: bool`** : (Default: False) Setting this to True signals that the current agent cannot handle the request and should pass control up to its parent agent (if in a hierarchy). In a LoopAgent, setting **escalate=True** in a sub-agent's tool will terminate the loop.

#### Example

```

from google.adk.agents import Agent
from google.adk.tools import FunctionTool
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import ToolContext
from google.genai import types

APP_NAME="customer_support_agent"
USER_ID="user1234"
SESSION_ID="1234"

def check_and_transfer(query: str, tool_context: ToolContext) -> str:
    """Checks if the query requires escalation and transfers to another agent if needed."""
    if "urgent" in query.lower():
        print("Tool: Detected urgency, transferring to the support agent.")
        tool_context.actions.transfer_to_agent = "support_agent"
        return "Transferring to the support agent..."
    else:
        return f"Processed query: '{query}'. No further action needed."

escalation_tool = FunctionTool(func=check_and_transfer)

main_agent = Agent(
    model='gemini-2.5-flash',
    name='main_agent',
    instruction="""You are the first point of contact for customer support of an analytics tool. Answer general queries. If the user indicates urgency, use the 'check_and_transfer' tool.""",
    tools=[check_and_transfer]
)

support_agent = Agent(
    model='gemini-2.5-flash',
    name='support_agent',
    instruction="""You are the dedicated support agent. Mentioned you are a support handler and please help the user with their urgent issue."""
)

main_agent.sub_agents = [support_agent]

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=main_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("this is urgent, i cant login")
```

##### Explanation

- We define two agents: `main_agent` and `support_agent` . The `main_agent` is designed to be the initial point of contact.
- The `check_and_transfer` tool, when called by `main_agent` , examines the user's query.
- If the query contains the word "urgent", the tool accesses the `tool_context` , specifically **`tool_context.actions`** , and sets the transfer_to_agent attribute to `support_agent` .
- This action signals to the framework to **transfer the control of the conversation to the agent named `support_agent`** .
- When the `main_agent` processes the urgent query, the `check_and_transfer` tool triggers the transfer. The subsequent response would ideally come from the `support_agent` .
- For a normal query without urgency, the tool simply processes it without triggering a transfer.

This example illustrates how a tool, through EventActions in its ToolContext, can dynamically influence the flow of the conversation by transferring control to another specialized agent.

### Authentication

ToolContext provides mechanisms for tools interacting with authenticated APIs. If your tool needs to handle authentication, you might use the following:

- **`auth_response`** : Contains credentials (e.g., a token) if authentication was already handled by the framework before your tool was called (common with RestApiTool and OpenAPI security schemes).
- **`request_credential(auth_config: dict)`** : Call this method if your tool determines authentication is needed but credentials aren't available. This signals the framework to start an authentication flow based on the provided auth_config.
- **`get_auth_response()`** : Call this in a subsequent invocation (after request_credential was successfully handled) to retrieve the credentials the user provided.

For detailed explanations of authentication flows, configuration, and examples, please refer to the dedicated Tool Authentication documentation page.

### Context-Aware Data Access Methods

These methods provide convenient ways for your tool to interact with persistent data associated with the session or user, managed by configured services.

- **`list_artifacts()`** (or **`listArtifacts()`** in Java): Returns a list of filenames (or keys) for all artifacts currently stored for the session via the artifact_service. Artifacts are typically files (images, documents, etc.) uploaded by the user or generated by tools/agents.
- **`load_artifact(filename: str)`** : Retrieves a specific artifact by its filename from the **artifact_service** . You can optionally specify a version; if omitted, the latest version is returned. Returns a `google.genai.types.Part` object containing the artifact data and mime type, or None if not found.
- **`save_artifact(filename: str, artifact: types.Part)`** : Saves a new version of an artifact to the artifact_service. Returns the new version number (starting from 0).
- **`search_memory(query: str)`**

Queries the user's long-term memory using the configured `memory_service` . This is useful for retrieving relevant information from past interactions or stored knowledge. The structure of the **SearchMemoryResponse** depends on the specific memory service implementation but typically contains relevant text snippets or conversation excerpts.

#### Example

```

from google.adk.tools import ToolContext, FunctionTool
from google.genai import types

def process_document(
    document_name: str, analysis_query: str, tool_context: ToolContext
) -> dict:
    """Analyzes a document using context from memory."""

    # 1. Load the artifact
    print(f"Tool: Attempting to load artifact: {document_name}")
    document_part = tool_context.load_artifact(document_name)

    if not document_part:
        return {"status": "error", "message": f"Document '{document_name}' not found."}

    document_text = document_part.text  # Assuming it's text for simplicity
    print(f"Tool: Loaded document '{document_name}' ({len(document_text)} chars).")

    # 2. Search memory for related context
    print(f"Tool: Searching memory for context related to: '{analysis_query}'")
    memory_response = tool_context.search_memory(
        f"Context for analyzing document about {analysis_query}"
    )
    memory_context = "\n".join(
        [
            m.events[].content.parts[0].text
            for m in memory_response.memories
            if m.events and m.events[0].content
        ]
    )  # Simplified extraction
    print(f"Tool: Found memory context: {memory_context[:100]}...")

    # 3. Perform analysis (placeholder)
    analysis_result = f"Analysis of '{document_name}' regarding '{analysis_query}' using memory context: [Placeholder Analysis Result]"
    print("Tool: Performed analysis.")

    # 4. Save the analysis result as a new artifact
    analysis_part = types.Part.from_text(text=analysis_result)
    new_artifact_name = f"analysis_{document_name}"
    version = await tool_context.save_artifact(new_artifact_name, analysis_part)
    print(f"Tool: Saved analysis result as '{new_artifact_name}' version {version}.")

    return {
        "status": "success",
        "analysis_artifact": new_artifact_name,
        "version": version,
    }

doc_analysis_tool = FunctionTool(func=process_document)

# In an Agent:
# Assume artifact 'report.txt' was previously saved.
# Assume memory service is configured and has relevant past data.
# my_agent = Agent(..., tools=[doc_analysis_tool], artifact_service=..., memory_service=...)
```

By leveraging the **ToolContext** , developers can create more sophisticated and context-aware custom tools that seamlessly integrate with ADK's architecture and enhance the overall capabilities of their agents.

## Defining Effective Tool Functions

When using a method or function as an ADK Tool, how you define it significantly impacts the agent's ability to use it correctly. The agent's Large Language Model (LLM) relies heavily on the function's **name** , **parameters (arguments)** , **type hints** , and **docstring** / **source code comments** to understand its purpose and generate the correct call.

Here are key guidelines for defining effective tool functions:

- **Function Name:**

- Use descriptive, verb-noun based names that clearly indicate the action (e.g., `get_weather` , `searchDocuments` , `schedule_meeting` ).
- Avoid generic names like `run` , `process` , `handle_data` , or overly ambiguous names like `doStuff` . Even with a good description, a name like `do_stuff` might confuse the model about when to use the tool versus, for example, `cancelFlight` .
- The LLM uses the function name as a primary identifier during tool selection.

- **Parameters (Arguments):**

- Your function can have any number of parameters.
- Use clear and descriptive names (e.g., `city` instead of `c` , `search_query` instead of `q` ).
- **Provide type hints in Python** for all parameters (e.g., `city: str` , `user_id: int` , `items: list[str]` ). This is essential for ADK to generate the correct schema for the LLM.
- Ensure all parameter types are **JSON serializable** . All java primitives as well as standard Python types like `str` , `int` , `float` , `bool` , `list` , `dict` , and their combinations are generally safe. Avoid complex custom class instances as direct parameters unless they have a clear JSON representation.
- **Do not set default values** for parameters. E.g., `def my_func(param1: str = "default")` . Default values are not reliably supported or used by the underlying models during function call generation. All necessary information should be derived by the LLM from the context or explicitly requested if missing.
- **`self` / `cls` Handled Automatically:** Implicit parameters like `self` (for instance methods) or `cls` (for class methods) are automatically handled by ADK and excluded from the schema shown to the LLM. You only need to define type hints and descriptions for the logical parameters your tool requires the LLM to provide.

- **Return Type:**

- The function's return value **must be a dictionary ( `dict` )** in Python or a **Map** in Java.
- If your function returns a non-dictionary type (e.g., a string, number, list), the ADK framework will automatically wrap it into a dictionary/Map like `{'result': your_original_return_value}` before passing the result back to the model.
- Design the dictionary/Map keys and values to be **descriptive and easily understood *by the LLM*** . Remember, the model reads this output to decide its next step.
- Include meaningful keys. For example, instead of returning just an error code like `500` , return `{'status': 'error', 'error_message': 'Database connection failed'}` .
- It's a **highly recommended practice** to include a `status` key (e.g., `'success'` , `'error'` , `'pending'` , `'ambiguous'` ) to clearly indicate the outcome of the tool execution for the model.

- **Docstring / Source Code Comments:**

- **This is critical.** The docstring is the primary source of descriptive information for the LLM.
- **Clearly state what the tool *does* .** Be specific about its purpose and limitations.
- **Explain *when* the tool should be used.** Provide context or example scenarios to guide the LLM's decision-making.
- **Describe *each parameter* clearly.** Explain what information the LLM needs to provide for that argument.
- Describe the **structure and meaning of the expected `dict` return value** , especially the different `status` values and associated data keys.
- **Do not describe the injected ToolContext parameter** . Avoid mentioning the optional `tool_context: ToolContext` parameter within the docstring description since it is not a parameter the LLM needs to know about. ToolContext is injected by ADK, *after* the LLM decides to call it.

**Example of a good definition:**

```
def lookup_order_status(order_id: str) -> dict:
  """Fetches the current status of a customer's order using its ID.

  Use this tool ONLY when a user explicitly asks for the status of
  a specific order and provides the order ID. Do not use it for
  general inquiries.

  Args:
      order_id: The unique identifier of the order to look up.

  Returns:
      A dictionary indicating the outcome.
      On success, status is 'success' and includes an 'order' dictionary.
      On failure, status is 'error' and includes an 'error_message'.
      Example success: {'status': 'success', 'order': {'state': 'shipped', 'tracking_number': '1Z9...'}}
      Example error: {'status': 'error', 'error_message': 'Order ID not found.'}
  """
  # ... function implementation to fetch status ...
  if status_details := fetch_status_from_backend(order_id):
    return {
        "status": "success",
        "order": {
            "state": status_details.state,
            "tracking_number": status_details.tracking,
        },
    }
  else:
    return {"status": "error", "error_message": f"Order ID {order_id} not found."}
```

- **Simplicity and Focus:**
- **Keep Tools Focused:** Each tool should ideally perform one well-defined task.
- **Fewer Parameters are Better:** Models generally handle tools with fewer, clearly defined parameters more reliably than those with many optional or complex ones.
- **Use Simple Data Types:** Prefer basic types ( `str` , `int` , `bool` , `float` , `List[str]` , in **Python** , or `int` , `byte` , `short` , `long` , `float` , `double` , `boolean` and `char` in **Java** ) over complex custom classes or deeply nested structures as parameters when possible.
- **Decompose Complex Tasks:** Break down functions that perform multiple distinct logical steps into smaller, more focused tools. For instance, instead of a single `update_user_profile(profile: ProfileObject)` tool, consider separate tools like `update_user_name(name: str)` , `update_user_address(address: str)` , `update_user_preferences(preferences: list[str])` , etc. This makes it easier for the LLM to select and use the correct capability.

By adhering to these guidelines, you provide the LLM with the clarity and structure it needs to effectively utilize your custom function tools, leading to more capable and reliable agent behavior.

## Toolsets: Grouping and Dynamically Providing Tools

Beyond individual tools, ADK introduces the concept of a **Toolset** via the `BaseToolset` interface (defined in `google.adk.tools.base_toolset` ). A toolset allows you to manage and provide a collection of `BaseTool` instances, often dynamically, to an agent.

This approach is beneficial for:

- **Organizing Related Tools:** Grouping tools that serve a common purpose (e.g., all tools for mathematical operations, or all tools interacting with a specific API).
- **Dynamic Tool Availability:** Enabling an agent to have different tools available based on the current context (e.g., user permissions, session state, or other runtime conditions). The `get_tools` method of a toolset can decide which tools to expose.
- **Integrating External Tool Providers:** Toolsets can act as adapters for tools coming from external systems, like an OpenAPI specification or an MCP server, converting them into ADK-compatible `BaseTool` objects.

### The BaseToolset Interface

Any class acting as a toolset in ADK should implement the `BaseToolset` abstract base class. This interface primarily defines two methods:

- **`async def get_tools(...) -> list[BaseTool]:`** This is the core method of a toolset. When an ADK agent needs to know its available tools, it will call `get_tools()` on each `BaseToolset` instance provided in its `tools` list.

- It receives an optional `readonly_context` (an instance of `ReadonlyContext` ). This context provides read-only access to information like the current session state ( `readonly_context.state` ), agent name, and invocation ID. The toolset can use this context to dynamically decide which tools to return.
- It **must** return a `list` of `BaseTool` instances (e.g., `FunctionTool` , `RestApiTool` ).

- **`async def close(self) -> None:`** This asynchronous method is called by the ADK framework when the toolset is no longer needed, for example, when an agent server is shutting down or the `Runner` is being closed. Implement this method to perform any necessary cleanup, such as closing network connections, releasing file handles, or cleaning up other resources managed by the toolset.

### Using Toolsets with Agents

You can include instances of your `BaseToolset` implementations directly in an `LlmAgent` 's `tools` list, alongside individual `BaseTool` instances.

When the agent initializes or needs to determine its available capabilities, the ADK framework will iterate through the `tools` list:

- If an item is a `BaseTool` instance, it's used directly.
- If an item is a `BaseToolset` instance, its `get_tools()` method is called (with the current `ReadonlyContext` ), and the returned list of `BaseTool` s is added to the agent's available tools.

### Example: A Simple Math Toolset

Let's create a basic example of a toolset that provides simple arithmetic operations.

```
# 1. Define the individual tool functions
def add_numbers(a: int, b: int, tool_context: ToolContext) -> Dict[str, Any]:
    """Adds two integer numbers.
    Args:
        a: The first number.
        b: The second number.
    Returns:
        A dictionary with the sum, e.g., {'status': 'success', 'result': 5}
    """
    print(f"Tool: add_numbers called with a={a}, b={b}")
    result = a + b
    # Example: Storing something in tool_context state
    tool_context.state["last_math_operation"] = "addition"
    return {"status": "success", "result": result}

def subtract_numbers(a: int, b: int) -> Dict[str, Any]:
    """Subtracts the second number from the first.
    Args:
        a: The first number.
        b: The second number.
    Returns:
        A dictionary with the difference, e.g., {'status': 'success', 'result': 1}
    """
    print(f"Tool: subtract_numbers called with a={a}, b={b}")
    return {"status": "success", "result": a - b}

# 2. Create the Toolset by implementing BaseToolset
class SimpleMathToolset(BaseToolset):
    def __init__(self, prefix: str = "math_"):
        self.prefix = prefix
        # Create FunctionTool instances once
        self._add_tool = FunctionTool(
            func=add_numbers,
            name=f"{self.prefix}add_numbers",  # Toolset can customize names
        )
        self._subtract_tool = FunctionTool(
            func=subtract_numbers, name=f"{self.prefix}subtract_numbers"
        )
        print(f"SimpleMathToolset initialized with prefix '{self.prefix}'")

    async def get_tools(
        self, readonly_context: Optional[ReadonlyContext] = None
    ) -> List[BaseTool]:
        print(f"SimpleMathToolset.get_tools() called.")
        # Example of dynamic behavior:
        # Could use readonly_context.state to decide which tools to return
        # For instance, if readonly_context.state.get("enable_advanced_math"):
        #    return [self._add_tool, self._subtract_tool, self._multiply_tool]

        # For this simple example, always return both tools
        tools_to_return = [self._add_tool, self._subtract_tool]
        print(f"SimpleMathToolset providing tools: {[t.name for t in tools_to_return]}")
        return tools_to_return

    async def close(self) -> None:
        # No resources to clean up in this simple example
        print(f"SimpleMathToolset.close() called for prefix '{self.prefix}'.")
        await asyncio.sleep()  # Placeholder for async cleanup if needed

# 3. Define an individual tool (not part of the toolset)
def greet_user(name: str = "User") -> Dict[str, str]:
    """Greets the user."""
    print(f"Tool: greet_user called with name={name}")
    return {"greeting": f"Hello, {name}!"}

greet_tool = FunctionTool(func=greet_user)

# 4. Instantiate the toolset
math_toolset_instance = SimpleMathToolset(prefix="calculator_")

# 5. Define an agent that uses both the individual tool and the toolset
calculator_agent = LlmAgent(
    name="CalculatorAgent",
    model="gemini-2.5-flash",  # Replace with your desired model
    instruction="You are a helpful calculator and greeter. "
    "Use 'greet_user' for greetings. "
    "Use 'calculator_add_numbers' to add and 'calculator_subtract_numbers' to subtract. "
    "Announce the state of 'last_math_operation' if it's set.",
    tools=[greet_tool, math_toolset_instance],  # Individual tool  # Toolset instance
)
```

In this example:

- `SimpleMathToolset` implements `BaseToolset` and its `get_tools()` method returns `FunctionTool` instances for `add_numbers` and `subtract_numbers` . It also customizes their names using a prefix.
- The `calculator_agent` is configured with both an individual `greet_tool` and an instance of `SimpleMathToolset` .
- When `calculator_agent` is run, ADK will call `math_toolset_instance.get_tools()` . The agent's LLM will then have access to `greet_user` , `calculator_add_numbers` , and `calculator_subtract_numbers` to handle user requests.
- The `add_numbers` tool demonstrates writing to `tool_context.state` , and the agent's instruction mentions reading this state.
- The `close()` method is called to ensure any resources held by the toolset are released.

Toolsets offer a powerful way to organize, manage, and dynamically provide collections of tools to your ADK agents, leading to more modular, maintainable, and adaptable agentic applications.

-----------------

# Function tools

When pre-built ADK tools don't meet your requirements, you can create custom *function tools* . Building function tools allows you to create tailored functionality, such as connecting to proprietary databases or implementing unique algorithms.
For example, a function tool, `myfinancetool` , might be a function that calculates a specific financial metric. ADK also supports long running functions, so if that calculation takes a while, the agent can continue working on other tasks.

ADK offers several ways to create functions tools, each suited to different levels of complexity and control:

- Function Tools
- Long Running Function Tools
- Agents-as-a-Tool

## Function Tools

Transforming a Python function into a tool is a straightforward way to integrate custom logic into your agents. When you assign a function to an agent’s `tools` list, the framework automatically wraps it as a `FunctionTool` .

### How it Works

The ADK framework automatically inspects your Python function's signature—including its name, docstring, parameters, type hints, and default values—to generate a schema. This schema is what the LLM uses to understand the tool's purpose, when to use it, and what arguments it requires.

### Defining Function Signatures

A well-defined function signature is crucial for the LLM to use your tool correctly.

#### Parameters

You can define functions with required parameters, optional parameters, and variadic arguments. Here’s how each is handled:

##### Required Parameters

A parameter is considered **required** if it has a type hint but **no default value** . The LLM must provide a value for this argument when it calls the tool.

Example: Required Parameters
```
def get_weather(city: str, unit: str):
    """
    Retrieves the weather for a city in the specified unit.

    Args:
        city (str): The city name.
        unit (str): The temperature unit, either 'Celsius' or 'Fahrenheit'.
    """
    # ... function logic ...
    return {"status": "success", "report": f"Weather for {city} is sunny."}
```

In this example, both `city` and `unit` are mandatory. If the LLM tries to call `get_weather` without one of them, the ADK will return an error to the LLM, prompting it to correct the call.

##### Optional Parameters with Default Values

A parameter is considered **optional** if you provide a **default value** . This is the standard Python way to define optional arguments. The ADK correctly interprets these and does not list them in the `required` field of the tool schema sent to the LLM.

Example: Optional Parameter with Default Value
```
def search_flights(destination: str, departure_date: str, flexible_days: int = ):
    """
    Searches for flights.

    Args:
        destination (str): The destination city.
        departure_date (str): The desired departure date.
        flexible_days (int, optional): Number of flexible days for the search. Defaults to 0.
    """
    # ... function logic ...
    if flexible_days > 0:
        return {"status": "success", "report": f"Found flexible flights to {destination}."}
    return {"status": "success", "report": f"Found flights to {destination} on {departure_date}."}
```

Here, `flexible_days` is optional. The LLM can choose to provide it, but it's not required.

##### Optional Parameters with typing.Optional

You can also mark a parameter as optional using `typing.Optional[SomeType]` or the `| None` syntax (Python 3.10+). This signals that the parameter can be `None` . When combined with a default value of `None` , it behaves as a standard optional parameter.

Example: `typing.Optional`
```
from typing import Optional

def create_user_profile(username: str, bio: Optional[str] = None):
    """
    Creates a new user profile.

    Args:
        username (str): The user's unique username.
        bio (str, optional): A short biography for the user. Defaults to None.
    """
    # ... function logic ...
    if bio:
        return {"status": "success", "message": f"Profile for {username} created with a bio."}
    return {"status": "success", "message": f"Profile for {username} created."}
```

##### Variadic Parameters (*args and **kwargs)

While you can include `*args` (variable positional arguments) and `**kwargs` (variable keyword arguments) in your function signature for other purposes, they are **ignored by the ADK framework** when generating the tool schema for the LLM. The LLM will not be aware of them and cannot pass arguments to them. It's best to rely on explicitly defined parameters for all data you expect from the LLM.

#### Return Type

The preferred return type for a Function Tool is a **dictionary** in Python or **Map** in Java. This allows you to structure the response with key-value pairs, providing context and clarity to the LLM. If your function returns a type other than a dictionary, the framework automatically wraps it into a dictionary with a single key named **"result"** .

Strive to make your return values as descriptive as possible. *For example,* instead of returning a numeric error code, return a dictionary with an "error_message" key containing a human-readable explanation. **Remember that the LLM** , not a piece of code, needs to understand the result. As a best practice, include a "status" key in your return dictionary to indicate the overall outcome (e.g., "success", "error", "pending"), providing the LLM with a clear signal about the operation's state.

#### Docstrings

The docstring of your function serves as the tool's **description** and is sent to the LLM. Therefore, a well-written and comprehensive docstring is crucial for the LLM to understand how to use the tool effectively. Clearly explain the purpose of the function, the meaning of its parameters, and the expected return values.

### Passing Data Between Tools

When an agent calls multiple tools in a sequence, you might need to pass data from one tool to another. The recommended way to do this is by using the `temp:` prefix in the session state.

A tool can write data to a `temp:` variable, and a subsequent tool can read it. This data is only available for the current invocation and is discarded afterwards.

Shared Invocation Context

All tool calls within a single agent turn share the same `InvocationContext` . This means they also share the same temporary ( `temp:` ) state, which is how data can be passed between them.

### Example

Example

This tool is a python function which obtains the Stock price of a given Stock ticker/ symbol.

Note: You need to `pip install yfinance` library before using this tool.

```

from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

import yfinance as yf

APP_NAME = "stock_app"
USER_ID = "1234"
SESSION_ID = "session1234"

def get_stock_price(symbol: str):
    """
    Retrieves the current stock price for a given symbol.

    Args:
        symbol (str): The stock symbol (e.g., "AAPL", "GOOG").

    Returns:
        float: The current stock price, or None if an error occurs.
    """
    try:
        stock = yf.Ticker(symbol)
        historical_data = stock.history(period="1d")
        if not historical_data.empty:
            current_price = historical_data['Close'].iloc[-]
            return current_price
        else:
            return None
    except Exception as e:
        print(f"Error retrieving stock price for {symbol}: {e}")
        return None

stock_price_agent = Agent(
    model='gemini-2.5-flash',
    name='stock_agent',
    instruction= 'You are an agent who retrieves stock prices. If a ticker symbol is provided, fetch the current price. If only a company name is given, first perform a Google search to find the correct ticker symbol before retrieving the stock price. If the provided ticker symbol is invalid or data cannot be retrieved, inform the user that the stock price could not be found.',
    description='This agent specializes in retrieving real-time stock prices. Given a stock ticker symbol (e.g., AAPL, GOOG, MSFT) or the stock name, use the tools and reliable data sources to provide the most up-to-date price.',
    tools=[get_stock_price], # You can add Python functions directly to the tools list; they will be automatically wrapped as FunctionTools.
)

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=stock_price_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[0].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("stock price of GOOG")
```

The return value from this tool will be wrapped into a dictionary.

```
{"result": "$123"}
```

### Best Practices

While you have considerable flexibility in defining your function, remember that simplicity enhances usability for the LLM. Consider these guidelines:

- **Fewer Parameters are Better:** Minimize the number of parameters to reduce complexity.
- **Simple Data Types:** Favor primitive data types like `str` and `int` over custom classes whenever possible.
- **Meaningful Names:** The function's name and parameter names significantly influence how the LLM interprets and utilizes the tool. Choose names that clearly reflect the function's purpose and the meaning of its inputs. Avoid generic names like `do_stuff()` or `beAgent()` .
- **Build for Parallel Execution:** Improve function calling performance when multiple tools are run by building for asynchronous operation. For information on enabling parallel execution for tools, see Increase tool performance with parallel execution .

## Long Running Function Tools

This tool is designed to help you start and manage tasks that are handled outside the operation of your agent workflow, and require a significant amount of processing time, without blocking the agent's execution. This tool is a subclass of `FunctionTool` .

When using a `LongRunningFunctionTool` , your function can initiate the long-running operation and optionally return an **initial result** , such as a long-running operation id. Once a long running function tool is invoked the agent runner pauses the agent run and lets the agent client to decide whether to continue or wait until the long-running operation finishes. The agent client can query the progress of the long-running operation and send back an intermediate or final response. The agent can then continue with other tasks. An example is the human-in-the-loop scenario where the agent needs human approval before proceeding with a task.

Warning: Execution handling

Long Running Function Tools are designed to help you start and *manage* long running
tasks as part of your agent workflow, but ***not perform*** the actual, long task.
For tasks that require significant time to complete, you should implement a separate
server to do the task.

Tip: Parallel execution

Depending on the type of tool you are building, designing for asychronous
operation may be a better solution than creating a long running tool. For
more information, see Increase tool performance with parallel execution .

### How it Works

In Python, you wrap a function with `LongRunningFunctionTool` .  In Java, you pass a Method name to `LongRunningFunctionTool.create()` .

1. **Initiation:** When the LLM calls the tool, your function starts the long-running operation.
2. **Initial Updates:** Your function should optionally return an initial result (e.g. the long-running operaiton id). The ADK framework takes the result and sends it back to the LLM packaged within a `FunctionResponse` . This allows the LLM to inform the user (e.g., status, percentage complete, messages). And then the agent run is ended / paused.
3. **Continue or Wait:** After each agent run is completed. Agent client can query the progress of the long-running operation and decide whether to continue the agent run with an intermediate response (to update the progress) or wait until a final response is retrieved. Agent client should send the intermediate or final response back to the agent for the next run.
4. **Framework Handling:** The ADK framework manages the execution. It sends the intermediate or final `FunctionResponse` sent by agent client to the LLM to generate a user friendly message.

### Creating the Tool

Define your tool function and wrap it using the `LongRunningFunctionTool` class:

```
# 1. Define the long running function
def ask_for_approval(
    purpose: str, amount: float
) -> dict[str, Any]:
    """Ask for approval for the reimbursement."""
    # create a ticket for the approval
    # Send a notification to the approver with the link of the ticket
    return {'status': 'pending', 'approver': 'Sean Zhou', 'purpose' : purpose, 'amount': amount, 'ticket-id': 'approval-ticket-1'}

def reimburse(purpose: str, amount: float) -> str:
    """Reimburse the amount of money to the employee."""
    # send the reimbrusement request to payment vendor
    return {'status': 'ok'}

# 2. Wrap the function with LongRunningFunctionTool
long_running_tool = LongRunningFunctionTool(func=ask_for_approval)
```

### Intermediate / Final result Updates

Agent client received an event with long running function calls and check the status of the ticket. Then Agent client can send the intermediate or final response back to update the progress. The framework packages this value (even if it's None) into the content of the `FunctionResponse` sent back to the LLM.

Applies to only Java ADK
When passing `ToolContext` with Function Tools, ensure that one of the following is true:

- The Schema is passed with the ToolContext parameter in the function signature, like:

```
@com.google.adk.tools.Annotations.Schema(name = "toolContext") ToolContext toolContext
```

OR

- The following `-parameters` flag is set to the mvn compiler plugin

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.14.0</version> <!-- or newer -->
            <configuration>
                <compilerArgs>
                    <arg>-parameters</arg>
                </compilerArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

This constraint is temporary and will be removed.

```
# Agent Interaction
async def call_agent_async(query):

    def get_long_running_function_call(event: Event) -> types.FunctionCall:
        # Get the long running function call from the event
        if not event.long_running_tool_ids or not event.content or not event.content.parts:
            return
        for part in event.content.parts:
            if (
                part
                and part.function_call
                and event.long_running_tool_ids
                and part.function_call.id in event.long_running_tool_ids
            ):
                return part.function_call

    def get_function_response(event: Event, function_call_id: str) -> types.FunctionResponse:
        # Get the function response for the fuction call with specified id.
        if not event.content or not event.content.parts:
            return
        for part in event.content.parts:
            if (
                part
                and part.function_response
                and part.function_response.id == function_call_id
            ):
                return part.function_response

    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()

    print("\nRunning agent...")
    events_async = runner.run_async(
        session_id=session.id, user_id=USER_ID, new_message=content
    )

    long_running_function_call, long_running_function_response, ticket_id = None, None, None
    async for event in events_async:
        # Use helper to check for the specific auth request event
        if not long_running_function_call:
            long_running_function_call = get_long_running_function_call(event)
        else:
            _potential_response = get_function_response(event, long_running_function_call.id)
            if _potential_response: # Only update if we get a non-None response
                long_running_function_response = _potential_response
                ticket_id = long_running_function_response.response['ticket-id']
        if event.content and event.content.parts:
            if text := ''.join(part.text or '' for part in event.content.parts):
                print(f'[{event.author}]: {text}')

    if long_running_function_response:
        # query the status of the correpsonding ticket via tciket_id
        # send back an intermediate / final response
        updated_response = long_running_function_response.model_copy(deep=True)
        updated_response.response = {'status': 'approved'}
        async for event in runner.run_async(
          session_id=session.id, user_id=USER_ID, new_message=types.Content(parts=[types.Part(function_response = updated_response)], role='user')
        ):
            if event.content and event.content.parts:
                if text := ''.join(part.text or '' for part in event.content.parts):
                    print(f'[{event.author}]: {text}')
```

Python complete example: File Processing Simulation
```
import asyncio
from typing import Any
from google.adk.agents import Agent
from google.adk.events import Event
from google.adk.runners import Runner
from google.adk.tools import LongRunningFunctionTool
from google.adk.sessions import InMemorySessionService
from google.genai import types

# 1. Define the long running function
def ask_for_approval(
    purpose: str, amount: float
) -> dict[str, Any]:
    """Ask for approval for the reimbursement."""
    # create a ticket for the approval
    # Send a notification to the approver with the link of the ticket
    return {'status': 'pending', 'approver': 'Sean Zhou', 'purpose' : purpose, 'amount': amount, 'ticket-id': 'approval-ticket-1'}

def reimburse(purpose: str, amount: float) -> str:
    """Reimburse the amount of money to the employee."""
    # send the reimbrusement request to payment vendor
    return {'status': 'ok'}

# 2. Wrap the function with LongRunningFunctionTool
long_running_tool = LongRunningFunctionTool(func=ask_for_approval)

# 3. Use the tool in an Agent
file_processor_agent = Agent(
    # Use a model compatible with function calling
    model="gemini-2.5-flash",
    name='reimbursement_agent',
    instruction="""
      You are an agent whose job is to handle the reimbursement process for
      the employees. If the amount is less than $100, you will automatically
      approve the reimbursement.

      If the amount is greater than $100, you will
      ask for approval from the manager. If the manager approves, you will
      call reimburse() to reimburse the amount to the employee. If the manager
      rejects, you will inform the employee of the rejection.
    """,
    tools=[reimburse, long_running_tool]
)

APP_NAME = "human_in_the_loop"
USER_ID = "1234"
SESSION_ID = "session1234"

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=file_processor_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):

    def get_long_running_function_call(event: Event) -> types.FunctionCall:
        # Get the long running function call from the event
        if not event.long_running_tool_ids or not event.content or not event.content.parts:
            return
        for part in event.content.parts:
            if (
                part
                and part.function_call
                and event.long_running_tool_ids
                and part.function_call.id in event.long_running_tool_ids
            ):
                return part.function_call

    def get_function_response(event: Event, function_call_id: str) -> types.FunctionResponse:
        # Get the function response for the fuction call with specified id.
        if not event.content or not event.content.parts:
            return
        for part in event.content.parts:
            if (
                part
                and part.function_response
                and part.function_response.id == function_call_id
            ):
                return part.function_response

    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()

    print("\nRunning agent...")
    events_async = runner.run_async(
        session_id=session.id, user_id=USER_ID, new_message=content
    )

    long_running_function_call, long_running_function_response, ticket_id = None, None, None
    async for event in events_async:
        # Use helper to check for the specific auth request event
        if not long_running_function_call:
            long_running_function_call = get_long_running_function_call(event)
        else:
            _potential_response = get_function_response(event, long_running_function_call.id)
            if _potential_response: # Only update if we get a non-None response
                long_running_function_response = _potential_response
                ticket_id = long_running_function_response.response['ticket-id']
        if event.content and event.content.parts:
            if text := ''.join(part.text or '' for part in event.content.parts):
                print(f'[{event.author}]: {text}')

    if long_running_function_response:
        # query the status of the correpsonding ticket via tciket_id
        # send back an intermediate / final response
        updated_response = long_running_function_response.model_copy(deep=True)
        updated_response.response = {'status': 'approved'}
        async for event in runner.run_async(
          session_id=session.id, user_id=USER_ID, new_message=types.Content(parts=[types.Part(function_response = updated_response)], role='user')
        ):
            if event.content and event.content.parts:
                if text := ''.join(part.text or '' for part in event.content.parts):
                    print(f'[{event.author}]: {text}')

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.

# reimbursement that doesn't require approval
# asyncio.run(call_agent_async("Please reimburse 50$ for meals"))
await call_agent_async("Please reimburse 50$ for meals") # For Notebooks, uncomment this line and comment the above line
# reimbursement that requires approval
# asyncio.run(call_agent_async("Please reimburse 200$ for meals"))
await call_agent_async("Please reimburse 200$ for meals") # For Notebooks, uncomment this line and comment the above line
```

#### Key aspects of this example

- **`LongRunningFunctionTool`** : Wraps the supplied method/function; the framework handles sending yielded updates and the final return value as sequential FunctionResponses.
- **Agent instruction** : Directs the LLM to use the tool and understand the incoming FunctionResponse stream (progress vs. completion) for user updates.
- **Final return** : The function returns the final result dictionary, which is sent in the concluding FunctionResponse to indicate completion.

## Agent-as-a-Tool

This powerful feature allows you to leverage the capabilities of other agents within your system by calling them as tools. The Agent-as-a-Tool enables you to invoke another agent to perform a specific task, effectively **delegating responsibility** . This is conceptually similar to creating a Python function that calls another agent and uses the agent's response as the function's return value.

### Key difference from sub-agents

It's important to distinguish an Agent-as-a-Tool from a Sub-Agent.

- **Agent-as-a-Tool:** When Agent A calls Agent B as a tool (using Agent-as-a-Tool), Agent B's answer is **passed back** to Agent A, which then summarizes the answer and generates a response to the user. Agent A retains control and continues to handle future user input.
- **Sub-agent:** When Agent A calls Agent B as a sub-agent, the responsibility of answering the user is completely **transferred to Agent B** . Agent A is effectively out of the loop. All subsequent user input will be answered by Agent B.

### Usage

To use an agent as a tool, wrap the agent with the AgentTool class.

```
tools=[AgentTool(agent=agent_b)]
```

### Customization

The `AgentTool` class provides the following attributes for customizing its behavior:

- **skip_summarization: bool:** If set to True, the framework will **bypass the LLM-based summarization** of the tool agent's response. This can be useful when the tool's response is already well-formatted and requires no further processing.
Example
```

from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools.agent_tool import AgentTool
from google.genai import types

APP_NAME="summary_agent"
USER_ID="user1234"
SESSION_ID="1234"

summary_agent = Agent(
    model="gemini-2.5-flash",
    name="summary_agent",
    instruction="""You are an expert summarizer. Please read the following text and provide a concise summary.""",
    description="Agent to summarize text",
)

root_agent = Agent(
    model='gemini-2.5-flash',
    name='root_agent',
    instruction="""You are a helpful assistant. When the user provides a text, use the 'summarize' tool to generate a summary. Always forward the user's message exactly as received to the 'summarize' tool, without modifying or summarizing it yourself. Present the response from the tool to the user.""",
    tools=[AgentTool(agent=summary_agent, skip_summarization=True)]
)

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=root_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[].text
            print("Agent Response: ", final_response)

long_text = """Quantum computing represents a fundamentally different approach to computation,
leveraging the bizarre principles of quantum mechanics to process information. Unlike classical computers
that rely on bits representing either 0 or 1, quantum computers use qubits which can exist in a state of superposition - effectively
being 0, 1, or a combination of both simultaneously. Furthermore, qubits can become entangled,
meaning their fates are intertwined regardless of distance, allowing for complex correlations. This parallelism and
interconnectedness grant quantum computers the potential to solve specific types of incredibly complex problems - such
as drug discovery, materials science, complex system optimization, and breaking certain types of cryptography - far
faster than even the most powerful classical supercomputers could ever achieve, although the technology is still largely in its developmental stages."""

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async(long_text)
```

### How it works

1. When the `main_agent` receives the long text, its instruction tells it to use the 'summarize' tool for long texts.
2. The framework recognizes 'summarize' as an `AgentTool` that wraps the `summary_agent` .
3. Behind the scenes, the `main_agent` will call the `summary_agent` with the long text as input.
4. The `summary_agent` will process the text according to its instruction and generate a summary.
5. **The response from the `summary_agent` is then passed back to the `main_agent` .**
6. The `main_agent` can then take the summary and formulate its final response to the user (e.g., "Here's a summary of the text: ...")

-----------------

# Increase tool performance with parallel execution

Starting with Agent Development Kit (ADK) version 1.10.0, the framework
attempts to run any agent-requested function tools in parallel. This behavior can significantly improve the performance and
responsiveness of your agents, particularly for agents that rely on multiple
external APIs or long-running tasks. For example, if you have 3 tools that each
take 2 seconds, by running them in parallel, the total execution time will be
closer to 2 seconds, instead of 6 seconds. The ability to run tool functions
parallel can improve the performance of your agents, particularly in the
following scenarios:

- **Research tasks:** Where the agent collects information from multiple
    sources before proceeding to the next stage of the workflow.

- **API calls:** Where the agent accesses several APIs independently, such
    as searching for available flights using APIs from multiple airlines.

- **Publishing and communication tasks:** When the agent needs to publish
    or communicate through multiple, independent channels or multiple recipients.

However, your custom tools must be built with asynchronous execution support to
enable this performance improvement. This guide explains how parallel tool
execution works in the ADK and how to build your tools to take full advantage of
this processing feature.

Warning

Any ADK Tools that use synchronous processing in a set of tool function
calls will block other tools from executing in parallel, even if the other
tools allow for parallel execution.

## Build parallel-ready tools

Enable parallel execution of your tool functions by defining them as
asynchronous functions. In Python code, this means using `async def` and `await` syntax which allows the ADK to run them concurrently in an `asyncio` event loop.
The following sections show examples of agent tools built for parallel
processing and asynchronous operations.

### Example of http web call

The following code example show how to modify the `get_weather()` function to
operate asynchronously and allow for parallel execution:

```
async def get_weather(city: str) -> dict:
      async with aiohttp.ClientSession() as session:
          async with session.get(f"http://api.weather.com/{city}") as response:
              return await response.json()
```

### Example of database call

The following code example show how to write a database calling function to
operate asynchronously:

```
async def query_database(query: str) -> list:
      async with asyncpg.connect("postgresql://...") as conn:
          return await conn.fetch(query)
```

### Example of yielding behavior for long loops

In cases where a tool is processing multiple requests or numerous long running
requests, consider adding yielding code to allow other tools to execute, as
shown in the following code sample:

```
async def process_data(data: list) -> dict:
      results = []
      for i, item in enumerate(data):
          processed = await process_item(item)  # Yield point
          results.append(processed)

          # Add periodic yield points for long loops
          if i %  == 0:
              await asyncio.sleep(0)  # Yield control
      return {"results": results}
```

Important

Use the `asyncio.sleep()` function for pauses to avoid blocking
execution of other functions.

### Example of thread pools for intensive operations

When performing processing-intensive functions, consider creating thread pools
for better management of available computing resources, as shown in the
following example:

```
async def cpu_intensive_tool(data: list) -> dict:
      loop = asyncio.get_event_loop()

      # Use thread pool for CPU-bound work
      with ThreadPoolExecutor() as executor:
          result = await loop.run_in_executor(
              executor,
              expensive_computation,
              data
          )
      return {"result": result}
```

### Example of process chunking

When performing processes on long lists or large amounts of data, consider
combining a thread pool technique with dividing up processing into chunks of
data, and yielding processing time between the chunks, as shown in the following
example:

```
async def process_large_dataset(dataset: list) -> dict:
      results = []
      chunk_size =

      for i in range(0, len(dataset), chunk_size):
          chunk = dataset[i:i + chunk_size]

          # Process chunk in thread pool
          loop = asyncio.get_event_loop()
          with ThreadPoolExecutor() as executor:
              chunk_result = await loop.run_in_executor(
                  executor, process_chunk, chunk
              )

          results.extend(chunk_result)

          # Yield control between chunks
          await asyncio.sleep(0)

      return {"total_processed": len(results), "results": results}
```

## Write parallel-ready prompts and tool descriptions

When building prompts for AI models, consider explicitly specifying or hinting
that function calls be made in parallel. The following example of an AI prompt
directs the model to use tools in parallel:

```
When users ask for multiple pieces of information, always call functions in
parallel.

  Examples:
  - "Get weather for London and currency rate USD to EUR" → Call both functions
    simultaneously
  - "Compare cities A and B" → Call get_weather, get_population, get_distance in
    parallel
  - "Analyze multiple stocks" → Call get_stock_price for each stock in parallel

  Always prefer multiple specific function calls over single complex calls.
```

The following example shows a tool function description that hints at more
efficient use through parallel execution:

```
async def get_weather(city: str) -> dict:
      """Get current weather for a single city.

      This function is optimized for parallel execution - call multiple times for different cities.

      Args:
          city: Name of the city, for example: 'London', 'New York'

      Returns:
          Weather data including temperature, conditions, humidity
      """
      await asyncio.sleep()  # Simulate API call
      return {"city": city, "temp": 72, "condition": "sunny"}
```

## Next steps

For more information on building Tools for agents and function calling, see Function Tools . For
more detailed examples of tools that take advantage of parallel processing, see
the samples in the adk-python repository.

-----------------

# Get action confirmation for ADK Tools

Some agent workflows require confirmation for decision making, verification,
security, or general oversight. In these cases, you want to get a response from
a human or supervising system before proceeding with a workflow. The *Tool
Confirmation* feature in the Agent Development Kit (ADK) allows an ADK Tool to
pause its execution and interact with a user or other system for confirmation or
to gather structured data before proceeding. You can use Tool Confirmation with
an ADK Tool in the following ways:

- **Boolean Confirmation :** You can
    configure a FunctionTool with a `require_confirmation` parameter. This
    option pauses the tool for a yes or no confirmation response.

- **Advanced Confirmation :** For scenarios requiring
    structured data responses, you can configure a `FunctionTool` with a text
    prompt to explain the confirmation and an expected response.

Experimental

The Tool Confirmation feature is experimental and has some known limitations .
We welcome your feedback !

You can configure how a request is communicated to a user, and the system can
also use remote responses sent via the ADK
server's REST API. When using the confirmation feature with the ADK web user
interface, the agent workflow displays a dialog box to the user to request
input, as shown in Figure 1:

**Figure 1.** Example confirmation response request dialog box using an
advanced, tool response implementation.

The following sections describe how to use this feature for the confirmation
scenarios. For a complete code sample, see the human_tool_confirmation example. There are additional ways to incorporate human input into your agent
workflow, for more details, see the Human-in-the-loop agent pattern.

## Boolean confirmation

When your tool only requires a simple `yes` or `no` from the user, you can
append a confirmation step using the `FunctionTool` class as a wrapper. For
example, if you have a tool called `reimburse` , you can enable a confirmation
step by wrapping it with the `FunctionTool` class and setting the `require_confirmation` parameter to `True` , as shown in the following example:

```
# From agent.py
root_agent = Agent(
   ...
   tools=[
        # Set require_confirmation to True to require user confirmation
        # for the tool call.
        FunctionTool(reimburse, require_confirmation=True),
    ],
...
```

This implementation method requires minimal code, but is limited to simple
approvals from the user or confirming system. For a complete example of this
approach, see the human_tool_confirmation code sample.

### Require confirmation function

You can modify the behavior `require_confirmation` response by replacing its
input value with a function that returns a boolean response. The following
example shows a function for determining if a confirmation is required:

```
async def confirmation_threshold(
    amount: int, tool_context: ToolContext
) -> bool:
  """Returns true if the amount is greater than 1000."""
  return amount > 1000
```

This function than then be set as the parameter value for the `require_confirmation` parameter:

```
root_agent = Agent(
   ...
   tools=[
        # Set require_confirmation to True to require user confirmation
        FunctionTool(reimburse, require_confirmation=confirmation_threshold),
    ],
...
```

For a complete example of this implementation, see the human_tool_confirmation code sample.

## Advanced confirmation

When a tool confirmation requires more details for the user or a more complex
response, use a tool_confirmation implementation. This approach extends the `ToolContext` object to add a text description of the request for the user and
allows for more complex response data. When implementing tool confirmation this
way, you can pause a tool's execution, request specific information, and then
resume the tool with the provided data.

This confirmation flow has a request stage where the system assembles and sends
an input request human response, and a response stage where the system receives
and processes the returned data.

### Confirmation definition

When creating a Tool with an advanced confirmation, create a function that
includes a ToolContext object. Then define the confirmation using a
tool_confirmation object, the `tool_context.request_confirmation()` method with `hint` and `payload` parameters. These properties are used as follows:

- `hint` : Descriptive message that explains what is needed from the user.
- `payload` : The structure of the data you expect in return. This data
    type is Any and must be serializable into a JSON-formatted string, such as
    a dictionary or pydantic model.

The following code shows an example implementation for a tool that processes
time off requests for an employee:

```
def request_time_off(days: int, tool_context: ToolContext):
  """Request day off for the employee."""
  ...
  tool_confirmation = tool_context.tool_confirmation
  if not tool_confirmation:
    tool_context.request_confirmation(
        hint=(
            'Please approve or reject the tool call request_time_off() by'
            ' responding with a FunctionResponse with an expected'
            ' ToolConfirmation payload.'
        ),
        payload={
            'approved_days': 0,
        },
    )
    # Return intermediate status indicating that the tool is waiting for
    # a confirmation response:
    return {'status': 'Manager approval is required.'}

  approved_days = tool_confirmation.payload['approved_days']
  approved_days = min(approved_days, days)
  if approved_days == 0:
    return {'status': 'The time off request is rejected.', 'approved_days': 0}
  return {
      'status': 'ok',
      'approved_days': approved_days,
  }
```

For a complete example of this approach, see the human_tool_confirmation code sample. Keep in mind that the agent workflow tool execution pauses while a
confirmation is obtained. After confirmation is received, you can access the
confirmation response in the `tool_confirmation.payload` object and then proceed
with the execution of the workflow.

## Remote confirmation with REST API

If there is no active user interface for a human confirmation of an agent
workflow, you can handle the confirmation through a command-line interface or by
routing it through another channel like email or a chat application. To confirm
the tool call, the user or calling application needs to send a `FunctionResponse` event with the tool confirmation data.

You can send the request to the ADK API server's `/run` or `/run_sse` endpoint,
or directly to the ADK runner. The following example uses a `curl` command to
send the confirmation to the `/run_sse` endpoint:

```
curl -X POST http://localhost:8000/run_sse \
 -H "Content-Type: application/json" \
 -d '{
    "app_name": "human_tool_confirmation",
    "user_id": "user",
    "session_id": "7828f575-2402-489f-8079-74ea95b6a300",
    "new_message": {
        "parts": [
            {
                "function_response": {
                    "id": "adk-13b84a8c-c95c-4d66-b006-d72b30447e35",
                    "name": "adk_request_confirmation",
                    "response": {
                        "confirmed": true
                    }
                }
            }
        ],
        "role": "user"
    }
}'
```

A REST-based response for a confirmation must meet the following
requirements:

- The `id` in the `function_response` should match the `function_call_id` from the `RequestConfirmation`  `FunctionCall` event.
- The `name` should be `adk_request_confirmation` .
- The `response` object contains the confirmation status and any
    additional payload data required by the tool.

## Known limitations

The tool confirmation feature has the following limitations:

- DatabaseSessionService is not supported by this feature.
- VertexAiSessionService is not supported by this feature.

-----------------

# Built-in tools

These built-in tools provide ready-to-use functionality such as Google Search or
code executors that provide agents with common capabilities. For instance, an
agent that needs to retrieve information from the web can directly use the **google_search** tool without any additional setup.

## How to Use

1. **Import:** Import the desired tool from the tools module. This is `agents.tools` in Python or `com.google.adk.tools` in Java.
2. **Configure:** Initialize the tool, providing required parameters if any.
3. **Register:** Add the initialized tool to the **tools** list of your Agent.

Once added to an agent, the agent can decide to use the tool based on the **user
prompt** and its **instructions** . The framework handles the execution of the
tool when the agent calls it. Important: check the ***Limitations*** section of this page.

## Available Built-in tools

Note: Java only supports Google Search and Code Execution tools currently.

### Google Search

The `google_search` tool allows the agent to perform web searches using Google Search. The `google_search` tool is only compatible with Gemini 2 models. For further details of the tool, see Understanding Google Search grounding .

Additional requirements when using the `google_search` tool

When you use grounding with Google Search, and you receive Search suggestions in your response, you must display the Search suggestions in production and in your applications.
For more information on grounding with Google Search, see Grounding with Google Search documentation for Google AI Studio or Vertex AI . The UI code (HTML) is returned in the Gemini response as `renderedContent` , and you will need to show the HTML in your app, in accordance with the policy.

```

from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.genai import types

APP_NAME="google_search_agent"
USER_ID="user1234"
SESSION_ID="1234"

root_agent = Agent(
    name="basic_search_agent",
    model="gemini-2.5-flash",
    description="Agent to answer questions using Google Search.",
    instruction="I can answer your questions by searching the internet. Just ask me anything!",
    # google_search is a pre-built tool which allows the agent to perform Google searches.
    tools=[google_search]
)

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=root_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("what's the latest ai news?")
```

### Code Execution

The `built_in_code_execution` tool enables the agent to execute code,
specifically when using Gemini 2 models. This allows the model to perform tasks
like calculations, data manipulation, or running small scripts.

```

import asyncio
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.code_executors import BuiltInCodeExecutor
from google.genai import types

AGENT_NAME = "calculator_agent"
APP_NAME = "calculator"
USER_ID = "user1234"
SESSION_ID = "session_code_exec_async"
GEMINI_MODEL = "gemini-2.5-flash"

# Agent Definition
code_agent = LlmAgent(
    name=AGENT_NAME,
    model=GEMINI_MODEL,
    code_executor=BuiltInCodeExecutor(),
    instruction="""You are a calculator agent.
    When given a mathematical expression, write and execute Python code to calculate the result.
    Return only the final numerical result as plain text, without markdown or code blocks.
    """,
    description="Executes Python code to perform calculations.",
)

# Session and Runner
session_service = InMemorySessionService()
session = asyncio.run(session_service.create_session(
    app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID
))
runner = Runner(agent=code_agent, app_name=APP_NAME,
                session_service=session_service)

# Agent Interaction (Async)
async def call_agent_async(query):
    content = types.Content(role="user", parts=[types.Part(text=query)])
    print(f"\n--- Running Query: {query} ---")
    final_response_text = "No final text response captured."
    try:
        # Use run_async
        async for event in runner.run_async(
            user_id=USER_ID, session_id=SESSION_ID, new_message=content
        ):
            print(f"Event ID: {event.id}, Author: {event.author}")

            # --- Check for specific parts FIRST ---
            has_specific_part = False
            if event.content and event.content.parts:
                for part in event.content.parts:  # Iterate through all parts
                    if part.executable_code:
                        # Access the actual code string via .code
                        print(
                            f"  Debug: Agent generated code:\n
```python\n{part.executable_code.code}\n
```"
                        )
                        has_specific_part = True
                    elif part.code_execution_result:
                        # Access outcome and output correctly
                        print(
                            f"  Debug: Code Execution Result: {part.code_execution_result.outcome} - Output:\n{part.code_execution_result.output}"
                        )
                        has_specific_part = True
                    # Also print any text parts found in any event for debugging
                    elif part.text and not part.text.isspace():
                        print(f"  Text: '{part.text.strip()}'")
                        # Do not set has_specific_part=True here, as we want the final response logic below

            # --- Check for final response AFTER specific parts ---
            # Only consider it final if it doesn't have the specific code parts we just handled
            if not has_specific_part and event.is_final_response():
                if (
                    event.content
                    and event.content.parts
                    and event.content.parts[].text
                ):
                    final_response_text = event.content.parts[0].text.strip()
                    print(f"==> Final Agent Response: {final_response_text}")
                else:
                    print(
                        "==> Final Agent Response: [No text content in final event]")

    except Exception as e:
        print(f"ERROR during agent run: {e}")
    print("-" * 30)

# Main async function to run the examples
async def main():
    await call_agent_async("Calculate the value of (5 + 7) * 3")
    await call_agent_async("What is 10 factorial?")

# Execute the main async function
try:
    asyncio.run(main())
except RuntimeError as e:
    # Handle specific error when running asyncio.run in an already running loop (like Jupyter/Colab)
    if "cannot be called from a running event loop" in str(e):
        print("\nRunning in an existing event loop (like Colab/Jupyter).")
        print("Please run `await main()` in a notebook cell instead.")
        # If in an interactive environment like a notebook, you might need to run:
        # await main()
    else:
        raise e  # Re-raise other runtime errors
```

## Use Built-in tools with other tools

The following code sample demonstrates how to use multiple built-in tools or how
to use built-in tools with other tools by using multiple agents:

```
from google.adk.tools.agent_tool import AgentTool
from google.adk.agents import Agent
from google.adk.tools import google_search
from google.adk.code_executors import BuiltInCodeExecutor

search_agent = Agent(
    model='gemini-2.5-flash',
    name='SearchAgent',
    instruction="""
    You're a specialist in Google Search
    """,
    tools=[google_search],
)
coding_agent = Agent(
    model='gemini-2.5-flash',
    name='CodeAgent',
    instruction="""
    You're a specialist in Code Execution
    """,
    code_executor=BuiltInCodeExecutor(),
)
root_agent = Agent(
    name="RootAgent",
    model="gemini-2.5-flash",
    description="Root Agent",
    tools=[AgentTool(agent=search_agent), AgentTool(agent=coding_agent)],
)
```

### Limitations

Warning

Currently, for each root agent or single agent, only one built-in tool is
supported. No other tools of any type can be used in the same agent.

For example, the following approach that uses ***a built-in tool along with
 other tools*** within a single agent is **not** currently supported:

```
root_agent = Agent(
    name="RootAgent",
    model="gemini-2.5-flash",
    description="Root Agent",
    tools=[custom_function],
    code_executor=BuiltInCodeExecutor() # <-- not supported when used with tools
)
```

Warning

Built-in tools cannot be used within a sub-agent.

For example, the following approach that uses built-in tools within sub-agents
is **not** currently supported:

```
search_agent = Agent(
    model='gemini-2.5-flash',
    name='SearchAgent',
    instruction="""
    You're a specialist in Google Search
    """,
    tools=[google_search],
)
coding_agent = Agent(
    model='gemini-2.5-flash',
    name='CodeAgent',
    instruction="""
    You're a specialist in Code Execution
    """,
    code_executor=BuiltInCodeExecutor(),
)
root_agent = Agent(
    name="RootAgent",
    model="gemini-2.5-flash",
    description="Root Agent",
    sub_agents=[
        search_agent,
        coding_agent
    ],
)
```

-----------------

# Model Context Protocol Tools

This guide walks you through two ways of integrating Model Context Protocol (MCP) with ADK.

## What is Model Context Protocol (MCP)?

The Model Context Protocol (MCP) is an open standard designed to standardize how Large Language Models (LLMs) like Gemini and Claude communicate with external applications, data sources, and tools. Think of it as a universal connection mechanism that simplifies how LLMs obtain context, execute actions, and interact with various systems.

MCP follows a client-server architecture, defining how **data** (resources), **interactive templates** (prompts), and **actionable functions** (tools) are exposed by an **MCP server** and consumed by an **MCP client** (which could be an LLM host application or an AI agent).

This guide covers two primary integration patterns:

1. **Using Existing MCP Servers within ADK:** An ADK agent acts as an MCP client, leveraging tools provided by external MCP servers.
2. **Exposing ADK Tools via an MCP Server:** Building an MCP server that wraps ADK tools, making them accessible to any MCP client.

## Prerequisites

Before you begin, ensure you have the following set up:

- **Set up ADK:** Follow the standard ADK setup instructions in the quickstart.
- **Install/update Python/Java:** MCP requires Python version of 3.9 or higher for Python or Java 17 or higher.
- **Setup Node.js and npx:**  **(Python only)** Many community MCP servers are distributed as Node.js packages and run using `npx` . Install Node.js (which includes npx) if you haven't already. For details, see https://nodejs.org/en .
- **Verify Installations:**  **(Python only)** Confirm `adk` and `npx` are in your PATH within the activated virtual environment:
```
# Both commands should print the path to the executables.
which adk
which npx
```

## 1. Using MCP servers with ADK agents (ADK as an MCP client) in adk web

This section demonstrates how to integrate tools from external MCP (Model Context Protocol) servers into your ADK agents. This is the **most common** integration pattern when your ADK agent needs to use capabilities provided by an existing service that exposes an MCP interface. You will see how the `MCPToolset` class can be directly added to your agent's `tools` list, enabling seamless connection to an MCP server, discovery of its tools, and making them available for your agent to use. These examples primarily focus on interactions within the `adk web` development environment.

### MCPToolset class

The `MCPToolset` class is ADK's primary mechanism for integrating tools from an MCP server. When you include an `MCPToolset` instance in your agent's `tools` list, it automatically handles the interaction with the specified MCP server. Here's how it works:

1. **Connection Management:** On initialization, `MCPToolset` establishes and manages the connection to the MCP server. This can be a local server process (using `StdioConnectionParams` for communication over standard input/output) or a remote server (using `SseConnectionParams` for Server-Sent Events). The toolset also handles the graceful shutdown of this connection when the agent or application terminates.
2. **Tool Discovery & Adaptation:** Once connected, `MCPToolset` queries the MCP server for its available tools (via the `list_tools` MCP method). It then converts the schemas of these discovered MCP tools into ADK-compatible `BaseTool` instances.
3. **Exposure to Agent:** These adapted tools are then made available to your `LlmAgent` as if they were native ADK tools.
4. **Proxying Tool Calls:** When your `LlmAgent` decides to use one of these tools, `MCPToolset` transparently proxies the call (using the `call_tool` MCP method) to the MCP server, sends the necessary arguments, and returns the server's response back to the agent.
5. **Filtering (Optional):** You can use the `tool_filter` parameter when creating an `MCPToolset` to select a specific subset of tools from the MCP server, rather than exposing all of them to your agent.

The following examples demonstrate how to use `MCPToolset` within the `adk web` development environment. For scenarios where you need more fine-grained control over the MCP connection lifecycle or are not using `adk web` , refer to the "Using MCP Tools in your own Agent out of `adk web` " section later in this page.

### Example 1: File System MCP Server

This Python example demonstrates connecting to a local MCP server that provides file system operations.

#### Step 1: Define your Agent with MCPToolset

Create an `agent.py` file (e.g., in `./adk_agent_samples/mcp_agent/agent.py` ). The `MCPToolset` is instantiated directly within the `tools` list of your `LlmAgent` .

- **Important:** Replace `"/path/to/your/folder"` in the `args` list with the **absolute path** to an actual folder on your local system that the MCP server can access.
- **Important:** Place the `.env` file in the parent directory of the `./adk_agent_samples` directory.
```
# ./adk_agent_samples/mcp_agent/agent.py
import os # Required for path operations
from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# It's good practice to define paths dynamically if possible,
# or ensure the user understands the need for an ABSOLUTE path.
# For this example, we'll construct a path relative to this file,
# assuming '/path/to/your/folder' is in the same directory as agent.py.
# REPLACE THIS with an actual absolute path if needed for your setup.
TARGET_FOLDER_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), "/path/to/your/folder")
# Ensure TARGET_FOLDER_PATH is an absolute path for the MCP server.
# If you created ./adk_agent_samples/mcp_agent/your_folder,

root_agent = LlmAgent(
    model='gemini-2.5-flash',
    name='filesystem_assistant_agent',
    instruction='Help the user manage their files. You can list files, read files, etc.',
    tools=[
        MCPToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='npx',
                    args=[
                        "-y",  # Argument for npx to auto-confirm install
                        "@modelcontextprotocol/server-filesystem",
                        # IMPORTANT: This MUST be an ABSOLUTE path to a folder the
                        # npx process can access.
                        # Replace with a valid absolute path on your system.
                        # For example: "/Users/youruser/accessible_mcp_files"
                        # or use a dynamically constructed absolute path:
                        os.path.abspath(TARGET_FOLDER_PATH),
                    ],
                ),
            ),
            # Optional: Filter which tools from the MCP server are exposed
            # tool_filter=['list_directory', 'read_file']
        )
    ],
)
```

#### Step 2: Create an __init__.py file

Ensure you have an `__init__.py` in the same directory as `agent.py` to make it a discoverable Python package for ADK.

```
# ./adk_agent_samples/mcp_agent/__init__.py
from . import agent
```

#### Step 3: Run adk web and Interact

Navigate to the parent directory of `mcp_agent` (e.g., `adk_agent_samples` ) in your terminal and run:

```
cd ./adk_agent_samples # Or your equivalent parent directory
adk web
```

Note for Windows users

When hitting the `_make_subprocess_transport NotImplementedError` , consider using `adk web --no-reload` instead.

Once the ADK Web UI loads in your browser:

1. Select the `filesystem_assistant_agent` from the agent dropdown.
2. Try prompts like:
- "List files in the current directory."
- "Can you read the file named sample.txt?" (assuming you created it in `TARGET_FOLDER_PATH` ).
- "What is the content of `another_file.md` ?"

You should see the agent interacting with the MCP file system server, and the server's responses (file listings, file content) relayed through the agent. The `adk web` console (terminal where you ran the command) might also show logs from the `npx` process if it outputs to stderr.


### Example 2: Google Maps MCP Server

This example demonstrates connecting to the Google Maps MCP server.

#### Step 1: Get API Key and Enable APIs

1. **Google Maps API Key:** Follow the directions at Use API keys to obtain a Google Maps API Key.
2. **Enable APIs:** In your Google Cloud project, ensure the following APIs are enabled:
- Directions API
- Routes API
For instructions, see the Getting started with Google Maps Platform documentation.

#### Step 2: Define your Agent with MCPToolset for Google Maps

Modify your `agent.py` file (e.g., in `./adk_agent_samples/mcp_agent/agent.py` ). Replace `YOUR_GOOGLE_MAPS_API_KEY` with the actual API key you obtained.

```
# ./adk_agent_samples/mcp_agent/agent.py
import os
from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# Retrieve the API key from an environment variable or directly insert it.
# Using an environment variable is generally safer.
# Ensure this environment variable is set in the terminal where you run 'adk web'.
# Example: export GOOGLE_MAPS_API_KEY="YOUR_ACTUAL_KEY"
google_maps_api_key = os.environ.get("GOOGLE_MAPS_API_KEY")

if not google_maps_api_key:
    # Fallback or direct assignment for testing - NOT RECOMMENDED FOR PRODUCTION
    google_maps_api_key = "YOUR_GOOGLE_MAPS_API_KEY_HERE" # Replace if not using env var
    if google_maps_api_key == "YOUR_GOOGLE_MAPS_API_KEY_HERE":
        print("WARNING: GOOGLE_MAPS_API_KEY is not set. Please set it as an environment variable or in the script.")
        # You might want to raise an error or exit if the key is crucial and not found.

root_agent = LlmAgent(
    model='gemini-2.5-flash',
    name='maps_assistant_agent',
    instruction='Help the user with mapping, directions, and finding places using Google Maps tools.',
    tools=[
        MCPToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='npx',
                    args=[
                        "-y",
                        "@modelcontextprotocol/server-google-maps",
                    ],
                    # Pass the API key as an environment variable to the npx process
                    # This is how the MCP server for Google Maps expects the key.
                    env={
                        "GOOGLE_MAPS_API_KEY": google_maps_api_key
                    }
                ),
            ),
            # You can filter for specific Maps tools if needed:
            # tool_filter=['get_directions', 'find_place_by_id']
        )
    ],
)
```

#### Step 3: Ensure __init__.py Exists

If you created this in Example 1, you can skip this. Otherwise, ensure you have an `__init__.py` in the `./adk_agent_samples/mcp_agent/` directory:

```
# ./adk_agent_samples/mcp_agent/__init__.py
from . import agent
```

#### Step 4: Run adk web and Interact

1. **Set Environment Variable (Recommended):** Before running `adk web` , it's best to set your Google Maps API key as an environment variable in your terminal:

```
export GOOGLE_MAPS_API_KEY="YOUR_ACTUAL_GOOGLE_MAPS_API_KEY"
```

Replace `YOUR_ACTUAL_GOOGLE_MAPS_API_KEY` with your key.

2. **Run `adk web`** :
    Navigate to the parent directory of `mcp_agent` (e.g., `adk_agent_samples` ) and run:

```
cd ./adk_agent_samples # Or your equivalent parent directory
adk web
```

3. **Interact in the UI** :

- Select the `maps_assistant_agent` .
- Try prompts like:
- "Get directions from GooglePlex to SFO."
- "Find coffee shops near Golden Gate Park."
- "What's the route from Paris, France to Berlin, Germany?"

You should see the agent use the Google Maps MCP tools to provide directions or location-based information.


## 2. Building an MCP server with ADK tools (MCP server exposing ADK)

This pattern allows you to wrap existing ADK tools and make them available to any standard MCP client application. The example in this section exposes the ADK `load_web_page` tool through a custom-built MCP server.

### Summary of steps

You will create a standard Python MCP server application using the `mcp` library. Within this server, you will:

1. Instantiate the ADK tool(s) you want to expose (e.g., `FunctionTool(load_web_page)` ).
2. Implement the MCP server's `@app.list_tools()` handler to advertise the ADK tool(s). This involves converting the ADK tool definition to the MCP schema using the `adk_to_mcp_tool_type` utility from `google.adk.tools.mcp_tool.conversion_utils` .
3. Implement the MCP server's `@app.call_tool()` handler. This handler will:
- Receive tool call requests from MCP clients.
- Identify if the request targets one of your wrapped ADK tools.
- Execute the ADK tool's `.run_async()` method.
- Format the ADK tool's result into an MCP-compliant response (e.g., `mcp.types.TextContent` ).

### Prerequisites

Install the MCP server library in the same Python environment as your ADK installation:

```
pip install mcp
```

### Step 1: Create the MCP Server Script

Create a new Python file for your MCP server, for example, `my_adk_mcp_server.py` .

### Step 2: Implement the Server Logic

Add the following code to `my_adk_mcp_server.py` . This script sets up an MCP server that exposes the ADK `load_web_page` tool.

```
# my_adk_mcp_server.py
import asyncio
import json
import os
from dotenv import load_dotenv

# MCP Server Imports
from mcp import types as mcp_types # Use alias to avoid conflict
from mcp.server.lowlevel import Server, NotificationOptions
from mcp.server.models import InitializationOptions
import mcp.server.stdio # For running as a stdio server

# ADK Tool Imports
from google.adk.tools.function_tool import FunctionTool
from google.adk.tools.load_web_page import load_web_page # Example ADK tool
# ADK <-> MCP Conversion Utility
from google.adk.tools.mcp_tool.conversion_utils import adk_to_mcp_tool_type

# --- Load Environment Variables (If ADK tools need them, e.g., API keys) ---
load_dotenv() # Create a .env file in the same directory if needed

# --- Prepare the ADK Tool ---
# Instantiate the ADK tool you want to expose.
# This tool will be wrapped and called by the MCP server.
print("Initializing ADK load_web_page tool...")
adk_tool_to_expose = FunctionTool(load_web_page)
print(f"ADK tool '{adk_tool_to_expose.name}' initialized and ready to be exposed via MCP.")
# --- End ADK Tool Prep ---

# --- MCP Server Setup ---
print("Creating MCP Server instance...")
# Create a named MCP Server instance using the mcp.server library
app = Server("adk-tool-exposing-mcp-server")

# Implement the MCP server's handler to list available tools
@app.list_tools()
async def list_mcp_tools() -> list[mcp_types.Tool]:
    """MCP handler to list tools this server exposes."""
    print("MCP Server: Received list_tools request.")
    # Convert the ADK tool's definition to the MCP Tool schema format
    mcp_tool_schema = adk_to_mcp_tool_type(adk_tool_to_expose)
    print(f"MCP Server: Advertising tool: {mcp_tool_schema.name}")
    return [mcp_tool_schema]

# Implement the MCP server's handler to execute a tool call
@app.call_tool()
async def call_mcp_tool(
    name: str, arguments: dict
) -> list[mcp_types.Content]: # MCP uses mcp_types.Content
    """MCP handler to execute a tool call requested by an MCP client."""
    print(f"MCP Server: Received call_tool request for '{name}' with args: {arguments}")

    # Check if the requested tool name matches our wrapped ADK tool
    if name == adk_tool_to_expose.name:
        try:
            # Execute the ADK tool's run_async method.
            # Note: tool_context is None here because this MCP server is
            # running the ADK tool outside of a full ADK Runner invocation.
            # If the ADK tool requires ToolContext features (like state or auth),
            # this direct invocation might need more sophisticated handling.
            adk_tool_response = await adk_tool_to_expose.run_async(
                args=arguments,
                tool_context=None,
            )
            print(f"MCP Server: ADK tool '{name}' executed. Response: {adk_tool_response}")

            # Format the ADK tool's response (often a dict) into an MCP-compliant format.
            # Here, we serialize the response dictionary as a JSON string within TextContent.
            # Adjust formatting based on the ADK tool's output and client needs.
            response_text = json.dumps(adk_tool_response, indent=)
            # MCP expects a list of mcp_types.Content parts
            return [mcp_types.TextContent(type="text", text=response_text)]

        except Exception as e:
            print(f"MCP Server: Error executing ADK tool '{name}': {e}")
            # Return an error message in MCP format
            error_text = json.dumps({"error": f"Failed to execute tool '{name}': {str(e)}"})
            return [mcp_types.TextContent(type="text", text=error_text)]
    else:
        # Handle calls to unknown tools
        print(f"MCP Server: Tool '{name}' not found/exposed by this server.")
        error_text = json.dumps({"error": f"Tool '{name}' not implemented by this server."})
        return [mcp_types.TextContent(type="text", text=error_text)]

# --- MCP Server Runner ---
async def run_mcp_stdio_server():
    """Runs the MCP server, listening for connections over standard input/output."""
    # Use the stdio_server context manager from the mcp.server.stdio library
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        print("MCP Stdio Server: Starting handshake with client...")
        await app.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name=app.name, # Use the server name defined above
                server_version="0.1.0",
                capabilities=app.get_capabilities(
                    # Define server capabilities - consult MCP docs for options
                    notification_options=NotificationOptions(),
                    experimental_capabilities={},
                ),
            ),
        )
        print("MCP Stdio Server: Run loop finished or client disconnected.")

if __name__ == "__main__":
    print("Launching MCP Server to expose ADK tools via stdio...")
    try:
        asyncio.run(run_mcp_stdio_server())
    except KeyboardInterrupt:
        print("\nMCP Server (stdio) stopped by user.")
    except Exception as e:
        print(f"MCP Server (stdio) encountered an error: {e}")
    finally:
        print("MCP Server (stdio) process exiting.")
# --- End MCP Server ---
```

### Step 3: Test your Custom MCP Server with an ADK Agent

Now, create an ADK agent that will act as a client to the MCP server you just built. This ADK agent will use `MCPToolset` to connect to your `my_adk_mcp_server.py` script.

Create an `agent.py` (e.g., in `./adk_agent_samples/mcp_client_agent/agent.py` ):

```
# ./adk_agent_samples/mcp_client_agent/agent.py
import os
from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# IMPORTANT: Replace this with the ABSOLUTE path to your my_adk_mcp_server.py script
PATH_TO_YOUR_MCP_SERVER_SCRIPT = "/path/to/your/my_adk_mcp_server.py" # <<< REPLACE

if PATH_TO_YOUR_MCP_SERVER_SCRIPT == "/path/to/your/my_adk_mcp_server.py":
    print("WARNING: PATH_TO_YOUR_MCP_SERVER_SCRIPT is not set. Please update it in agent.py.")
    # Optionally, raise an error if the path is critical

root_agent = LlmAgent(
    model='gemini-2.5-flash',
    name='web_reader_mcp_client_agent',
    instruction="Use the 'load_web_page' tool to fetch content from a URL provided by the user.",
    tools=[
        MCPToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='python3', # Command to run your MCP server script
                    args=[PATH_TO_YOUR_MCP_SERVER_SCRIPT], # Argument is the path to the script
                )
            )
            # tool_filter=['load_web_page'] # Optional: ensure only specific tools are loaded
        )
    ],
)
```

And an `__init__.py` in the same directory:

```
# ./adk_agent_samples/mcp_client_agent/__init__.py
from . import agent
```

**To run the test:**

1. **Start your custom MCP server (optional, for separate observation):** You can run your `my_adk_mcp_server.py` directly in one terminal to see its logs:

```
python3 /path/to/your/my_adk_mcp_server.py
```

It will print "Launching MCP Server..." and wait. The ADK agent (run via `adk web` ) will then connect to this process if the `command` in `StdioConnectionParams` is set up to execute it. *(Alternatively, `MCPToolset` will start this server script as a subprocess automatically when the agent initializes).*

2. **Run `adk web` for the client agent:** Navigate to the parent directory of `mcp_client_agent` (e.g., `adk_agent_samples` ) and run:

```
cd ./adk_agent_samples # Or your equivalent parent directory
adk web
```

3. **Interact in the ADK Web UI:**

- Select the `web_reader_mcp_client_agent` .
- Try a prompt like: "Load the content from https://example.com"

The ADK agent ( `web_reader_mcp_client_agent` ) will use `MCPToolset` to start and connect to your `my_adk_mcp_server.py` . Your MCP server will receive the `call_tool` request, execute the ADK `load_web_page` tool, and return the result. The ADK agent will then relay this information. You should see logs from both the ADK Web UI (and its terminal) and potentially from your `my_adk_mcp_server.py` terminal if you ran it separately.

This example demonstrates how ADK tools can be encapsulated within an MCP server, making them accessible to a broader range of MCP-compliant clients, not just ADK agents.

Refer to the documentation , to try it out with Claude Desktop.

## Using MCP Tools in your own Agent out of adk web

This section is relevant to you if:

- You are developing your own Agent using ADK
- And, you are **NOT** using `adk web` ,
- And, you are exposing the agent via your own UI

Using MCP Tools requires a different setup than using regular tools, due to the fact that specs for MCP Tools are fetched asynchronously
from the MCP Server running remotely, or in another process.

The following example is modified from the "Example 1: File System MCP Server" example above. The main differences are:

1. Your tool and agent are created asynchronously
2. You need to properly manage the exit stack, so that your agents and tools are destructed properly when the connection to MCP Server is closed.
```
# agent.py (modify get_tools_async and other parts as needed)
# ./adk_agent_samples/mcp_agent/agent.py
import os
import asyncio
from dotenv import load_dotenv
from google.genai import types
from google.adk.agents.llm_agent import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.artifacts.in_memory_artifact_service import InMemoryArtifactService # Optional
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters

# Load environment variables from .env file in the parent directory
# Place this near the top, before using env vars like API keys
load_dotenv('../.env')

# Ensure TARGET_FOLDER_PATH is an absolute path for the MCP server.
TARGET_FOLDER_PATH = os.path.join(os.path.dirname(os.path.abspath(__file__)), "/path/to/your/folder")

# --- Step 1: Agent Definition ---
async def get_agent_async():
  """Creates an ADK Agent equipped with tools from the MCP Server."""
  toolset = MCPToolset(
      # Use StdioConnectionParams for local process communication
      connection_params=StdioConnectionParams(
          server_params = StdioServerParameters(
            command='npx', # Command to run the server
            args=["-y",    # Arguments for the command
                "@modelcontextprotocol/server-filesystem",
                TARGET_FOLDER_PATH],
          ),
      ),
      tool_filter=['read_file', 'list_directory'] # Optional: filter specific tools
      # For remote servers, you would use SseConnectionParams instead:
      # connection_params=SseConnectionParams(url="http://remote-server:port/path", headers={...})
  )

  # Use in an agent
  root_agent = LlmAgent(
      model='gemini-2.5-flash', # Adjust model name if needed based on availability
      name='enterprise_assistant',
      instruction='Help user accessing their file systems',
      tools=[toolset], # Provide the MCP tools to the ADK agent
  )
  return root_agent, toolset

# --- Step 2: Main Execution Logic ---
async def async_main():
  session_service = InMemorySessionService()
  # Artifact service might not be needed for this example
  artifacts_service = InMemoryArtifactService()

  session = await session_service.create_session(
      state={}, app_name='mcp_filesystem_app', user_id='user_fs'
  )

  # TODO: Change the query to be relevant to YOUR specified folder.
  # e.g., "list files in the 'documents' subfolder" or "read the file 'notes.txt'"
  query = "list files in the tests folder"
  print(f"User Query: '{query}'")
  content = types.Content(role='user', parts=[types.Part(text=query)])

  root_agent, toolset = await get_agent_async()

  runner = Runner(
      app_name='mcp_filesystem_app',
      agent=root_agent,
      artifact_service=artifacts_service, # Optional
      session_service=session_service,
  )

  print("Running agent...")
  events_async = runner.run_async(
      session_id=session.id, user_id=session.user_id, new_message=content
  )

  async for event in events_async:
    print(f"Event received: {event}")

  # Cleanup is handled automatically by the agent framework
  # But you can also manually close if needed:
  print("Closing MCP server connection...")
  await toolset.close()
  print("Cleanup complete.")

if __name__ == '__main__':
  try:
    asyncio.run(async_main())
  except Exception as e:
    print(f"An error occurred: {e}")
```

## Key considerations

When working with MCP and ADK, keep these points in mind:

- **Protocol vs. Library:** MCP is a protocol specification, defining communication rules. ADK is a Python library/framework for building agents. MCPToolset bridges these by implementing the client side of the MCP protocol within the ADK framework. Conversely, building an MCP server in Python requires using the model-context-protocol library.
- **ADK Tools vs. MCP Tools:**

- ADK Tools (BaseTool, FunctionTool, AgentTool, etc.) are Python objects designed for direct use within the ADK's LlmAgent and Runner.
- MCP Tools are capabilities exposed by an MCP Server according to the protocol's schema. MCPToolset makes these look like ADK tools to an LlmAgent.
- Langchain/CrewAI Tools are specific implementations within those libraries, often simple functions or classes, lacking the server/protocol structure of MCP. ADK offers wrappers (LangchainTool, CrewaiTool) for some interoperability.

- **Asynchronous nature:** Both ADK and the MCP Python library are heavily based on the asyncio Python library. Tool implementations and server handlers should generally be async functions.
- **Stateful sessions (MCP):** MCP establishes stateful, persistent connections between a client and server instance. This differs from typical stateless REST APIs.

- **Deployment:** This statefulness can pose challenges for scaling and deployment, especially for remote servers handling many users. The original MCP design often assumed client and server were co-located. Managing these persistent connections requires careful infrastructure considerations (e.g., load balancing, session affinity).
- **ADK MCPToolset:** Manages this connection lifecycle. The exit_stack pattern shown in the examples is crucial for ensuring the connection (and potentially the server process) is properly terminated when the ADK agent finishes.

-----------------

# Runtime

## What is runtime?

The ADK Runtime is the underlying engine that powers your agent application during user interactions. It's the system that takes your defined agents, tools, and callbacks and orchestrates their execution in response to user input, managing the flow of information, state changes, and interactions with external services like LLMs or storage.

Think of the Runtime as the **"engine"** of your agentic application. You define the parts (agents, tools), and the Runtime handles how they connect and run together to fulfill a user's request.

## Core Idea: The Event Loop

At its heart, the ADK Runtime operates on an **Event Loop** . This loop facilitates a back-and-forth communication between the `Runner` component and your defined "Execution Logic" (which includes your Agents, the LLM calls they make, Callbacks, and Tools).

In simple terms:

1. The `Runner` receives a user query and asks the main `Agent` to start processing.
2. The `Agent` (and its associated logic) runs until it has something to report (like a response, a request to use a tool, or a state change) – it then **yields** or **emits** an `Event` .
3. The `Runner` receives this `Event` , processes any associated actions (like saving state changes via `Services` ), and forwards the event onwards (e.g., to the user interface).
4. Only *after* the `Runner` has processed the event does the `Agent` 's logic **resume** from where it paused, now potentially seeing the effects of the changes committed by the Runner.
5. This cycle repeats until the agent has no more events to yield for the current user query.

This event-driven loop is the fundamental pattern governing how ADK executes your agent code.

## The Heartbeat: The Event Loop - Inner workings

The Event Loop is the core operational pattern defining the interaction between the `Runner` and your custom code (Agents, Tools, Callbacks, collectively referred to as "Execution Logic" or "Logic Components" in the design document). It establishes a clear division of responsibilities:

Note

The specific method names and parameter names may vary slightly by SDK language (e.g., `agent_to_run.runAsync(...)` in Java, `agent_to_run.run_async(...)` in Python). Refer to the language-specific API documentation for details.

### Runner's Role (Orchestrator)

The `Runner` acts as the central coordinator for a single user invocation. Its responsibilities in the loop are:

1. **Initiation:** Receives the end user's query ( `new_message` ) and typically appends it to the session history via the `SessionService` .
2. **Kick-off:** Starts the event generation process by calling the main agent's execution method (e.g., `agent_to_run.run_async(...)` ).
3. **Receive & Process:** Waits for the agent logic to `yield` or `emit` an `Event` . Upon receiving an event, the Runner **promptly processes** it. This involves:
- Using configured `Services` ( `SessionService` , `ArtifactService` , `MemoryService` ) to commit changes indicated in `event.actions` (like `state_delta` , `artifact_delta` ).
- Performing other internal bookkeeping.

4. **Yield Upstream:** Forwards the processed event onwards (e.g., to the calling application or UI for rendering).
5. **Iterate:** Signals the agent logic that processing is complete for the yielded event, allowing it to resume and generate the *next* event.

*Conceptual Runner Loop:*

```
# Simplified view of Runner's main loop logic
def run(new_query, ...) -> Generator[Event]:
    # 1. Append new_query to session event history (via SessionService)
    session_service.append_event(session, Event(author='user', content=new_query))

    # 2. Kick off event loop by calling the agent
    agent_event_generator = agent_to_run.run_async(context)

    async for event in agent_event_generator:
        # 3. Process the generated event and commit changes
        session_service.append_event(session, event) # Commits state/artifact deltas etc.
        # memory_service.update_memory(...) # If applicable
        # artifact_service might have already been called via context during agent run

        # 4. Yield event for upstream processing (e.g., UI rendering)
        yield event
        # Runner implicitly signals agent generator can continue after yielding
```

### Execution Logic's Role (Agent, Tool, Callback)

Your code within agents, tools, and callbacks is responsible for the actual computation and decision-making. Its interaction with the loop involves:

1. **Execute:** Runs its logic based on the current `InvocationContext` , including the session state *as it was when execution resumed* .
2. **Yield:** When the logic needs to communicate (send a message, call a tool, report a state change), it constructs an `Event` containing the relevant content and actions, and then `yield` s this event back to the `Runner` .
3. **Pause:** Crucially, execution of the agent logic **pauses immediately** after the `yield` statement (or `return` in RxJava). It waits for the `Runner` to complete step 3 (processing and committing).
4. **Resume:**  *Only after* the `Runner` has processed the yielded event does the agent logic resume execution from the statement immediately following the `yield` .
5. **See Updated State:** Upon resumption, the agent logic can now reliably access the session state ( `ctx.session.state` ) reflecting the changes that were committed by the `Runner` from the *previously yielded* event.

*Conceptual Execution Logic:*

```
# Simplified view of logic inside Agent.run_async, callbacks, or tools

# ... previous code runs based on current state ...

# 1. Determine a change or output is needed, construct the event
# Example: Updating state
update_data = {'field_1': 'value_2'}
event_with_state_change = Event(
    author=self.name,
    actions=EventActions(state_delta=update_data),
    content=types.Content(parts=[types.Part(text="State updated.")])
    # ... other event fields ...
)

# 2. Yield the event to the Runner for processing & commit
yield event_with_state_change
# <<<<<<<<<<<< EXECUTION PAUSES HERE >>>>>>>>>>>>

# <<<<<<<<<<<< RUNNER PROCESSES & COMMITS THE EVENT >>>>>>>>>>>>

# 3. Resume execution ONLY after Runner is done processing the above event.
# Now, the state committed by the Runner is reliably reflected.
# Subsequent code can safely assume the change from the yielded event happened.
val = ctx.session.state['field_1']
# here `val` is guaranteed to be "value_2" (assuming Runner committed successfully)
print(f"Resumed execution. Value of field_1 is now: {val}")

# ... subsequent code continues ...
# Maybe yield another event later...
```

This cooperative yield/pause/resume cycle between the `Runner` and your Execution Logic, mediated by `Event` objects, forms the core of the ADK Runtime.

## Key components of the Runtime

Several components work together within the ADK Runtime to execute an agent invocation. Understanding their roles clarifies how the event loop functions:

1. ### Runner

- **Role:** The main entry point and orchestrator for a single user query ( `run_async` ).
- **Function:** Manages the overall Event Loop, receives events yielded by the Execution Logic, coordinates with Services to process and commit event actions (state/artifact changes), and forwards processed events upstream (e.g., to the UI). It essentially drives the conversation turn by turn based on yielded events. (Defined in `google.adk.runners.runner` ).

2. ### Execution Logic Components

- **Role:** The parts containing your custom code and the core agent capabilities.
- **Components:**
- `Agent` ( `BaseAgent` , `LlmAgent` , etc.): Your primary logic units that process information and decide on actions. They implement the `_run_async_impl` method which yields events.
- `Tools` ( `BaseTool` , `FunctionTool` , `AgentTool` , etc.): External functions or capabilities used by agents (often `LlmAgent` ) to interact with the outside world or perform specific tasks. They execute and return results, which are then wrapped in events.
- `Callbacks` (Functions): User-defined functions attached to agents (e.g., `before_agent_callback` , `after_model_callback` ) that hook into specific points in the execution flow, potentially modifying behavior or state, whose effects are captured in events.
- **Function:** Perform the actual thinking, calculation, or external interaction. They communicate their results or needs by **yielding `Event` objects** and pausing until the Runner processes them.

3. ### Event

- **Role:** The message passed back and forth between the `Runner` and the Execution Logic.
- **Function:** Represents an atomic occurrence (user input, agent text, tool call/result, state change request, control signal). It carries both the content of the occurrence and the intended side effects ( `actions` like `state_delta` ).

4. ### Services

- **Role:** Backend components responsible for managing persistent or shared resources. Used primarily by the `Runner` during event processing.
- **Components:**
- `SessionService` ( `BaseSessionService` , `InMemorySessionService` , etc.): Manages `Session` objects, including saving/loading them, applying `state_delta` to the session state, and appending events to the `event history` .
- `ArtifactService` ( `BaseArtifactService` , `InMemoryArtifactService` , `GcsArtifactService` , etc.): Manages the storage and retrieval of binary artifact data. Although `save_artifact` is called via context during execution logic, the `artifact_delta` in the event confirms the action for the Runner/SessionService.
- `MemoryService` ( `BaseMemoryService` , etc.): (Optional) Manages long-term semantic memory across sessions for a user.
- **Function:** Provide the persistence layer. The `Runner` interacts with them to ensure changes signaled by `event.actions` are reliably stored *before* the Execution Logic resumes.

5. ### Session

- **Role:** A data container holding the state and history for *one specific conversation* between a user and the application.
- **Function:** Stores the current `state` dictionary, the list of all past `events` ( `event history` ), and references to associated artifacts. It's the primary record of the interaction, managed by the `SessionService` .

6. ### Invocation

- **Role:** A conceptual term representing everything that happens in response to a *single* user query, from the moment the `Runner` receives it until the agent logic finishes yielding events for that query.
- **Function:** An invocation might involve multiple agent runs (if using agent transfer or `AgentTool` ), multiple LLM calls, tool executions, and callback executions, all tied together by a single `invocation_id` within the `InvocationContext` . State variables prefixed with `temp:` are strictly scoped to a single invocation and discarded afterwards.

These players interact continuously through the Event Loop to process a user's request.

## How It Works: A Simplified Invocation

Let's trace a simplified flow for a typical user query that involves an LLM agent calling a tool:

### Step-by-Step Breakdown

1. **User Input:** The User sends a query (e.g., "What's the capital of France?").
2. **Runner Starts:**  `Runner.run_async` begins. It interacts with the `SessionService` to load the relevant `Session` and adds the user query as the first `Event` to the session history. An `InvocationContext` ( `ctx` ) is prepared.
3. **Agent Execution:** The `Runner` calls `agent.run_async(ctx)` on the designated root agent (e.g., an `LlmAgent` ).
4. **LLM Call (Example):** The `Agent_Llm` determines it needs information, perhaps by calling a tool. It prepares a request for the `LLM` . Let's assume the LLM decides to call `MyTool` .
5. **Yield FunctionCall Event:** The `Agent_Llm` receives the `FunctionCall` response from the LLM, wraps it in an `Event(author='Agent_Llm', content=Content(parts=[Part(function_call=...)]))` , and `yields` or `emits` this event.
6. **Agent Pauses:** The `Agent_Llm` 's execution pauses immediately after the `yield` .
7. **Runner Processes:** The `Runner` receives the FunctionCall event. It passes it to the `SessionService` to record it in the history. The `Runner` then yields the event upstream to the `User` (or application).
8. **Agent Resumes:** The `Runner` signals that the event is processed, and `Agent_Llm` resumes execution.
9. **Tool Execution:** The `Agent_Llm` 's internal flow now proceeds to execute the requested `MyTool` . It calls `tool.run_async(...)` .
10. **Tool Returns Result:**  `MyTool` executes and returns its result (e.g., `{'result': 'Paris'}` ).
11. **Yield FunctionResponse Event:** The agent ( `Agent_Llm` ) wraps the tool result into an `Event` containing a `FunctionResponse` part (e.g., `Event(author='Agent_Llm', content=Content(role='user', parts=[Part(function_response=...)]))` ). This event might also contain `actions` if the tool modified state ( `state_delta` ) or saved artifacts ( `artifact_delta` ). The agent `yield` s this event.
12. **Agent Pauses:**  `Agent_Llm` pauses again.
13. **Runner Processes:**  `Runner` receives the FunctionResponse event. It passes it to `SessionService` which applies any `state_delta` / `artifact_delta` and adds the event to history. `Runner` yields the event upstream.
14. **Agent Resumes:**  `Agent_Llm` resumes, now knowing the tool result and any state changes are committed.
15. **Final LLM Call (Example):**  `Agent_Llm` sends the tool result back to the `LLM` to generate a natural language response.
16. **Yield Final Text Event:**  `Agent_Llm` receives the final text from the `LLM` , wraps it in an `Event(author='Agent_Llm', content=Content(parts=[Part(text=...)]))` , and `yield` s it.
17. **Agent Pauses:**  `Agent_Llm` pauses.
18. **Runner Processes:**  `Runner` receives the final text event, passes it to `SessionService` for history, and yields it upstream to the `User` . This is likely marked as the `is_final_response()` .
19. **Agent Resumes & Finishes:**  `Agent_Llm` resumes. Having completed its task for this invocation, its `run_async` generator finishes.
20. **Runner Completes:** The `Runner` sees the agent's generator is exhausted and finishes its loop for this invocation.

This yield/pause/process/resume cycle ensures that state changes are consistently applied and that the execution logic always operates on the most recently committed state after yielding an event.

## Important Runtime Behaviors

Understanding a few key aspects of how the ADK Runtime handles state, streaming, and asynchronous operations is crucial for building predictable and efficient agents.

### State Updates & Commitment Timing

- **The Rule:** When your code (in an agent, tool, or callback) modifies the session state (e.g., `context.state['my_key'] = 'new_value'` ), this change is initially recorded locally within the current `InvocationContext` . The change is only **guaranteed to be persisted** (saved by the `SessionService` ) *after* the `Event` carrying the corresponding `state_delta` in its `actions` has been `yield` -ed by your code and subsequently processed by the `Runner` .
- **Implication:** Code that runs *after* resuming from a `yield` can reliably assume that the state changes signaled in the *yielded event* have been committed.

```
# Inside agent logic (conceptual)

# 1. Modify state
ctx.session.state['status'] = 'processing'
event1 = Event(..., actions=EventActions(state_delta={'status': 'processing'}))

# 2. Yield event with the delta
yield event1
# --- PAUSE --- Runner processes event1, SessionService commits 'status' = 'processing' ---

# 3. Resume execution
# Now it's safe to rely on the committed state
current_status = ctx.session.state['status'] # Guaranteed to be 'processing'
print(f"Status after resuming: {current_status}")
```

### "Dirty Reads" of Session State

- **Definition:** While commitment happens *after* the yield, code running *later within the same invocation* , but *before* the state-changing event is actually yielded and processed, **can often see the local, uncommitted changes** . This is sometimes called a "dirty read".
- **Example:**

```
# Code in before_agent_callback
callback_context.state['field_1'] = 'value_1'
# State is locally set to 'value_1', but not yet committed by Runner

# ... agent runs ...

# Code in a tool called later *within the same invocation*
# Readable (dirty read), but 'value_1' isn't guaranteed persistent yet.
val = tool_context.state['field_1'] # 'val' will likely be 'value_1' here
print(f"Dirty read value in tool: {val}")

# Assume the event carrying the state_delta={'field_1': 'value_1'}
# is yielded *after* this tool runs and is processed by the Runner.
```

- **Implications:**
- **Benefit:** Allows different parts of your logic within a single complex step (e.g., multiple callbacks or tool calls before the next LLM turn) to coordinate using state without waiting for a full yield/commit cycle.
- **Caveat:** Relying heavily on dirty reads for critical logic can be risky. If the invocation fails *before* the event carrying the `state_delta` is yielded and processed by the `Runner` , the uncommitted state change will be lost. For critical state transitions, ensure they are associated with an event that gets successfully processed.

### Streaming vs. Non-Streaming Output (partial=True)

This primarily relates to how responses from the LLM are handled, especially when using streaming generation APIs.

- **Streaming:** The LLM generates its response token-by-token or in small chunks.
- The framework (often within `BaseLlmFlow` ) yields multiple `Event` objects for a single conceptual response. Most of these events will have `partial=True` .
- The `Runner` , upon receiving an event with `partial=True` , typically **forwards it immediately** upstream (for UI display) but **skips processing its `actions`** (like `state_delta` ).
- Eventually, the framework yields a final event for that response, marked as non-partial ( `partial=False` or implicitly via `turn_complete=True` ).
- The `Runner`  **fully processes only this final event** , committing any associated `state_delta` or `artifact_delta` .
- **Non-Streaming:** The LLM generates the entire response at once. The framework yields a single event marked as non-partial, which the `Runner` processes fully.
- **Why it Matters:** Ensures that state changes are applied atomically and only once based on the *complete* response from the LLM, while still allowing the UI to display text progressively as it's generated.

## Async is Primary (run_async)

- **Core Design:** The ADK Runtime is fundamentally built on asynchronous libraries (like Python's `asyncio` and Java's `RxJava` ) to handle concurrent operations (like waiting for LLM responses or tool executions) efficiently without blocking.
- **Main Entry Point:**  `Runner.run_async` is the primary method for executing agent invocations. All core runnable components (Agents, specific flows) use `asynchronous` methods internally.
- **Synchronous Convenience ( `run` ):** A synchronous `Runner.run` method exists mainly for convenience (e.g., in simple scripts or testing environments). However, internally, `Runner.run` typically just calls `Runner.run_async` and manages the async event loop execution for you.
- **Developer Experience:** We recommend designing your applications (e.g., web servers using ADK) to be asynchronous for best performance. In Python, this means using `asyncio` ; in Java, leverage `RxJava` 's reactive programming model.
- **Sync Callbacks/Tools:** The ADK framework supports both asynchronous and synchronous functions for tools and callbacks.
- **Blocking I/O:** For long-running synchronous I/O operations, the framework attempts to prevent stalls. Python ADK may use asyncio.to_thread, while Java ADK often relies on appropriate RxJava schedulers or wrappers for blocking calls.
- **CPU-Bound Work:** Purely CPU-intensive synchronous tasks will still block their execution thread in both environments.

Understanding these behaviors helps you write more robust ADK applications and debug issues related to state consistency, streaming updates, and asynchronous execution.

-----------------

# Runtime Configuration

`RunConfig` defines runtime behavior and options for agents in the ADK. It
controls speech and streaming settings, function calling, artifact saving, and
limits on LLM calls.

When constructing an agent run, you can pass a `RunConfig` to customize how the
agent interacts with models, handles audio, and streams responses. By default,
no streaming is enabled and inputs aren’t retained as artifacts. Use `RunConfig` to override these defaults.

## Class Definition

The `RunConfig` class holds configuration parameters for an agent's runtime behavior.

- Python ADK uses Pydantic for this validation.
- Java ADK typically uses immutable data classes.

```
class RunConfig(BaseModel):
    """Configs for runtime behavior of agents."""

    model_config = ConfigDict(
        extra='forbid',
    )

    speech_config: Optional[types.SpeechConfig] = None
    response_modalities: Optional[list[str]] = None
    save_input_blobs_as_artifacts: bool = False
    support_cfc: bool = False
    streaming_mode: StreamingMode = StreamingMode.NONE
    output_audio_transcription: Optional[types.AudioTranscriptionConfig] = None
    max_llm_calls: int =
```

## Runtime Parameters

| Parameter | Python Type | Java Type | Default (Py / Java) | Description |
| --- | --- | --- | --- | --- |
| speech_config | Optional[types.SpeechConfig] | SpeechConfig (nullable via @Nullable) | None / null | Configures speech synthesis (voice, language) using the SpeechConfig type. |
| response_modalities | Optional[list[str]] | ImmutableList<Modality> | None / Empty ImmutableList | List of desired output modalities (e.g., Python: ["TEXT", "AUDIO"]; Java: uses structured Modality objects). |
| save_input_blobs_as_artifacts | bool | boolean | False / false | If true, saves input blobs (e.g., uploaded files) as run artifacts for debugging/auditing. |
| streaming_mode | StreamingMode | Currently not supported | StreamingMode.NONE / N/A | Sets the streaming behavior: NONE (default), SSE (server-sent events), or BIDI (bidirectional). |
| output_audio_transcription | Optional[types.AudioTranscriptionConfig] | AudioTranscriptionConfig (nullable via @Nullable) | None / null | Configures transcription of generated audio output using the AudioTranscriptionConfig type. |
| max_llm_calls | int | int | 500 / 500 | Limits total LLM calls per run. 0 or negative means unlimited (warned); sys.maxsize raises ValueError. |
| support_cfc | bool | Currently not supported | False / N/A | Python: Enables Compositional Function Calling. Requires streaming_mode=SSE and uses the LIVE API. Experimental. |

### speech_config

Note

The interface or definition of `SpeechConfig` is the same, irrespective of the language.

Speech configuration settings for live agents with audio capabilities. The `SpeechConfig` class has the following structure:

```
class SpeechConfig(_common.BaseModel):
    """The speech generation configuration."""

    voice_config: Optional[VoiceConfig] = Field(
        default=None,
        description="""The configuration for the speaker to use.""",
    )
    language_code: Optional[str] = Field(
        default=None,
        description="""Language code (ISO 639. e.g. en-US) for the speech synthesization.
        Only available for Live API.""",
    )
```

The `voice_config` parameter uses the `VoiceConfig` class:

```
class VoiceConfig(_common.BaseModel):
    """The configuration for the voice to use."""

    prebuilt_voice_config: Optional[PrebuiltVoiceConfig] = Field(
        default=None,
        description="""The configuration for the speaker to use.""",
    )
```

And `PrebuiltVoiceConfig` has the following structure:

```
class PrebuiltVoiceConfig(_common.BaseModel):
    """The configuration for the prebuilt speaker to use."""

    voice_name: Optional[str] = Field(
        default=None,
        description="""The name of the prebuilt voice to use.""",
    )
```

These nested configuration classes allow you to specify:

- `voice_config` : The name of the prebuilt voice to use (in the `PrebuiltVoiceConfig` )
- `language_code` : ISO 639 language code (e.g., "en-US") for speech synthesis

When implementing voice-enabled agents, configure these parameters to control
how your agent sounds when speaking.

### response_modalities

Defines the output modalities for the agent. If not set, defaults to AUDIO.
Response modalities determine how the agent communicates with users through
various channels (e.g., text, audio).

### save_input_blobs_as_artifacts

When enabled, input blobs will be saved as artifacts during agent execution.
This is useful for debugging and audit purposes, allowing developers to review
the exact data received by agents.

### support_cfc

Enables Compositional Function Calling (CFC) support. Only applicable when using
StreamingMode.SSE. When enabled, the LIVE API will be invoked as only it
supports CFC functionality.

Warning

The `support_cfc` feature is experimental and its API or behavior might
change in future releases.

### streaming_mode

Configures the streaming behavior of the agent. Possible values:

- `StreamingMode.NONE` : No streaming; responses delivered as complete units
- `StreamingMode.SSE` : Server-Sent Events streaming; one-way streaming from server to client
- `StreamingMode.BIDI` : Bidirectional streaming; simultaneous communication in both directions

Streaming modes affect both performance and user experience. SSE streaming lets users see partial responses as they're generated, while BIDI streaming enables real-time interactive experiences.

### output_audio_transcription

Configuration for transcribing audio outputs from live agents with audio
response capability. This enables automatic transcription of audio responses for
accessibility, record-keeping, and multi-modal applications.

### max_llm_calls

Sets a limit on the total number of LLM calls for a given agent run.

- Values greater than 0 and less than `sys.maxsize` : Enforces a bound on LLM calls
- Values less than or equal to 0: Allows unbounded LLM calls *(not recommended for production)*

This parameter prevents excessive API usage and potential runaway processes.
Since LLM calls often incur costs and consume resources, setting appropriate
limits is crucial.

## Validation Rules

The `RunConfig` class validates its parameters to ensure proper agent operation. While Python ADK uses `Pydantic` for automatic type validation, Java ADK relies on its static typing and may include explicit checks in the RunConfig's construction.
For the `max_llm_calls` parameter specifically:

1. Extremely large values (like `sys.maxsize` in Python or `Integer.MAX_VALUE` in Java) are typically disallowed to prevent issues.
2. Values of zero or less will usually trigger a warning about unlimited LLM interactions.

## Examples

### Basic runtime configuration

```
from google.genai.adk import RunConfig, StreamingMode

config = RunConfig(
    streaming_mode=StreamingMode.NONE,
    max_llm_calls=
)
```

This configuration creates a non-streaming agent with a limit of 100 LLM calls,
suitable for simple task-oriented agents where complete responses are
preferable.

### Enabling streaming

```
from google.genai.adk import RunConfig, StreamingMode

config = RunConfig(
    streaming_mode=StreamingMode.SSE,
    max_llm_calls=
)
```

Using SSE streaming allows users to see responses as they're generated,
providing a more responsive feel for chatbots and assistants.

### Enabling speech support

```
from google.genai.adk import RunConfig, StreamingMode
from google.genai import types

config = RunConfig(
    speech_config=types.SpeechConfig(
        language_code="en-US",
        voice_config=types.VoiceConfig(
            prebuilt_voice_config=types.PrebuiltVoiceConfig(
                voice_name="Kore"
            )
        ),
    ),
    response_modalities=["AUDIO", "TEXT"],
    save_input_blobs_as_artifacts=True,
    support_cfc=True,
    streaming_mode=StreamingMode.SSE,
    max_llm_calls=,
)
```

This comprehensive example configures an agent with:

- Speech capabilities using the "Kore" voice (US English)
- Both audio and text output modalities
- Artifact saving for input blobs (useful for debugging)
- Experimental CFC support enabled **(Python only)**
- SSE streaming for responsive interaction
- A limit of 1000 LLM calls

### Enabling Experimental CFC Support

```
from google.genai.adk import RunConfig, StreamingMode

config = RunConfig(
    streaming_mode=StreamingMode.SSE,
    support_cfc=True,
    max_llm_calls=
)
```

Enabling Compositional Function Calling creates an agent that can dynamically
execute functions based on model outputs, powerful for applications requiring
complex workflows.

-----------------

# Introduction to Conversational Context: Session, State, and Memory

## Why Context Matters

Meaningful, multi-turn conversations require agents to understand context. Just
like humans, they need to recall the conversation history: what's been said and
done to maintain continuity and avoid repetition. The Agent Development Kit
(ADK) provides structured ways to manage this context through `Session` , `State` , and `Memory` .

## Core Concepts

Think of different instances of your conversations with the agent as distinct **conversation threads** , potentially drawing upon **long-term knowledge** .

1. **`Session`** : The Current Conversation Thread

- Represents a *single, ongoing interaction* between a user and your agent
    system.

- Contains the chronological sequence of messages and actions taken by the
    agent (referred to `Events` ) during *that specific interaction* .

- A `Session` can also hold temporary data ( `State` ) relevant only *during
    this conversation* .

2. **`State` ( `session.state` )** : Data Within the Current Conversation

- Data stored within a specific `Session` .
- Used to manage information relevant *only* to the *current, active* conversation thread (e.g., items in a shopping cart *during this chat* ,
    user preferences mentioned *in this session* ).

3. **`Memory`** : Searchable, Cross-Session Information

- Represents a store of information that might span *multiple past
    sessions* or include external data sources.

- It acts as a knowledge base the agent can *search* to recall information
    or context beyond the immediate conversation.

## Managing Context: Services

ADK provides services to manage these concepts:

1. **`SessionService`** : Manages the different conversation threads ( `Session` objects)

- Handles the lifecycle: creating, retrieving, updating (appending `Events` , modifying `State` ), and deleting individual `Session` s.

2. **`MemoryService`** : Manages the Long-Term Knowledge Store ( `Memory` )

- Handles ingesting information (often from completed `Session` s) into the
    long-term store.

- Provides methods to search this stored knowledge based on queries.

**Implementations** : ADK offers different implementations for both `SessionService` and `MemoryService` , allowing you to choose the storage backend
that best fits your application's needs. Notably, **in-memory implementations** are provided for both services; these are designed specifically for **local
testing and fast development** . It's important to remember that **all data
stored using these in-memory options (sessions, state, or long-term knowledge)
is lost when your application restarts** . For persistence and scalability beyond
local testing, ADK also offers cloud-based and database service options.

**In Summary:**

- **`Session` & `State`** : Focus on the **current interaction** – the history
    and data of the *single, active conversation* . Managed primarily by a `SessionService` .

- **Memory** : Focuses on the **past and external information** – a *searchable
    archive* potentially spanning across conversations. Managed by a `MemoryService` .

## What's Next?

In the following sections, we'll dive deeper into each of these components:

- **`Session`** : Understanding its structure and `Events` .
- **`State`** : How to effectively read, write, and manage session-specific
    data.

- **`SessionService`** : Choosing the right storage backend for your sessions.
- **`MemoryService`** : Exploring options for storing and retrieving broader
    context.

Understanding these concepts is fundamental to building agents that can engage
in complex, stateful, and context-aware conversations.

-----------------

# Session: Tracking Individual Conversations

Following our Introduction, let's dive into the `Session` . Think back to the
idea of a "conversation thread." Just like you wouldn't start every text message
from scratch, agents need context regarding the ongoing interaction. **`Session`** is the ADK object designed specifically to track and manage these
individual conversation threads.

## The Session Object

When a user starts interacting with your agent, the `SessionService` creates a `Session` object ( `google.adk.sessions.Session` ). This object acts as the
container holding everything related to that *one specific chat thread* . Here
are its key properties:

- **Identification ( `id` , `appName` , `userId` ):** Unique labels for the
    conversation.
- `id` : A unique identifier for *this specific* conversation thread, essential for retrieving it later. A SessionService object can handle multiple `Session` (s). This field identifies which particular session object are we referring to. For example, "test_id_modification".
- `app_name` : Identifies which agent application this conversation belongs to. For example, "id_modifier_workflow".
- `userId` : Links the conversation to a particular user.

- **History ( `events` ):** A chronological sequence of all interactions
    ( `Event` objects – user messages, agent responses, tool actions) that have
    occurred within this specific thread.

- **Session State ( `state` ):** A place to store temporary data relevant *only* to this specific, ongoing conversation. This acts as a scratchpad for the
    agent during the interaction. We will cover how to use and manage `state` in
    detail in the next section.

- **Activity Tracking ( `lastUpdateTime` ):** A timestamp indicating the last
    time an event occurred in this conversation thread.

### Example: Examining Session Properties

```
from google.adk.sessions import InMemorySessionService, Session

 # Create a simple session to examine its properties
 temp_service = InMemorySessionService()
 example_session = await temp_service.create_session(
     app_name="my_app",
     user_id="example_user",
     state={"initial_key": "initial_value"} # State can be initialized
 )

 print(f"--- Examining Session Properties ---")
 print(f"ID (`id`):                {example_session.id}")
 print(f"Application Name (`app_name`): {example_session.app_name}")
 print(f"User ID (`user_id`):         {example_session.user_id}")
 print(f"State (`state`):           {example_session.state}") # Note: Only shows initial state here
 print(f"Events (`events`):         {example_session.events}") # Initially empty
 print(f"Last Update (`last_update_time`): {example_session.last_update_time:.2f}")
 print(f"---------------------------------")

 # Clean up (optional for this example)
 temp_service = await temp_service.delete_session(app_name=example_session.app_name,
                             user_id=example_session.user_id, session_id=example_session.id)
 print("The final status of temp_service - ", temp_service)
```

*(*  *Note:*  *The state shown above is only the initial state. State updates
happen via events, as discussed in the State section.)*

## Managing Sessions with a SessionService

As seen above, you don't typically create or manage `Session` objects directly.
Instead, you use a **`SessionService`** . This service acts as the central
manager responsible for the entire lifecycle of your conversation sessions.

Its core responsibilities include:

- **Starting New Conversations:** Creating fresh `Session` objects when a user
    begins an interaction.

- **Resuming Existing Conversations:** Retrieving a specific `Session` (using
    its ID) so the agent can continue where it left off.

- **Saving Progress:** Appending new interactions ( `Event` objects) to a
    session's history. This is also the mechanism through which session `state` gets updated (more in the `State` section).

- **Listing Conversations:** Finding the active session threads for a
    particular user and application.

- **Cleaning Up:** Deleting `Session` objects and their associated data when
    conversations are finished or no longer needed.

## SessionService Implementations

ADK provides different `SessionService` implementations, allowing you to choose
the storage backend that best suits your needs:

1. **`InMemorySessionService`**

- **How it works:** Stores all session data directly in the application's
    memory.

- **Persistence:** None. **All conversation data is lost if the
    application restarts.**

- **Requires:** Nothing extra.
- **Best for:** Quick development, local testing, examples, and scenarios
    where long-term persistence isn't required.

```
from google.adk.sessions import InMemorySessionService
 session_service = InMemorySessionService()
```

2. **`VertexAiSessionService`**

- **How it works:** Uses Google Cloud Vertex AI infrastructure via API
    calls for session management.

- **Persistence:** Yes. Data is managed reliably and scalably via Vertex AI Agent Engine .
- **Requires:**
- A Google Cloud project ( `pip install vertexai` )
- A Google Cloud storage bucket that can be configured by this step .
- A Reasoning Engine resource name/ID that can setup following this tutorial .
- If you do not have a Google Cloud project and you want to try the VertexAiSessionService for free, see how to try Session and Memory for free.

- **Best for:** Scalable production applications deployed on Google Cloud,
    especially when integrating with other Vertex AI features.

```
# Requires: pip install google-adk[vertexai]
# Plus GCP setup and authentication
from google.adk.sessions import VertexAiSessionService

PROJECT_ID = "your-gcp-project-id"
LOCATION = "us-central1"
# The app_name used with this service should be the Reasoning Engine ID or name
REASONING_ENGINE_APP_NAME = "projects/your-gcp-project-id/locations/us-central1/reasoningEngines/your-engine-id"

session_service = VertexAiSessionService(project=PROJECT_ID, location=LOCATION)
# Use REASONING_ENGINE_APP_NAME when calling service methods, e.g.:
# session_service = await session_service.create_session(app_name=REASONING_ENGINE_APP_NAME, ...)
```

3. **`DatabaseSessionService`**

- **How it works:** Connects to a relational database (e.g., PostgreSQL,
    MySQL, SQLite) to store session data persistently in tables.

- **Persistence:** Yes. Data survives application restarts.
- **Requires:** A configured database.
- **Best for:** Applications needing reliable, persistent storage that you
    manage yourself.

```
from google.adk.sessions import DatabaseSessionService
# Example using a local SQLite file:
db_url = "sqlite:///./my_agent_data.db"
session_service = DatabaseSessionService(db_url=db_url)
```

Choosing the right `SessionService` is key to defining how your agent's
conversation history and temporary data are stored and persist.

## The Session Lifecycle

Here’s a simplified flow of how `Session` and `SessionService` work together
during a conversation turn:

1. **Start or Resume:** Your application needs to use the `SessionService` to
    either `create_session` (for a new chat) or use an existing session id.

2. **Context Provided:** The `Runner` gets the appropriate `Session` object
    from the appropriate service method, providing the agent with access to the
    corresponding Session's `state` and `events` .

3. **Agent Processing:** The user prompts the agent with a query. The agent
    analyzes the query and potentially the session `state` and `events` history
    to determine the response.

4. **Response & State Update:** The agent generates a response (and potentially
    flags data to be updated in the `state` ). The `Runner` packages this as an `Event` .

5. **Save Interaction:** The `Runner` calls `sessionService.append_event(session, event)` with the `session` and the new `event` as the arguments. The service adds the `Event` to the history and
    updates the session's `state` in storage based on information within the
    event. The session's `last_update_time` also get updated.

6. **Ready for Next:** The agent's response goes to the user. The updated `Session` is now stored by the `SessionService` , ready for the next turn
    (which restarts the cycle at step 1, usually with the continuation of the
    conversation in the current session).

7. **End Conversation:** When the conversation is over, your application calls `sessionService.delete_session(...)` to clean up the stored session data if
    it is no longer required.

This cycle highlights how the `SessionService` ensures conversational continuity
by managing the history and state associated with each `Session` object.

-----------------

# State: The Session's Scratchpad

Within each `Session` (our conversation thread), the **`state`** attribute acts like the agent's dedicated scratchpad for that specific interaction. While `session.events` holds the full history, `session.state` is where the agent stores and updates dynamic details needed *during* the conversation.

## What is session.state?

Conceptually, `session.state` is a collection (dictionary or Map) holding key-value pairs. It's designed for information the agent needs to recall or track to make the current conversation effective:

- **Personalize Interaction:** Remember user preferences mentioned earlier (e.g., `'user_preference_theme': 'dark'` ).
- **Track Task Progress:** Keep tabs on steps in a multi-turn process (e.g., `'booking_step': 'confirm_payment'` ).
- **Accumulate Information:** Build lists or summaries (e.g., `'shopping_cart_items': ['book', 'pen']` ).
- **Make Informed Decisions:** Store flags or values influencing the next response (e.g., `'user_is_authenticated': True` ).

### Key Characteristics of State

1. **Structure: Serializable Key-Value Pairs**

- Data is stored as `key: value` .
- **Keys:** Always strings ( `str` ). Use clear names (e.g., `'departure_city'` , `'user:language_preference'` ).
- **Values:** Must be **serializable** . This means they can be easily saved and loaded by the `SessionService` . Stick to basic types in the specific languages (Python/ Java) like strings, numbers, booleans, and simple lists or dictionaries containing *only* these basic types. (See API documentation for precise details).
- **⚠️ Avoid Complex Objects:**  **Do not store non-serializable objects** (custom class instances, functions, connections, etc.) directly in the state. Store simple identifiers if needed, and retrieve the complex object elsewhere.

2. **Mutability: It Changes**

- The contents of the `state` are expected to change as the conversation evolves.

3. **Persistence: Depends on `SessionService`**

- Whether state survives application restarts depends on your chosen service:
- `InMemorySessionService` : **Not Persistent.** State is lost on restart.
- `DatabaseSessionService` / `VertexAiSessionService` : **Persistent.** State is saved reliably.

Note

The specific parameters or method names for the primitives may vary slightly by SDK language (e.g., `session.state['current_intent'] = 'book_flight'` in Python, `session.state().put("current_intent", "book_flight)` in Java). Refer to the language-specific API documentation for details.

### Organizing State with Prefixes: Scope Matters

Prefixes on state keys define their scope and persistence behavior, especially with persistent services:

- **No Prefix (Session State):**

- **Scope:** Specific to the *current* session ( `id` ).
- **Persistence:** Only persists if the `SessionService` is persistent ( `Database` , `VertexAI` ).
- **Use Cases:** Tracking progress within the current task (e.g., `'current_booking_step'` ), temporary flags for this interaction (e.g., `'needs_clarification'` ).
- **Example:**  `session.state['current_intent'] = 'book_flight'`

- **`user:` Prefix (User State):**

- **Scope:** Tied to the `user_id` , shared across *all* sessions for that user (within the same `app_name` ).
- **Persistence:** Persistent with `Database` or `VertexAI` . (Stored by `InMemory` but lost on restart).
- **Use Cases:** User preferences (e.g., `'user:theme'` ), profile details (e.g., `'user:name'` ).
- **Example:**  `session.state['user:preferred_language'] = 'fr'`

- **`app:` Prefix (App State):**

- **Scope:** Tied to the `app_name` , shared across *all* users and sessions for that application.
- **Persistence:** Persistent with `Database` or `VertexAI` . (Stored by `InMemory` but lost on restart).
- **Use Cases:** Global settings (e.g., `'app:api_endpoint'` ), shared templates.
- **Example:**  `session.state['app:global_discount_code'] = 'SAVE10'`

- **`temp:` Prefix (Temporary Invocation State):**

- **Scope:** Specific to the current **invocation** (the entire process from an agent receiving user input to generating the final output for that input).
- **Persistence:**  **Not Persistent.** Discarded after the invocation completes and does not carry over to the next one.
- **Use Cases:** Storing intermediate calculations, flags, or data passed between tool calls within a single invocation.
- **When Not to Use:** For information that must persist across different invocations, such as user preferences, conversation history summaries, or accumulated data.
- **Example:**  `session.state['temp:raw_api_response'] = {...}`

Sub-Agents and Invocation Context

When a parent agent calls a sub-agent (e.g., using `SequentialAgent` or `ParallelAgent` ), it passes its `InvocationContext` to the sub-agent. This means the entire chain of agent calls shares the same invocation ID and, therefore, the same `temp:` state.

**How the Agent Sees It:** Your agent code interacts with the *combined* state through the single `session.state` collection (dict/ Map). The `SessionService` handles fetching/merging state from the correct underlying storage based on prefixes.

### Accessing Session State in Agent Instructions

When working with `LlmAgent` instances, you can directly inject session state values into the agent's instruction string using a simple templating syntax. This allows you to create dynamic and context-aware instructions without relying solely on natural language directives.

#### Using {key} Templating

To inject a value from the session state, enclose the key of the desired state variable within curly braces: `{key}` . The framework will automatically replace this placeholder with the corresponding value from `session.state` before passing the instruction to the LLM.

**Example:**

```
from google.adk.agents import LlmAgent

story_generator = LlmAgent(
    name="StoryGenerator",
    model="gemini-2.5-flash",
    instruction="""Write a short story about a cat, focusing on the theme: {topic}."""
)

# Assuming session.state['topic'] is set to "friendship", the LLM
# will receive the following instruction:
# "Write a short story about a cat, focusing on the theme: friendship."
```

#### Important Considerations

- Key Existence: Ensure that the key you reference in the instruction string exists in the session.state. If the key is missing, the agent will throw an error. To use a key that may or may not be present, you can include a question mark (?) after the key (e.g. {topic?}).
- Data Types: The value associated with the key should be a string or a type that can be easily converted to a string.
- Escaping: If you need to use literal curly braces in your instruction (e.g., for JSON formatting), you'll need to escape them.

#### Bypassing State Injection with InstructionProvider

In some cases, you might want to use `{{` and `}}` literally in your instructions without triggering the state injection mechanism. For example, you might be writing instructions for an agent that helps with a templating language that uses the same syntax.

To achieve this, you can provide a function to the `instruction` parameter instead of a string. This function is called an `InstructionProvider` . When you use an `InstructionProvider` , the ADK will not attempt to inject state, and your instruction string will be passed to the model as-is.

The `InstructionProvider` function receives a `ReadonlyContext` object, which you can use to access session state or other contextual information if you need to build the instruction dynamically.

```
from google.adk.agents import LlmAgent
from google.adk.agents.readonly_context import ReadonlyContext

# This is an InstructionProvider
def my_instruction_provider(context: ReadonlyContext) -> str:
    # You can optionally use the context to build the instruction
    # For this example, we'll return a static string with literal braces.
    return "This is an instruction with {{literal_braces}} that will not be replaced."

agent = LlmAgent(
    model="gemini-2.5-flash",
    name="template_helper_agent",
    instruction=my_instruction_provider
)
```

If you want to both use an `InstructionProvider`  *and* inject state into your instructions, you can use the `inject_session_state` utility function.

```
from google.adk.agents import LlmAgent
from google.adk.agents.readonly_context import ReadonlyContext
from google.adk.utils import instructions_utils

async def my_dynamic_instruction_provider(context: ReadonlyContext) -> str:
    template = "This is a {adjective} instruction with {{literal_braces}}."
    # This will inject the 'adjective' state variable but leave the literal braces.
    return await instructions_utils.inject_session_state(template, context)

agent = LlmAgent(
    model="gemini-2.5-flash",
    name="dynamic_template_helper_agent",
    instruction=my_dynamic_instruction_provider
)
```

**Benefits of Direct Injection**

- Clarity: Makes it explicit which parts of the instruction are dynamic and based on session state.
- Reliability: Avoids relying on the LLM to correctly interpret natural language instructions to access state.
- Maintainability: Simplifies instruction strings and reduces the risk of errors when updating state variable names.

**Relation to Other State Access Methods**

This direct injection method is specific to LlmAgent instructions. Refer to the following section for more information on other state access methods.

### How State is Updated: Recommended Methods

The Right Way to Modify State

When you need to change the session state, the correct and safest method is to **directly modify the `state` object on the `Context`** provided to your function (e.g., `callback_context.state['my_key'] = 'new_value'` ). This is considered "direct state manipulation" in the right way, as the framework automatically tracks these changes.

This is critically different from directly modifying the `state` on a `Session` object you retrieve from the `SessionService` (e.g., `my_session.state['my_key'] = 'new_value'` ). **You should avoid this** , as it bypasses the ADK's event tracking and can lead to lost data. The "Warning" section at the end of this page has more details on this important distinction.

State should **always** be updated as part of adding an `Event` to the session history using `session_service.append_event()` . This ensures changes are tracked, persistence works correctly, and updates are thread-safe.

**1. The Easy Way: `output_key` (for Agent Text Responses)**

This is the simplest method for saving an agent's final text response directly into the state. When defining your `LlmAgent` , specify the `output_key` :

```
from google.adk.agents import LlmAgent
from google.adk.sessions import InMemorySessionService, Session
from google.adk.runners import Runner
from google.genai.types import Content, Part

# Define agent with output_key
greeting_agent = LlmAgent(
    name="Greeter",
    model="gemini-2.5-flash", # Use a valid model
    instruction="Generate a short, friendly greeting.",
    output_key="last_greeting" # Save response to state['last_greeting']
)

# --- Setup Runner and Session ---
app_name, user_id, session_id = "state_app", "user1", "session1"
session_service = InMemorySessionService()
runner = Runner(
    agent=greeting_agent,
    app_name=app_name,
    session_service=session_service
)
session = await session_service.create_session(app_name=app_name,
                                    user_id=user_id,
                                    session_id=session_id)
print(f"Initial state: {session.state}")

# --- Run the Agent ---
# Runner handles calling append_event, which uses the output_key
# to automatically create the state_delta.
user_message = Content(parts=[Part(text="Hello")])
for event in runner.run(user_id=user_id,
                        session_id=session_id,
                        new_message=user_message):
    if event.is_final_response():
      print(f"Agent responded.") # Response text is also in event.content

# --- Check Updated State ---
updated_session = await session_service.get_session(app_name=APP_NAME, user_id=USER_ID, session_id=session_id)
print(f"State after agent run: {updated_session.state}")
# Expected output might include: {'last_greeting': 'Hello there! How can I help you today?'}
```

Behind the scenes, the `Runner` uses the `output_key` to create the necessary `EventActions` with a `state_delta` and calls `append_event` .

**2. The Standard Way: `EventActions.state_delta` (for Complex Updates)**

For more complex scenarios (updating multiple keys, non-string values, specific scopes like `user:` or `app:` , or updates not tied directly to the agent's final text), you manually construct the `state_delta` within `EventActions` .

```
from google.adk.sessions import InMemorySessionService, Session
from google.adk.events import Event, EventActions
from google.genai.types import Part, Content
import time

# --- Setup ---
session_service = InMemorySessionService()
app_name, user_id, session_id = "state_app_manual", "user2", "session2"
session = await session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id,
    state={"user:login_count": , "task_status": "idle"}
)
print(f"Initial state: {session.state}")

# --- Define State Changes ---
current_time = time.time()
state_changes = {
    "task_status": "active",              # Update session state
    "user:login_count": session.state.get("user:login_count", 0) + 1, # Update user state
    "user:last_login_ts": current_time,   # Add user state
    "temp:validation_needed": True        # Add temporary state (will be discarded)
}

# --- Create Event with Actions ---
actions_with_update = EventActions(state_delta=state_changes)
# This event might represent an internal system action, not just an agent response
system_event = Event(
    invocation_id="inv_login_update",
    author="system", # Or 'agent', 'tool' etc.
    actions=actions_with_update,
    timestamp=current_time
    # content might be None or represent the action taken
)

# --- Append the Event (This updates the state) ---
await session_service.append_event(session, system_event)
print("`append_event` called with explicit state delta.")

# --- Check Updated State ---
updated_session = await session_service.get_session(app_name=app_name,
                                            user_id=user_id,
                                            session_id=session_id)
print(f"State after event: {updated_session.state}")
# Expected: {'user:login_count': 1, 'task_status': 'active', 'user:last_login_ts': <timestamp>}
# Note: 'temp:validation_needed' is NOT present.
```

**3. Via `CallbackContext` or `ToolContext` (Recommended for Callbacks and Tools)**

Modifying state within agent callbacks (e.g., `on_before_agent_call` , `on_after_agent_call` ) or tool functions is best done using the `state` attribute of the `CallbackContext` or `ToolContext` provided to your function.

- `callback_context.state['my_key'] = my_value`
- `tool_context.state['my_key'] = my_value`

These context objects are specifically designed to manage state changes within their respective execution scopes. When you modify `context.state` , the ADK framework ensures that these changes are automatically captured and correctly routed into the `EventActions.state_delta` for the event being generated by the callback or tool. This delta is then processed by the `SessionService` when the event is appended, ensuring proper persistence and tracking.

This method abstracts away the manual creation of `EventActions` and `state_delta` for most common state update scenarios within callbacks and tools, making your code cleaner and less error-prone.

For more comprehensive details on context objects, refer to the Context documentation .

```
# In an agent callback or tool function
from google.adk.agents import CallbackContext # or ToolContext

def my_callback_or_tool_function(context: CallbackContext, # Or ToolContext
                                 # ... other parameters ...
                                ):
    # Update existing state
    count = context.state.get("user_action_count", )
    context.state["user_action_count"] = count + 1

    # Add new state
    context.state["temp:last_operation_status"] = "success"

    # State changes are automatically part of the event's state_delta
    # ... rest of callback/tool logic ...
```

**What `append_event` Does:**

- Adds the `Event` to `session.events` .
- Reads the `state_delta` from the event's `actions` .
- Applies these changes to the state managed by the `SessionService` , correctly handling prefixes and persistence based on the service type.
- Updates the session's `last_update_time` .
- Ensures thread-safety for concurrent updates.

### ⚠️ A Warning About Direct State Modification

Avoid directly modifying the `session.state` collection (dictionary/Map) on a `Session` object that was obtained directly from the `SessionService` (e.g., via `session_service.get_session()` or `session_service.create_session()` ) *outside* of the managed lifecycle of an agent invocation (i.e., not through a `CallbackContext` or `ToolContext` ). For example, code like `retrieved_session = await session_service.get_session(...); retrieved_session.state['key'] = value` is problematic.

State modifications *within* callbacks or tools using `CallbackContext.state` or `ToolContext.state` are the correct way to ensure changes are tracked, as these context objects handle the necessary integration with the event system.

**Why direct modification (outside of contexts) is strongly discouraged:**

1. **Bypasses Event History:** The change isn't recorded as an `Event` , losing auditability.
2. **Breaks Persistence:** Changes made this way **will likely NOT be saved** by `DatabaseSessionService` or `VertexAiSessionService` . They rely on `append_event` to trigger saving.
3. **Not Thread-Safe:** Can lead to race conditions and lost updates.
4. **Ignores Timestamps/Logic:** Doesn't update `last_update_time` or trigger related event logic.

**Recommendation:** Stick to updating state via `output_key` , `EventActions.state_delta` (when manually creating events), or by modifying the `state` property of `CallbackContext` or `ToolContext` objects when within their respective scopes. These methods ensure reliable, trackable, and persistent state management. Use direct access to `session.state` (from a `SessionService` -retrieved session) only for *reading* state.

### Best Practices for State Design Recap

- **Minimalism:** Store only essential, dynamic data.
- **Serialization:** Use basic, serializable types.
- **Descriptive Keys & Prefixes:** Use clear names and appropriate prefixes ( `user:` , `app:` , `temp:` , or none).
- **Shallow Structures:** Avoid deep nesting where possible.
- **Standard Update Flow:** Rely on `append_event` .

-----------------

# Memory: Long-Term Knowledge with MemoryService

We've seen how `Session` tracks the history ( `events` ) and temporary data ( `state` ) for a *single, ongoing conversation* . But what if an agent needs to recall information from *past* conversations? This is where the concept of **Long-Term Knowledge** and the **`MemoryService`** come into play.

Think of it this way:

- **`Session` / `State` :** Like your short-term memory during one specific chat.
- **Long-Term Knowledge ( `MemoryService` )** : Like a searchable archive or knowledge library the agent can consult, potentially containing information from many past chats or other sources.

## The MemoryService Role

The `BaseMemoryService` defines the interface for managing this searchable, long-term knowledge store. Its primary responsibilities are:

1. **Ingesting Information ( `add_session_to_memory` ):** Taking the contents of a (usually completed) `Session` and adding relevant information to the long-term knowledge store.
2. **Searching Information ( `search_memory` ):** Allowing an agent (typically via a `Tool` ) to query the knowledge store and retrieve relevant snippets or context based on a search query.

## Choosing the Right Memory Service

The ADK offers two distinct `MemoryService` implementations, each tailored to different use cases. Use the table below to decide which is the best fit for your agent.

| Feature | InMemoryMemoryService | VertexAiMemoryBankService |
| --- | --- | --- |
| Persistence | None (data is lost on restart) | Yes (Managed by Vertex AI) |
| Primary Use Case | Prototyping, local development, and simple testing. | Building meaningful, evolving memories from user conversations. |
| Memory Extraction | Stores full conversation | Extracts meaningful information from conversations and consolidates it with existing memories (powered by LLM) |
| Search Capability | Basic keyword matching. | Advanced semantic search. |
| Setup Complexity | None. It's the default. | Low. Requires an Agent Engine instance in Vertex AI. |
| Dependencies | None. | Google Cloud Project, Vertex AI API |
| When to use it | When you want to search across multiple sessions’ chat histories for prototyping. | When you want your agent to remember and learn from past interactions. |

## In-Memory Memory

The `InMemoryMemoryService` stores session information in the application's memory and performs basic keyword matching for searches. It requires no setup and is best for prototyping and simple testing scenarios where persistence isn't required.

```
from google.adk.memory import InMemoryMemoryService
memory_service = InMemoryMemoryService()
```

**Example: Adding and Searching Memory**

This example demonstrates the basic flow using the `InMemoryMemoryService` for simplicity.

Full Code
```
import asyncio
from google.adk.agents import LlmAgent
from google.adk.sessions import InMemorySessionService, Session
from google.adk.memory import InMemoryMemoryService # Import MemoryService
from google.adk.runners import Runner
from google.adk.tools import load_memory # Tool to query memory
from google.genai.types import Content, Part

# --- Constants ---
APP_NAME = "memory_example_app"
USER_ID = "mem_user"
MODEL = "gemini-2.5-flash" # Use a valid model

# --- Agent Definitions ---
# Agent 1: Simple agent to capture information
info_capture_agent = LlmAgent(
    model=MODEL,
    name="InfoCaptureAgent",
    instruction="Acknowledge the user's statement.",
)

# Agent 2: Agent that can use memory
memory_recall_agent = LlmAgent(
    model=MODEL,
    name="MemoryRecallAgent",
    instruction="Answer the user's question. Use the 'load_memory' tool "
                "if the answer might be in past conversations.",
    tools=[load_memory] # Give the agent the tool
)

# --- Services ---
# Services must be shared across runners to share state and memory
session_service = InMemorySessionService()
memory_service = InMemoryMemoryService() # Use in-memory for demo

async def run_scenario():
    # --- Scenario ---

    # Turn 1: Capture some information in a session
    print("--- Turn 1: Capturing Information ---")
    runner1 = Runner(
        # Start with the info capture agent
        agent=info_capture_agent,
        app_name=APP_NAME,
        session_service=session_service,
        memory_service=memory_service # Provide the memory service to the Runner
    )
    session1_id = "session_info"
    await runner1.session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=session1_id)
    user_input1 = Content(parts=[Part(text="My favorite project is Project Alpha.")], role="user")

    # Run the agent
    final_response_text = "(No final response)"
    async for event in runner1.run_async(user_id=USER_ID, session_id=session1_id, new_message=user_input1):
        if event.is_final_response() and event.content and event.content.parts:
            final_response_text = event.content.parts[].text
    print(f"Agent 1 Response: {final_response_text}")

    # Get the completed session
    completed_session1 = await runner1.session_service.get_session(app_name=APP_NAME, user_id=USER_ID, session_id=session1_id)

    # Add this session's content to the Memory Service
    print("\n--- Adding Session 1 to Memory ---")
    await memory_service.add_session_to_memory(completed_session1)
    print("Session added to memory.")

    # Turn 2: Recall the information in a new session
    print("\n--- Turn 2: Recalling Information ---")
    runner2 = Runner(
        # Use the second agent, which has the memory tool
        agent=memory_recall_agent,
        app_name=APP_NAME,
        session_service=session_service, # Reuse the same service
        memory_service=memory_service   # Reuse the same service
    )
    session2_id = "session_recall"
    await runner2.session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=session2_id)
    user_input2 = Content(parts=[Part(text="What is my favorite project?")], role="user")

    # Run the second agent
    final_response_text_2 = "(No final response)"
    async for event in runner2.run_async(user_id=USER_ID, session_id=session2_id, new_message=user_input2):
        if event.is_final_response() and event.content and event.content.parts:
            final_response_text_2 = event.content.parts[0].text
    print(f"Agent 2 Response: {final_response_text_2}")

# To run this example, you can use the following snippet:
# asyncio.run(run_scenario())

# await run_scenario()
```

## Using Memory in Your Agent

When a memory service is configured, your agent can use a tool or callback to retrieve memories. ADK includes two pre-built tools for retrieving memories:

- `PreloadMemory` : Always retrieve memory at the beginning of each turn (similar to a callback).
- `LoadMemory` : Retrieve memory when your agent decides it would be helpful.

**Example:**

```
from google.adk.agents import Agent
from google.adk.tools.preload_memory_tool import PreloadMemoryTool

agent = Agent(
    model=MODEL_ID,
    name='weather_sentiment_agent',
    instruction="...",
    tools=[PreloadMemoryTool()]
)
```

To extract memories from your session, you need to call `add_session_to_memory` . For example, you can automate this via a callback:

```
from google import adk

async def auto_save_session_to_memory_callback(callback_context):
    await callback_context._invocation_context.memory_service.add_session_to_memory(
        callback_context._invocation_context.session)

agent = Agent(
    model=MODEL,
    name="Generic_QA_Agent",
    instruction="Answer the user's questions",
    tools=[adk.tools.preload_memory_tool.PreloadMemoryTool()],
    after_agent_callback=auto_save_session_to_memory_callback,
)
```

## Advanced Concepts

### How Memory Works in Practice

The memory workflow internally involves these steps:

1. **Session Interaction:** A user interacts with an agent via a `Session` , managed by a `SessionService` . Events are added, and state might be updated.
2. **Ingestion into Memory:** At some point (often when a session is considered complete or has yielded significant information), your application calls `memory_service.add_session_to_memory(session)` . This extracts relevant information from the session's events and adds it to the long-term knowledge store (in-memory dictionary or Agent Engine Memory Bank).
3. **Later Query:** In a *different* (or the same) session, the user might ask a question requiring past context (e.g., "What did we discuss about project X last week?").
4. **Agent Uses Memory Tool:** An agent equipped with a memory-retrieval tool (like the built-in `load_memory` tool) recognizes the need for past context. It calls the tool, providing a search query (e.g., "discussion project X last week").
5. **Search Execution:** The tool internally calls `memory_service.search_memory(app_name, user_id, query)` .
6. **Results Returned:** The `MemoryService` searches its store (using keyword matching or semantic search) and returns relevant snippets as a `SearchMemoryResponse` containing a list of `MemoryResult` objects (each potentially holding events from a relevant past session).
7. **Agent Uses Results:** The tool returns these results to the agent, usually as part of the context or function response. The agent can then use this retrieved information to formulate its final answer to the user.

### Can an agent have access to more than one memory service?

- **Through Standard Configuration: No.** The framework ( `adk web` , `adk api_server` ) is designed to be configured with one single memory service at a time via the `--memory_service_uri` flag. This single service is then provided to the agent and accessed through the built-in `self.search_memory()` method. From a configuration standpoint, you can only choose one backend ( `InMemory` , `VertexAiMemoryBankService` ) for all agents served by that process.
- **Within Your Agent's Code: Yes, absolutely.** There is nothing preventing you from manually importing and instantiating another memory service directly inside your agent's code. This allows you to access multiple memory sources within a single agent turn.

For example, your agent could use the framework-configured `VertexAiMemoryBankService` to recall conversational history, and also manually instantiate a `InMemoryMemoryService` to look up information in a technical manual.

#### Example: Using Two Memory Services

Here’s how you could implement that in your agent's code:

```
from google.adk.agents import Agent
from google.adk.memory import InMemoryMemoryService, VertexAiMemoryBankService
from google.genai import types

class MultiMemoryAgent(Agent):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.memory_service = InMemoryMemoryService()
        # Manually instantiate a second memory service for document lookups
        self.vertexai_memorybank_service = VertexAiMemoryBankService(
            project="PROJECT_ID",
            location="LOCATION",
            agent_engine_id="AGENT_ENGINE_ID"
        )

    async def run(self, request: types.Content, **kwargs) -> types.Content:
        user_query = request.parts[].text

        # 1. Search conversational history using the framework-provided memory
        #    (This would be InMemoryMemoryService if configured)
        conversation_context = await self.memory_service.search_memory(query=user_query)

        # 2. Search the document knowledge base using the manually created service
        document_context = await self.vertexai_memorybank_service.search_memory(query=user_query)

        # Combine the context from both sources to generate a better response
        prompt = "From our past conversations, I remember:\n"
        prompt += f"{conversation_context.memories}\n\n"
        prompt += "From the technical manuals, I found:\n"
        prompt += f"{document_context.memories}\n\n"
        prompt += f"Based on all this, here is my answer to '{user_query}':"

        return await self.llm.generate_content_async(prompt)
```

-----------------

# Callbacks: Observe, Customize, and Control Agent Behavior

## Introduction: What are Callbacks and Why Use Them?

Callbacks are a cornerstone feature of ADK, providing a powerful mechanism to hook into an agent's execution process. They allow you to observe, customize, and even control the agent's behavior at specific, predefined points without modifying the core ADK framework code.

**What are they?** In essence, callbacks are standard functions that you define. You then associate these functions with an agent when you create it. The ADK framework automatically calls your functions at key stages, letting you observe or intervene. Think of it like checkpoints during the agent's process:

- **Before the agent starts its main work on a request, and after it finishes:** When you ask an agent to do something (e.g., answer a question), it runs its internal logic to figure out the response.
- The `Before Agent` callback executes *right before* this main work begins for that specific request.
- The `After Agent` callback executes *right after* the agent has finished all its steps for that request and has prepared the final result, but just before the result is returned.
- This "main work" encompasses the agent's *entire* process for handling that single request. This might involve deciding to call an LLM, actually calling the LLM, deciding to use a tool, using the tool, processing the results, and finally putting together the answer. These callbacks essentially wrap the whole sequence from receiving the input to producing the final output for that one interaction.
- **Before sending a request to, or after receiving a response from, the Large Language Model (LLM):** These callbacks ( `Before Model` , `After Model` ) allow you to inspect or modify the data going to and coming from the LLM specifically.
- **Before executing a tool (like a Python function or another agent) or after it finishes:** Similarly, `Before Tool` and `After Tool` callbacks give you control points specifically around the execution of tools invoked by the agent.

**Why use them?** Callbacks unlock significant flexibility and enable advanced agent capabilities:

- **Observe & Debug:** Log detailed information at critical steps for monitoring and troubleshooting.
- **Customize & Control:** Modify data flowing through the agent (like LLM requests or tool results) or even bypass certain steps entirely based on your logic.
- **Implement Guardrails:** Enforce safety rules, validate inputs/outputs, or prevent disallowed operations.
- **Manage State:** Read or dynamically update the agent's session state during execution.
- **Integrate & Enhance:** Trigger external actions (API calls, notifications) or add features like caching.

Tip

When implementing security guardrails and policies, use ADK Plugins for
better modularity and flexibility than Callbacks. For more details, see Callbacks and Plugins for Security Guardrails .

**How are they added:**

Code
```
from google.adk.agents import LlmAgent
from google.adk.agents.callback_context import CallbackContext
from google.adk.models import LlmResponse, LlmRequest
from typing import Optional

# --- Define your callback function ---
def my_before_model_logic(
    callback_context: CallbackContext, llm_request: LlmRequest
) -> Optional[LlmResponse]:
    print(f"Callback running before model call for agent: {callback_context.agent_name}")
    # ... your custom logic here ...
    return None # Allow the model call to proceed

# --- Register it during Agent creation ---
my_agent = LlmAgent(
    name="MyCallbackAgent",
    model="gemini-2.5-flash", # Or your desired model
    instruction="Be helpful.",
    # Other agent parameters...
    before_model_callback=my_before_model_logic # Pass the function here
)
```

## The Callback Mechanism: Interception and Control

When the ADK framework encounters a point where a callback can run (e.g., just before calling the LLM), it checks if you provided a corresponding callback function for that agent. If you did, the framework executes your function.

**Context is Key:** Your callback function isn't called in isolation. The framework provides special **context objects** ( `CallbackContext` or `ToolContext` ) as arguments. These objects contain vital information about the current state of the agent's execution, including the invocation details, session state, and potentially references to services like artifacts or memory. You use these context objects to understand the situation and interact with the framework. (See the dedicated "Context Objects" section for full details).

**Controlling the Flow (The Core Mechanism):** The most powerful aspect of callbacks lies in how their **return value** influences the agent's subsequent actions. This is how you intercept and control the execution flow:

1. **`return None` (Allow Default Behavior):**

- The specific return type can vary depending on the language. In Java, the equivalent return type is `Optional.empty()` . Refer to the API documentation for language specific guidance.
- This is the standard way to signal that your callback has finished its work (e.g., logging, inspection, minor modifications to *mutable* input arguments like `llm_request` ) and that the ADK agent should **proceed with its normal operation** .
- For `before_*` callbacks ( `before_agent` , `before_model` , `before_tool` ), returning `None` means the next step in the sequence (running the agent logic, calling the LLM, executing the tool) will occur.
- For `after_*` callbacks ( `after_agent` , `after_model` , `after_tool` ), returning `None` means the result just produced by the preceding step (the agent's output, the LLM's response, the tool's result) will be used as is.

2. **`return <Specific Object>` (Override Default Behavior):**

- Returning a *specific type of object* (instead of `None` ) is how you **override** the ADK agent's default behavior. The framework will use the object you return and *skip* the step that would normally follow or *replace* the result that was just generated.
- **`before_agent_callback` → `types.Content`** : Skips the agent's main execution logic ( `_run_async_impl` / `_run_live_impl` ). The returned `Content` object is immediately treated as the agent's final output for this turn. Useful for handling simple requests directly or enforcing access control.
- **`before_model_callback` → `LlmResponse`** : Skips the call to the external Large Language Model. The returned `LlmResponse` object is processed as if it were the actual response from the LLM. Ideal for implementing input guardrails, prompt validation, or serving cached responses.
- **`before_tool_callback` → `dict` or `Map`** : Skips the execution of the actual tool function (or sub-agent). The returned `dict` is used as the result of the tool call, which is then typically passed back to the LLM. Perfect for validating tool arguments, applying policy restrictions, or returning mocked/cached tool results.
- **`after_agent_callback` → `types.Content`** : *Replaces* the `Content` that the agent's run logic just produced.
- **`after_model_callback` → `LlmResponse`** : *Replaces* the `LlmResponse` received from the LLM. Useful for sanitizing outputs, adding standard disclaimers, or modifying the LLM's response structure.
- **`after_tool_callback` → `dict` or `Map`** : *Replaces* the `dict` result returned by the tool. Allows for post-processing or standardization of tool outputs before they are sent back to the LLM.

**Conceptual Code Example (Guardrail):**

This example demonstrates the common pattern for a guardrail using `before_model_callback` .

Code
```
from google.adk.agents import LlmAgent
from google.adk.agents.callback_context import CallbackContext
from google.adk.models import LlmResponse, LlmRequest
from google.adk.runners import Runner
from typing import Optional
from google.genai import types
from google.adk.sessions import InMemorySessionService

GEMINI_2_FLASH="gemini-2.5-flash"

# --- Define the Callback Function ---
def simple_before_model_modifier(
    callback_context: CallbackContext, llm_request: LlmRequest
) -> Optional[LlmResponse]:
    """Inspects/modifies the LLM request or skips the call."""
    agent_name = callback_context.agent_name
    print(f"[Callback] Before model call for agent: {agent_name}")

    # Inspect the last user message in the request contents
    last_user_message = ""
    if llm_request.contents and llm_request.contents[-].role == 'user':
         if llm_request.contents[-1].parts:
            last_user_message = llm_request.contents[-1].parts[0].text
    print(f"[Callback] Inspecting last user message: '{last_user_message}'")

    # --- Modification Example ---
    # Add a prefix to the system instruction
    original_instruction = llm_request.config.system_instruction or types.Content(role="system", parts=[])
    prefix = "[Modified by Callback] "
    # Ensure system_instruction is Content and parts list exists
    if not isinstance(original_instruction, types.Content):
         # Handle case where it might be a string (though config expects Content)
         original_instruction = types.Content(role="system", parts=[types.Part(text=str(original_instruction))])
    if not original_instruction.parts:
        original_instruction.parts.append(types.Part(text="")) # Add an empty part if none exist

    # Modify the text of the first part
    modified_text = prefix + (original_instruction.parts[0].text or "")
    original_instruction.parts[0].text = modified_text
    llm_request.config.system_instruction = original_instruction
    print(f"[Callback] Modified system instruction to: '{modified_text}'")

    # --- Skip Example ---
    # Check if the last user message contains "BLOCK"
    if "BLOCK" in last_user_message.upper():
        print("[Callback] 'BLOCK' keyword found. Skipping LLM call.")
        # Return an LlmResponse to skip the actual LLM call
        return LlmResponse(
            content=types.Content(
                role="model",
                parts=[types.Part(text="LLM call was blocked by before_model_callback.")],
            )
        )
    else:
        print("[Callback] Proceeding with LLM call.")
        # Return None to allow the (modified) request to go to the LLM
        return None

# Create LlmAgent and Assign Callback
my_llm_agent = LlmAgent(
        name="ModelCallbackAgent",
        model=GEMINI_2_FLASH,
        instruction="You are a helpful assistant.", # Base instruction
        description="An LLM agent demonstrating before_model_callback",
        before_model_callback=simple_before_model_modifier # Assign the function here
)

APP_NAME = "guardrail_app"
USER_ID = "user_1"
SESSION_ID = "session_001"

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=my_llm_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[0].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("write a joke on BLOCK")
```

By understanding this mechanism of returning `None` versus returning specific objects, you can precisely control the agent's execution path, making callbacks an essential tool for building sophisticated and reliable agents with ADK.

-----------------

# Types of Callbacks

The framework provides different types of callbacks that trigger at various stages of an agent's execution. Understanding when each callback fires and what context it receives is key to using them effectively.

## Agent Lifecycle Callbacks

These callbacks are available on *any* agent that inherits from `BaseAgent` (including `LlmAgent` , `SequentialAgent` , `ParallelAgent` , `LoopAgent` , etc).

Note

The specific method names or return types may vary slightly by SDK language (e.g., return `None` in Python, return `Optional.empty()` or `Maybe.empty()` in Java). Refer to the language-specific API documentation for details.

### Before Agent Callback

**When:** Called *immediately before* the agent's `_run_async_impl` (or `_run_live_impl` ) method is executed. It runs after the agent's `InvocationContext` is created but *before* its core logic begins.

**Purpose:** Ideal for setting up resources or state needed only for this specific agent's run, performing validation checks on the session state (callback_context.state) before execution starts, logging the entry point of the agent's activity, or potentially modifying the invocation context before the core logic uses it.

Code
```

# # --- Setup Instructions ---
# # 1. Install the ADK package:
# !pip install google-adk
# # Make sure to restart kernel if using colab/jupyter notebooks

# # 2. Set up your Gemini API Key:
# #    - Get a key from Google AI Studio: https://aistudio.google.com/app/apikey
# #    - Set it as an environment variable:
# import os
# os.environ["GOOGLE_API_KEY"] = "YOUR_API_KEY_HERE" # <--- REPLACE with your actual key
# # Or learn about other authentication methods (like Vertex AI):
# # https://google.github.io/adk-docs/agents/models/

# ADK Imports
from google.adk.agents import LlmAgent
from google.adk.agents.callback_context import CallbackContext
from google.adk.runners import InMemoryRunner # Use InMemoryRunner
from google.genai import types # For types.Content
from typing import Optional

# Define the model - Use the specific model name requested
GEMINI_2_FLASH="gemini-2.5-flash"

# --- 1. Define the Callback Function ---
def check_if_agent_should_run(callback_context: CallbackContext) -> Optional[types.Content]:
    """
    Logs entry and checks 'skip_llm_agent' in session state.
    If True, returns Content to skip the agent's execution.
    If False or not present, returns None to allow execution.
    """
    agent_name = callback_context.agent_name
    invocation_id = callback_context.invocation_id
    current_state = callback_context.state.to_dict()

    print(f"\n[Callback] Entering agent: {agent_name} (Inv: {invocation_id})")
    print(f"[Callback] Current State: {current_state}")

    # Check the condition in session state dictionary
    if current_state.get("skip_llm_agent", False):
        print(f"[Callback] State condition 'skip_llm_agent=True' met: Skipping agent {agent_name}.")
        # Return Content to skip the agent's run
        return types.Content(
            parts=[types.Part(text=f"Agent {agent_name} skipped by before_agent_callback due to state.")],
            role="model" # Assign model role to the overriding response
        )
    else:
        print(f"[Callback] State condition not met: Proceeding with agent {agent_name}.")
        # Return None to allow the LlmAgent's normal execution
        return None

# --- 2. Setup Agent with Callback ---
llm_agent_with_before_cb = LlmAgent(
    name="MyControlledAgent",
    model=GEMINI_2_FLASH,
    instruction="You are a concise assistant.",
    description="An LLM agent demonstrating stateful before_agent_callback",
    before_agent_callback=check_if_agent_should_run # Assign the callback
)

# --- 3. Setup Runner and Sessions using InMemoryRunner ---
async def main():
    app_name = "before_agent_demo"
    user_id = "test_user"
    session_id_run = "session_will_run"
    session_id_skip = "session_will_skip"

    # Use InMemoryRunner - it includes InMemorySessionService
    runner = InMemoryRunner(agent=llm_agent_with_before_cb, app_name=app_name)
    # Get the bundled session service to create sessions
    session_service = runner.session_service

    # Create session 1: Agent will run (default empty state)
    session_service.create_session(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id_run
        # No initial state means 'skip_llm_agent' will be False in the callback check
    )

    # Create session 2: Agent will be skipped (state has skip_llm_agent=True)
    session_service.create_session(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id_skip,
        state={"skip_llm_agent": True} # Set the state flag here
    )

    # --- Scenario 1: Run where callback allows agent execution ---
    print("\n" + "="* + f" SCENARIO 1: Running Agent on Session '{session_id_run}' (Should Proceed) " + "="*20)
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id_run,
        new_message=types.Content(role="user", parts=[types.Part(text="Hello, please respond.")])
    ):
        # Print final output (either from LLM or callback override)
        if event.is_final_response() and event.content:
            print(f"Final Output: [{event.author}] {event.content.parts[0].text.strip()}")
        elif event.is_error():
             print(f"Error Event: {event.error_details}")

    # --- Scenario 2: Run where callback intercepts and skips agent ---
    print("\n" + "="*20 + f" SCENARIO 2: Running Agent on Session '{session_id_skip}' (Should Skip) " + "="*20)
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id_skip,
        new_message=types.Content(role="user", parts=[types.Part(text="This message won't reach the LLM.")])
    ):
         # Print final output (either from LLM or callback override)
         if event.is_final_response() and event.content:
            print(f"Final Output: [{event.author}] {event.content.parts[0].text.strip()}")
         elif event.is_error():
             print(f"Error Event: {event.error_details}")

# --- 4. Execute ---
# In a Python script:
# import asyncio
# if __name__ == "__main__":
#     # Make sure GOOGLE_API_KEY environment variable is set if not using Vertex AI auth
#     # Or ensure Application Default Credentials (ADC) are configured for Vertex AI
#     asyncio.run(main())

# In a Jupyter Notebook or similar environment:
await main()
```

**Note on the `before_agent_callback` Example:**

- **What it Shows:** This example demonstrates the `before_agent_callback` . This callback runs *right before* the agent's main processing logic starts for a given request.
- **How it Works:** The callback function ( `check_if_agent_should_run` ) looks at a flag ( `skip_llm_agent` ) in the session's state.
- If the flag is `True` , the callback returns a `types.Content` object. This tells the ADK framework to **skip** the agent's main execution entirely and use the callback's returned content as the final response.
- If the flag is `False` (or not set), the callback returns `None` or an empty object. This tells the ADK framework to **proceed** with the agent's normal execution (calling the LLM in this case).

- **Expected Outcome:** You'll see two scenarios:
1. In the session *with* the `skip_llm_agent: True` state, the agent's LLM call is bypassed, and the output comes directly from the callback ("Agent... skipped...").
2. In the session *without* that state flag, the callback allows the agent to run, and you see the actual response from the LLM (e.g., "Hello!").

- **Understanding Callbacks:** This highlights how `before_` callbacks act as **gatekeepers** , allowing you to intercept execution *before* a major step and potentially prevent it based on checks (like state, input validation, permissions).

### After Agent Callback

**When:** Called *immediately after* the agent's `_run_async_impl` (or `_run_live_impl` ) method successfully completes. It does *not* run if the agent was skipped due to `before_agent_callback` returning content or if `end_invocation` was set during the agent's run.

**Purpose:** Useful for cleanup tasks, post-execution validation, logging the completion of an agent's activity, modifying final state, or augmenting/replacing the agent's final output.

Code
```

# # --- Setup Instructions ---
# # 1. Install the ADK package:
# !pip install google-adk
# # Make sure to restart kernel if using colab/jupyter notebooks

# # 2. Set up your Gemini API Key:
# #    - Get a key from Google AI Studio: https://aistudio.google.com/app/apikey
# #    - Set it as an environment variable:
# import os
# os.environ["GOOGLE_API_KEY"] = "YOUR_API_KEY_HERE" # <--- REPLACE with your actual key
# # Or learn about other authentication methods (like Vertex AI):
# # https://google.github.io/adk-docs/agents/models/

# ADK Imports
from google.adk.agents import LlmAgent
from google.adk.agents.callback_context import CallbackContext
from google.adk.runners import InMemoryRunner # Use InMemoryRunner
from google.genai import types # For types.Content
from typing import Optional

# Define the model - Use the specific model name requested
GEMINI_2_FLASH="gemini-2.5-flash"

# --- 1. Define the Callback Function ---
def modify_output_after_agent(callback_context: CallbackContext) -> Optional[types.Content]:
    """
    Logs exit from an agent and checks 'add_concluding_note' in session state.
    If True, returns new Content to *replace* the agent's original output.
    If False or not present, returns None, allowing the agent's original output to be used.
    """
    agent_name = callback_context.agent_name
    invocation_id = callback_context.invocation_id
    current_state = callback_context.state.to_dict()

    print(f"\n[Callback] Exiting agent: {agent_name} (Inv: {invocation_id})")
    print(f"[Callback] Current State: {current_state}")

    # Example: Check state to decide whether to modify the final output
    if current_state.get("add_concluding_note", False):
        print(f"[Callback] State condition 'add_concluding_note=True' met: Replacing agent {agent_name}'s output.")
        # Return Content to *replace* the agent's own output
        return types.Content(
            parts=[types.Part(text=f"Concluding note added by after_agent_callback, replacing original output.")],
            role="model" # Assign model role to the overriding response
        )
    else:
        print(f"[Callback] State condition not met: Using agent {agent_name}'s original output.")
        # Return None - the agent's output produced just before this callback will be used.
        return None

# --- 2. Setup Agent with Callback ---
llm_agent_with_after_cb = LlmAgent(
    name="MySimpleAgentWithAfter",
    model=GEMINI_2_FLASH,
    instruction="You are a simple agent. Just say 'Processing complete!'",
    description="An LLM agent demonstrating after_agent_callback for output modification",
    after_agent_callback=modify_output_after_agent # Assign the callback here
)

# --- 3. Setup Runner and Sessions using InMemoryRunner ---
async def main():
    app_name = "after_agent_demo"
    user_id = "test_user_after"
    session_id_normal = "session_run_normally"
    session_id_modify = "session_modify_output"

    # Use InMemoryRunner - it includes InMemorySessionService
    runner = InMemoryRunner(agent=llm_agent_with_after_cb, app_name=app_name)
    # Get the bundled session service to create sessions
    session_service = runner.session_service

    # Create session 1: Agent output will be used as is (default empty state)
    session_service.create_session(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id_normal
        # No initial state means 'add_concluding_note' will be False in the callback check
    )
    # print(f"Session '{session_id_normal}' created with default state.")

    # Create session 2: Agent output will be replaced by the callback
    session_service.create_session(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id_modify,
        state={"add_concluding_note": True} # Set the state flag here
    )
    # print(f"Session '{session_id_modify}' created with state={{'add_concluding_note': True}}.")

    # --- Scenario 1: Run where callback allows agent's original output ---
    print("\n" + "="* + f" SCENARIO 1: Running Agent on Session '{session_id_normal}' (Should Use Original Output) " + "="*20)
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id_normal,
        new_message=types.Content(role="user", parts=[types.Part(text="Process this please.")])
    ):
        # Print final output (either from LLM or callback override)
        if event.is_final_response() and event.content:
            print(f"Final Output: [{event.author}] {event.content.parts[0].text.strip()}")
        elif event.is_error():
             print(f"Error Event: {event.error_details}")

    # --- Scenario 2: Run where callback replaces the agent's output ---
    print("\n" + "="*20 + f" SCENARIO 2: Running Agent on Session '{session_id_modify}' (Should Replace Output) " + "="*20)
    async for event in runner.run_async(
        user_id=user_id,
        session_id=session_id_modify,
        new_message=types.Content(role="user", parts=[types.Part(text="Process this and add note.")])
    ):
         # Print final output (either from LLM or callback override)
         if event.is_final_response() and event.content:
            print(f"Final Output: [{event.author}] {event.content.parts[0].text.strip()}")
         elif event.is_error():
             print(f"Error Event: {event.error_details}")

# --- 4. Execute ---
# In a Python script:
# import asyncio
# if __name__ == "__main__":
#     # Make sure GOOGLE_API_KEY environment variable is set if not using Vertex AI auth
#     # Or ensure Application Default Credentials (ADC) are configured for Vertex AI
#     asyncio.run(main())

# In a Jupyter Notebook or similar environment:
await main()
```

**Note on the `after_agent_callback` Example:**

- **What it Shows:** This example demonstrates the `after_agent_callback` . This callback runs *right after* the agent's main processing logic has finished and produced its result, but *before* that result is finalized and returned.
- **How it Works:** The callback function ( `modify_output_after_agent` ) checks a flag ( `add_concluding_note` ) in the session's state.
- If the flag is `True` , the callback returns a *new*  `types.Content` object. This tells the ADK framework to **replace** the agent's original output with the content returned by the callback.
- If the flag is `False` (or not set), the callback returns `None` or an empty object. This tells the ADK framework to **use** the original output generated by the agent.

- **Expected Outcome:** You'll see two scenarios:
1. In the session *without* the `add_concluding_note: True` state, the callback allows the agent's original output ("Processing complete!") to be used.
2. In the session *with* that state flag, the callback intercepts the agent's original output and replaces it with its own message ("Concluding note added...").

- **Understanding Callbacks:** This highlights how `after_` callbacks allow **post-processing** or **modification** . You can inspect the result of a step (the agent's run) and decide whether to let it pass through, change it, or completely replace it based on your logic.

## LLM Interaction Callbacks

These callbacks are specific to `LlmAgent` and provide hooks around the interaction with the Large Language Model.

### Before Model Callback

**When:** Called just before the `generate_content_async` (or equivalent) request is sent to the LLM within an `LlmAgent` 's flow.

**Purpose:** Allows inspection and modification of the request going to the LLM. Use cases include adding dynamic instructions, injecting few-shot examples based on state, modifying model config, implementing guardrails (like profanity filters), or implementing request-level caching.

**Return Value Effect:**
If the callback returns `None` (or a `Maybe.empty()` object in Java), the LLM continues its normal workflow. If the callback returns an `LlmResponse` object, then the call to the LLM is **skipped** . The returned `LlmResponse` is used directly as if it came from the model. This is powerful for implementing guardrails or caching.

Code
```

from google.adk.agents import LlmAgent
from google.adk.agents.callback_context import CallbackContext
from google.adk.models import LlmResponse, LlmRequest
from google.adk.runners import Runner
from typing import Optional
from google.genai import types
from google.adk.sessions import InMemorySessionService

GEMINI_2_FLASH="gemini-2.5-flash"

# --- Define the Callback Function ---
def simple_before_model_modifier(
    callback_context: CallbackContext, llm_request: LlmRequest
) -> Optional[LlmResponse]:
    """Inspects/modifies the LLM request or skips the call."""
    agent_name = callback_context.agent_name
    print(f"[Callback] Before model call for agent: {agent_name}")

    # Inspect the last user message in the request contents
    last_user_message = ""
    if llm_request.contents and llm_request.contents[-].role == 'user':
         if llm_request.contents[-1].parts:
            last_user_message = llm_request.contents[-1].parts[0].text
    print(f"[Callback] Inspecting last user message: '{last_user_message}'")

    # --- Modification Example ---
    # Add a prefix to the system instruction
    original_instruction = llm_request.config.system_instruction or types.Content(role="system", parts=[])
    prefix = "[Modified by Callback] "
    # Ensure system_instruction is Content and parts list exists
    if not isinstance(original_instruction, types.Content):
         # Handle case where it might be a string (though config expects Content)
         original_instruction = types.Content(role="system", parts=[types.Part(text=str(original_instruction))])
    if not original_instruction.parts:
        original_instruction.parts.append(types.Part(text="")) # Add an empty part if none exist

    # Modify the text of the first part
    modified_text = prefix + (original_instruction.parts[0].text or "")
    original_instruction.parts[0].text = modified_text
    llm_request.config.system_instruction = original_instruction
    print(f"[Callback] Modified system instruction to: '{modified_text}'")

    # --- Skip Example ---
    # Check if the last user message contains "BLOCK"
    if "BLOCK" in last_user_message.upper():
        print("[Callback] 'BLOCK' keyword found. Skipping LLM call.")
        # Return an LlmResponse to skip the actual LLM call
        return LlmResponse(
            content=types.Content(
                role="model",
                parts=[types.Part(text="LLM call was blocked by before_model_callback.")],
            )
        )
    else:
        print("[Callback] Proceeding with LLM call.")
        # Return None to allow the (modified) request to go to the LLM
        return None

# Create LlmAgent and Assign Callback
my_llm_agent = LlmAgent(
        name="ModelCallbackAgent",
        model=GEMINI_2_FLASH,
        instruction="You are a helpful assistant.", # Base instruction
        description="An LLM agent demonstrating before_model_callback",
        before_model_callback=simple_before_model_modifier # Assign the function here
)

APP_NAME = "guardrail_app"
USER_ID = "user_1"
SESSION_ID = "session_001"

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=my_llm_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[0].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("write a joke on BLOCK")
```

### After Model Callback

**When:** Called just after a response ( `LlmResponse` ) is received from the LLM, before it's processed further by the invoking agent.

**Purpose:** Allows inspection or modification of the raw LLM response. Use cases include

- logging model outputs,
- reformatting responses,
- censoring sensitive information generated by the model,
- parsing structured data from the LLM response and storing it in `callback_context.state`
- or handling specific error codes.
Code
```

from google.adk.agents import LlmAgent
from google.adk.agents.callback_context import CallbackContext
from google.adk.runners import Runner
from typing import Optional
from google.genai import types
from google.adk.sessions import InMemorySessionService
from google.adk.models import LlmResponse

GEMINI_2_FLASH="gemini-2.5-flash"

# --- Define the Callback Function ---
def simple_after_model_modifier(
    callback_context: CallbackContext, llm_response: LlmResponse
) -> Optional[LlmResponse]:
    """Inspects/modifies the LLM response after it's received."""
    agent_name = callback_context.agent_name
    print(f"[Callback] After model call for agent: {agent_name}")

    # --- Inspection ---
    original_text = ""
    if llm_response.content and llm_response.content.parts:
        # Assuming simple text response for this example
        if llm_response.content.parts[].text:
            original_text = llm_response.content.parts[0].text
            print(f"[Callback] Inspected original response text: '{original_text[:100]}...'") # Log snippet
        elif llm_response.content.parts[0].function_call:
             print(f"[Callback] Inspected response: Contains function call '{llm_response.content.parts[0].function_call.name}'. No text modification.")
             return None # Don't modify tool calls in this example
        else:
             print("[Callback] Inspected response: No text content found.")
             return None
    elif llm_response.error_message:
        print(f"[Callback] Inspected response: Contains error '{llm_response.error_message}'. No modification.")
        return None
    else:
        print("[Callback] Inspected response: Empty LlmResponse.")
        return None # Nothing to modify

    # --- Modification Example ---
    # Replace "joke" with "funny story" (case-insensitive)
    search_term = "joke"
    replace_term = "funny story"
    if search_term in original_text.lower():
        print(f"[Callback] Found '{search_term}'. Modifying response.")
        modified_text = original_text.replace(search_term, replace_term)
        modified_text = modified_text.replace(search_term.capitalize(), replace_term.capitalize()) # Handle capitalization

        # Create a NEW LlmResponse with the modified content
        # Deep copy parts to avoid modifying original if other callbacks exist
        modified_parts = [copy.deepcopy(part) for part in llm_response.content.parts]
        modified_parts[0].text = modified_text # Update the text in the copied part

        new_response = LlmResponse(
             content=types.Content(role="model", parts=modified_parts),
             # Copy other relevant fields if necessary, e.g., grounding_metadata
             grounding_metadata=llm_response.grounding_metadata
             )
        print(f"[Callback] Returning modified response.")
        return new_response # Return the modified response
    else:
        print(f"[Callback] '{search_term}' not found. Passing original response through.")
        # Return None to use the original llm_response
        return None

# Create LlmAgent and Assign Callback
my_llm_agent = LlmAgent(
        name="AfterModelCallbackAgent",
        model=GEMINI_2_FLASH,
        instruction="You are a helpful assistant.",
        description="An LLM agent demonstrating after_model_callback",
        after_model_callback=simple_after_model_modifier # Assign the function here
)

APP_NAME = "guardrail_app"
USER_ID = "user_1"
SESSION_ID = "session_001"

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=my_llm_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
  session, runner = await setup_session_and_runner()

  content = types.Content(role='user', parts=[types.Part(text=query)])
  events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

  async for event in events:
      if event.is_final_response():
          final_response = event.content.parts[0].text
          print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("""write multiple time the word "joke" """)
```

## Tool Execution Callbacks

These callbacks are also specific to `LlmAgent` and trigger around the execution of tools (including `FunctionTool` , `AgentTool` , etc.) that the LLM might request.

### Before Tool Callback

**When:** Called just before a specific tool's `run_async` method is invoked, after the LLM has generated a function call for it.

**Purpose:** Allows inspection and modification of tool arguments, performing authorization checks before execution, logging tool usage attempts, or implementing tool-level caching.

**Return Value Effect:**

1. If the callback returns `None` (or a `Maybe.empty()` object in Java), the tool's `run_async` method is executed with the (potentially modified) `args` .
2. If a dictionary (or `Map` in Java) is returned, the tool's `run_async` method is **skipped** . The returned dictionary is used directly as the result of the tool call. This is useful for caching or overriding tool behavior.
Code
```
# Copyright 2025 Google LLC

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at

#     http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from typing import Optional
from google.genai import types
from google.adk.sessions import InMemorySessionService
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext
from google.adk.tools.base_tool import BaseTool
from typing import Dict, Any

GEMINI_2_FLASH="gemini-2.5-flash"

def get_capital_city(country: str) -> str:
    """Retrieves the capital city of a given country."""
    print(f"--- Tool 'get_capital_city' executing with country: {country} ---")
    country_capitals = {
        "united states": "Washington, D.C.",
        "canada": "Ottawa",
        "france": "Paris",
        "germany": "Berlin",
    }
    return country_capitals.get(country.lower(), f"Capital not found for {country}")

capital_tool = FunctionTool(func=get_capital_city)

def simple_before_tool_modifier(
    tool: BaseTool, args: Dict[str, Any], tool_context: ToolContext
) -> Optional[Dict]:
    """Inspects/modifies tool args or skips the tool call."""
    agent_name = tool_context.agent_name
    tool_name = tool.name
    print(f"[Callback] Before tool call for tool '{tool_name}' in agent '{agent_name}'")
    print(f"[Callback] Original args: {args}")

    if tool_name == 'get_capital_city' and args.get('country', '').lower() == 'canada':
        print("[Callback] Detected 'Canada'. Modifying args to 'France'.")
        args['country'] = 'France'
        print(f"[Callback] Modified args: {args}")
        return None

    # If the tool is 'get_capital_city' and country is 'BLOCK'
    if tool_name == 'get_capital_city' and args.get('country', '').upper() == 'BLOCK':
        print("[Callback] Detected 'BLOCK'. Skipping tool execution.")
        return {"result": "Tool execution was blocked by before_tool_callback."}

    print("[Callback] Proceeding with original or previously modified args.")
    return None

my_llm_agent = LlmAgent(
        name="ToolCallbackAgent",
        model=GEMINI_2_FLASH,
        instruction="You are an agent that can find capital cities. Use the get_capital_city tool.",
        description="An LLM agent demonstrating before_tool_callback",
        tools=[capital_tool],
        before_tool_callback=simple_before_tool_modifier
)

APP_NAME = "guardrail_app"
USER_ID = "user_1"
SESSION_ID = "session_001"

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=my_llm_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("Canada")
```

### After Tool Callback

**When:** Called just after the tool's `run_async` method completes successfully.

**Purpose:** Allows inspection and modification of the tool's result before it's sent back to the LLM (potentially after summarization). Useful for logging tool results, post-processing or formatting results, or saving specific parts of the result to the session state.

**Return Value Effect:**

1. If the callback returns `None` (or a `Maybe.empty()` object in Java), the original `tool_response` is used.
2. If a new dictionary is returned, it **replaces** the original `tool_response` . This allows modifying or filtering the result seen by the LLM.

Code
```
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from typing import Optional
from google.genai import types
from google.adk.sessions import InMemorySessionService
from google.adk.tools import FunctionTool
from google.adk.tools.tool_context import ToolContext
from google.adk.tools.base_tool import BaseTool
from typing import Dict, Any
from copy import deepcopy

GEMINI_2_FLASH="gemini-2.5-flash"

# --- Define a Simple Tool Function (Same as before) ---
def get_capital_city(country: str) -> str:
    """Retrieves the capital city of a given country."""
    print(f"--- Tool 'get_capital_city' executing with country: {country} ---")
    country_capitals = {
        "united states": "Washington, D.C.",
        "canada": "Ottawa",
        "france": "Paris",
        "germany": "Berlin",
    }
    return {"result": country_capitals.get(country.lower(), f"Capital not found for {country}")}

# --- Wrap the function into a Tool ---
capital_tool = FunctionTool(func=get_capital_city)

# --- Define the Callback Function ---
def simple_after_tool_modifier(
    tool: BaseTool, args: Dict[str, Any], tool_context: ToolContext, tool_response: Dict
) -> Optional[Dict]:
    """Inspects/modifies the tool result after execution."""
    agent_name = tool_context.agent_name
    tool_name = tool.name
    print(f"[Callback] After tool call for tool '{tool_name}' in agent '{agent_name}'")
    print(f"[Callback] Args used: {args}")
    print(f"[Callback] Original tool_response: {tool_response}")

    # Default structure for function tool results is {"result": <return_value>}
    original_result_value = tool_response.get("result", "")
    # original_result_value = tool_response

    # --- Modification Example ---
    # If the tool was 'get_capital_city' and result is 'Washington, D.C.'
    if tool_name == 'get_capital_city' and original_result_value == "Washington, D.C.":
        print("[Callback] Detected 'Washington, D.C.'. Modifying tool response.")

        # IMPORTANT: Create a new dictionary or modify a copy
        modified_response = deepcopy(tool_response)
        modified_response["result"] = f"{original_result_value} (Note: This is the capital of the USA)."
        modified_response["note_added_by_callback"] = True # Add extra info if needed

        print(f"[Callback] Modified tool_response: {modified_response}")
        return modified_response # Return the modified dictionary

    print("[Callback] Passing original tool response through.")
    # Return None to use the original tool_response
    return None

# Create LlmAgent and Assign Callback
my_llm_agent = LlmAgent(
        name="AfterToolCallbackAgent",
        model=GEMINI_2_FLASH,
        instruction="You are an agent that finds capital cities using the get_capital_city tool. Report the result clearly.",
        description="An LLM agent demonstrating after_tool_callback",
        tools=[capital_tool], # Add the tool
        after_tool_callback=simple_after_tool_modifier # Assign the callback
    )

APP_NAME = "guardrail_app"
USER_ID = "user_1"
SESSION_ID = "session_001"

# Session and Runner
async def setup_session_and_runner():
    session_service = InMemorySessionService()
    session = await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=my_llm_agent, app_name=APP_NAME, session_service=session_service)
    return session, runner

# Agent Interaction
async def call_agent_async(query):
    content = types.Content(role='user', parts=[types.Part(text=query)])
    session, runner = await setup_session_and_runner()
    events = runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    async for event in events:
        if event.is_final_response():
            final_response = event.content.parts[].text
            print("Agent Response: ", final_response)

# Note: In Colab, you can directly use 'await' at the top level.
# If running this code as a standalone Python script, you'll need to use asyncio.run() or manage the event loop.
await call_agent_async("united states")
```

-----------------

# Design Patterns and Best Practices for Callbacks

Callbacks offer powerful hooks into the agent lifecycle. Here are common design patterns illustrating how to leverage them effectively in ADK, followed by best practices for implementation.

## Design Patterns

These patterns demonstrate typical ways to enhance or control agent behavior using callbacks:

### 1. Guardrails & Policy Enforcement

**Pattern Overview:** Intercept requests before they reach the LLM or tools to enforce rules.

**Implementation:** - Use `before_model_callback` to inspect the `LlmRequest` prompt
- Use `before_tool_callback` to inspect tool arguments
- If a policy violation is detected (e.g., forbidden topics, profanity):
  - Return a predefined response ( `LlmResponse` or `dict` / `Map` ) to block the operation
  - Optionally update `context.state` to log the violation

**Example Use Case:** A `before_model_callback` checks `llm_request.contents` for sensitive keywords and returns a standard "Cannot process this request" `LlmResponse` if found, preventing the LLM call.

### 2. Dynamic State Management

**Pattern Overview:** Read from and write to session state within callbacks to make agent behavior context-aware and pass data between steps.

**Implementation:** - Access `callback_context.state` or `tool_context.state` - Modifications ( `state['key'] = value` ) are automatically tracked in the subsequent `Event.actions.state_delta` - Changes are persisted by the `SessionService`

**Example Use Case:** An `after_tool_callback` saves a `transaction_id` from the tool's result to `tool_context.state['last_transaction_id']` . A later `before_agent_callback` might read `state['user_tier']` to customize the agent's greeting.

### 3. Logging and Monitoring

**Pattern Overview:** Add detailed logging at specific lifecycle points for observability and debugging.

**Implementation:** - Implement callbacks (e.g., `before_agent_callback` , `after_tool_callback` , `after_model_callback` )
- Print or send structured logs containing:
  - Agent name
  - Tool name
  - Invocation ID
  - Relevant data from the context or arguments

**Example Use Case:** Log messages like `INFO: [Invocation: e-123] Before Tool: search_api - Args: {'query': 'ADK'}` .

### 4. Caching

**Pattern Overview:** Avoid redundant LLM calls or tool executions by caching results.

**Implementation Steps:** 1. **Before Operation:** In `before_model_callback` or `before_tool_callback` :
   - Generate a cache key based on the request/arguments
   - Check `context.state` (or an external cache) for this key
   - If found, return the cached `LlmResponse` or result directly

1. **After Operation:** If cache miss occurred:
2. Use the corresponding `after_` callback to store the new result in the cache using the key

**Example Use Case:**  `before_tool_callback` for `get_stock_price(symbol)` checks `state[f"cache:stock:{symbol}"]` . If present, returns the cached price; otherwise, allows the API call and `after_tool_callback` saves the result to the state key.

### 5. Request/Response Modification

**Pattern Overview:** Alter data just before it's sent to the LLM/tool or just after it's received.

**Implementation Options:** - **`before_model_callback` :** Modify `llm_request` (e.g., add system instructions based on `state` )
- **`after_model_callback` :** Modify the returned `LlmResponse` (e.g., format text, filter content)
- **`before_tool_callback` :** Modify the tool `args` dictionary (or Map in Java)
- **`after_tool_callback` :** Modify the `tool_response` dictionary (or Map in Java)

**Example Use Case:**  `before_model_callback` appends "User language preference: Spanish" to `llm_request.config.system_instruction` if `context.state['lang'] == 'es'` .

### 6. Conditional Skipping of Steps

**Pattern Overview:** Prevent standard operations (agent run, LLM call, tool execution) based on certain conditions.

**Implementation:** - Return a value from a `before_` callback to skip the normal execution:
  - `Content` from `before_agent_callback` - `LlmResponse` from `before_model_callback` - `dict` from `before_tool_callback` - The framework interprets this returned value as the result for that step

**Example Use Case:**  `before_tool_callback` checks `tool_context.state['api_quota_exceeded']` . If `True` , it returns `{'error': 'API quota exceeded'}` , preventing the actual tool function from running.

### 7. Tool-Specific Actions (Authentication & Summarization Control)

**Pattern Overview:** Handle actions specific to the tool lifecycle, primarily authentication and controlling LLM summarization of tool results.

**Implementation:** Use `ToolContext` within tool callbacks ( `before_tool_callback` , `after_tool_callback` ):

- **Authentication:** Call `tool_context.request_credential(auth_config)` in `before_tool_callback` if credentials are required but not found (e.g., via `tool_context.get_auth_response` or state check). This initiates the auth flow.
- **Summarization:** Set `tool_context.actions.skip_summarization = True` if the raw dictionary output of the tool should be passed back to the LLM or potentially displayed directly, bypassing the default LLM summarization step.

**Example Use Case:** A `before_tool_callback` for a secure API checks for an auth token in state; if missing, it calls `request_credential` . An `after_tool_callback` for a tool returning structured JSON might set `skip_summarization = True` .

### 8. Artifact Handling

**Pattern Overview:** Save or load session-related files or large data blobs during the agent lifecycle.

**Implementation:** - **Saving:** Use `callback_context.save_artifact` / `await tool_context.save_artifact` to store data:
  - Generated reports
  - Logs
  - Intermediate data
- **Loading:** Use `load_artifact` to retrieve previously stored artifacts
- **Tracking:** Changes are tracked via `Event.actions.artifact_delta`

**Example Use Case:** An `after_tool_callback` for a "generate_report" tool saves the output file using `await tool_context.save_artifact("report.pdf", report_part)` . A `before_agent_callback` might load a configuration artifact using `callback_context.load_artifact("agent_config.json")` .

## Best Practices for Callbacks

### Design Principles

**Keep Focused:** Design each callback for a single, well-defined purpose (e.g., just logging, just validation). Avoid monolithic callbacks.

**Mind Performance:** Callbacks execute synchronously within the agent's processing loop. Avoid long-running or blocking operations (network calls, heavy computation). Offload if necessary, but be aware this adds complexity.

### Error Handling

**Handle Errors Gracefully:** - Use `try...except/catch` blocks within your callback functions
- Log errors appropriately
- Decide if the agent invocation should halt or attempt recovery
- Don't let callback errors crash the entire process

### State Management

**Manage State Carefully:** - Be deliberate about reading from and writing to `context.state` - Changes are immediately visible within the *current* invocation and persisted at the end of the event processing
- Use specific state keys rather than modifying broad structures to avoid unintended side effects
- Consider using state prefixes ( `State.APP_PREFIX` , `State.USER_PREFIX` , `State.TEMP_PREFIX` ) for clarity, especially with persistent `SessionService` implementations

### Reliability

**Consider Idempotency:** If a callback performs actions with external side effects (e.g., incrementing an external counter), design it to be idempotent (safe to run multiple times with the same input) if possible, to handle potential retries in the framework or your application.
