
## Before you begin

You need a Gemini API key. If you don't already have one, you can get it for free in Google AI Studio.

## Old SDK (Deprecated)

The old `google-generativeai` package was used like below. This method is deprecated. DO NOT use the `google-generativeai` package.

```
from google import generativeai as genai

genai.configure(api_key=GEMINI_API_KEY)

model = genai.GenerativeModel('gemini-1.5-flash')
```

The package to be used is given below.

## New Google GenAI SDK

Install the  `google-genai` package using the following pip command, or add it to the requirements.txt file.

```
pip install -q -U google-genai
```

DO NOT use the `google-generativeai` package. USE the `google-genai` package.

## Make your first request

Use the  `generateContent`  method
to send a request to the Gemini API.

```
from google import genai

client = genai.Client(api_key="YOUR_API_KEY")

response = client.models.generate_content(
    model="gemini-2.5-flash", contents="Explain how AI works in a few words"
)
print(response.text)
```

# Text generation

The Gemini API can generate text output from various inputs, including text,
images, video, and audio, leveraging Gemini models.

Here's a basic example that takes a single text input:

```
from google import genai

client = genai.Client(api_key="GEMINI_API_KEY")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=["How does AI work?"]
)
print(response.text)
```

## System instructions and configurations

You can guide the behavior of Gemini models with system instructions. To do so,
pass a  `GenerateContentConfig`  object.

```
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    config=types.GenerateContentConfig(
        system_instruction="You are a cat. Your name is Neko."),
    contents="Hello there"
)

print(response.text)
```

The  `GenerateContentConfig`  object also lets you override default generation parameters, such as temperature .

```
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=["Explain how AI works"],
    config=types.GenerateContentConfig(
        max_output_tokens=,
        temperature=0.1
    )
)
print(response.text)
```

Refer to the  `GenerateContentConfig`  in our API reference for a complete list of configurable parameters and their
descriptions.

## Multimodal inputs

The Gemini API supports multimodal inputs, allowing you to combine text with
media files. The following example demonstrates providing an image:

```
from PIL import Image
from google import genai

client = genai.Client(api_key="GEMINI_API_KEY")

image = Image.open("/path/to/organ.png")
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[image, "Tell me about this instrument"]
)
print(response.text)
```

For alternative methods of providing images and more advanced image processing,
see our image understanding guide .
The API also supports document , video , and audio inputs and understanding.

## Streaming responses

By default, the model returns a response only after the entire generation 
process is complete.

For more fluid interactions, use streaming to receive  `GenerateContentResponse`  instances incrementally
as they're generated.

```
from google import genai

client = genai.Client(api_key="GEMINI_API_KEY")

response = client.models.generate_content_stream(
    model="gemini-2.5-flash",
    contents=["Explain how AI works"]
)
for chunk in response:
    print(chunk.text, end="")
```

## Multi-turn conversations (Chat)

Our SDKs provide functionality to collect multiple rounds of prompts and
responses into a chat, giving you an easy way to keep track of the conversation
history.

**Note:**Chat functionality is only implemented as part of the SDKs. Behind the
scenes, it still uses the  `generateContent`  API.

```
from google import genai

client = genai.Client(api_key="GEMINI_API_KEY")
chat = client.chats.create(model="gemini-2.5-flash")

response = chat.send_message("I have 2 dogs in my house.")
print(response.text)

response = chat.send_message("How many paws are in my house?")
print(response.text)

for message in chat.get_history():
    print(f'role - {message.role}',end=": ")
    print(message.parts[].text)
```

Streaming can also be used for multi-turn conversations.

```
from google import genai

client = genai.Client(api_key="GEMINI_API_KEY")
chat = client.chats.create(model="gemini-2.5-flash")

response = chat.send_message_stream("I have 2 dogs in my house.")
for chunk in response:
    print(chunk.text, end="")

response = chat.send_message_stream("How many paws are in my house?")
for chunk in response:
    print(chunk.text, end="")

for message in chat.get_history():
    print(f'role - {message.role}', end=": ")
    print(message.parts[].text)
```

## Supported models

All models in the Gemini family support text generation. To learn more
about the models and their capabilities, visit the Models page.

## Best practices

### Prompting tips

For basic text generation, a zero-shot prompt often suffices without needing examples, system instructions or specific
formatting.

For more tailored outputs:

- Use System instructions to guide the model.
- Provide few example inputs and outputs to guide the model. This is often referred to as few-shot prompting.

# Structured output

You can configure Gemini for structured output instead of unstructured text,
allowing precise extraction and standardization of information for further processing.
For example, you can use structured output to extract information from resumes,
standardize them to build a structured database.

Gemini can generate either JSON or enum values as structured output.

## Generating JSON

There are two ways to generate JSON using the Gemini API:

- Configure a schema on the model
- Provide a schema in a text prompt

Configuring a schema on the model is the**recommended**way to generate JSON,
because it constrains the model to output JSON.

### Configuring a schema (recommended)

To constrain the model to generate JSON, configure a `responseSchema` . The model
will then respond to any prompt with JSON-formatted output.

```
from google import genai
from pydantic import BaseModel

class Recipe(BaseModel):
    recipe_name: str
    ingredients: list[str]

client = genai.Client(api_key="GOOGLE_API_KEY")
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="List a few popular cookie recipes, and include the amounts of ingredients.",
    config={
        "response_mime_type": "application/json",
        "response_schema": list[Recipe],
    },
)
# Use the response as a JSON string.
print(response.text)

# Use instantiated objects.
my_recipes: list[Recipe] = response.parsed
```

**Note:** Pydantic validators are not yet supported. If a `pydantic.ValidationError` occurs, it is
suppressed, and `.parsed` may be empty/null.
The output might look like this:

```
[
  {
    "recipeName": "Chocolate Chip Cookies",
    "ingredients": [
      "1 cup (2 sticks) unsalted butter, softened",
      "3/4 cup granulated sugar",
      "3/4 cup packed brown sugar",
      "1 teaspoon vanilla extract",
      "2 large eggs",
      "2 1/4 cups all-purpose flour",
      "1 teaspoon baking soda",
      "1 teaspoon salt",
      "2 cups chocolate chips"
    ]
  },
  ...
]
```

## Generating enum values

In some cases you might want the model to choose a single option from a list of
options. To implement this behavior, you can pass an*enum*in your schema. You
can use an enum option anywhere you could use a `string` in the `responseSchema` , because an enum is an array of strings. Like a JSON schema, an
enum lets you constrain model output to meet the requirements of your
application.

For example, assume that you're developing an application to classify
musical instruments into one of five categories: `"Percussion"` , `"String"` , `"Woodwind"` , `"Brass"` , or " `"Keyboard"` ". You could create an enum to help
with this task.

In the following example, you pass an enum as the `responseSchema` , constraining the model to choose the most appropriate option.

```
from google import genai
import enum

class Instrument(enum.Enum):
  PERCUSSION = "Percussion"
  STRING = "String"
  WOODWIND = "Woodwind"
  BRASS = "Brass"
  KEYBOARD = "Keyboard"

client = genai.Client(api_key="GEMINI_API_KEY")
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='What type of instrument is an oboe?',
    config={
        'response_mime_type': 'text/x.enum',
        'response_schema': Instrument,
    },
)

print(response.text)
# Woodwind
```

The Python library will translate the type declarations for the API. However,
the API accepts a subset of the OpenAPI 3.0 schema
( Schema ).

There are two other ways to specify an enumeration. You can use a  `Literal`  :

```
Literal["Percussion", "String", "Woodwind", "Brass", "Keyboard"]
```

And you can also pass the schema as JSON:

```
from google import genai

client = genai.Client(api_key="GEMINI_API_KEY")
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='What type of instrument is an oboe?',
    config={
        'response_mime_type': 'text/x.enum',
        'response_schema': {
            "type": "STRING",
            "enum": ["Percussion", "String", "Woodwind", "Brass", "Keyboard"],
        },
    },
)

print(response.text)
# Woodwind
```

Beyond basic multiple choice problems, you can use an enum anywhere in a JSON
schema. For example, you could ask the model for a list of recipe titles and
use a `Grade` enum to give each title a popularity grade:

```
from google import genai

import enum
from pydantic import BaseModel

class Grade(enum.Enum):
    A_PLUS = "a+"
    A = "a"
    B = "b"
    C = "c"
    D = "d"
    F = "f"

class Recipe(BaseModel):
  recipe_name: str
  rating: Grade

client = genai.Client(api_key="GEMINI_API_KEY")
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='List 10 home-baked cookie recipes and give them grades based on tastiness.',
    config={
        'response_mime_type': 'application/json',
        'response_schema': list[Recipe],
    },
)

print(response.text)
```

The response might look like this:

```
[
  {
    "recipe_name": "Chocolate Chip Cookies",
    "rating": "a+"
  },
  {
    "recipe_name": "Peanut Butter Cookies",
    "rating": "a"
  },
  {
    "recipe_name": "Oatmeal Raisin Cookies",
    "rating": "b"
  },
  ...
]
```

## About JSON schemas

Configuring the model for JSON output using `responseSchema` parameter relies on `Schema` object to define its structure.  This object represents a select
subset of the OpenAPI 3.0 Schema object ,
and also adds a `propertyOrdering` field.

**Tip:**On Python, when you use a Pydantic model, you don't need to directly work with `Schema` objects,
as it gets automatically converted to the corresponding JSON schema. To learn
more, see JSON schemas in Python .
Here's a pseudo-JSON representation of all the `Schema` fields:

```
{
  "type": enum (Type),
  "format": string,
  "description": string,
  "nullable": boolean,
  "enum": [
    string
  ],
  "maxItems": integer,
  "minItems": integer,
  "properties": {
    string: {
      object (Schema)
    },
    ...
  },
  "required": [
    string
  ],
  "propertyOrdering": [
    string
  ],
  "items": {
    object (Schema)
  }
}
```

The `Type` of the schema must be one of the OpenAPI Data Types , or a union of
those types (using `anyOf` ). Only a subset of fields is valid for each `Type` .
The following list maps each `Type` to a subset of the fields that are valid for
that type:

- `string` -> `enum` , `format` , `nullable`
- `integer` -> `format` , `minimum` , `maximum` , `enum` , `nullable`
- `number` -> `format` , `minimum` , `maximum` , `enum` , `nullable`
- `boolean` -> `nullable`
- `array` -> `minItems` , `maxItems` , `items` , `nullable`
- `object` -> `properties` , `required` , `propertyOrdering` , `nullable`

Here are some example schemas showing valid type-and-field combinations:

```
{ "type": "string", "enum": ["a", "b", "c"] }

{ "type": "string", "format": "date-time" }

{ "type": "integer", "format": "int64" }

{ "type": "number", "format": "double" }

{ "type": "boolean" }

{ "type": "array", "minItems": , "maxItems": 3, "items": { "type": ... } }

{ "type": "object",
  "properties": {
    "a": { "type": ... },
    "b": { "type": ... },
    "c": { "type": ... }
  },
  "nullable": true,
  "required": ["c"],
  "propertyOrdering": ["c", "b", "a"]
}
```

For complete documentation of the Schema fields as they're used in the Gemini
API, see the Schema reference .

### Property ordering

**Warning:**When you're configuring a JSON schema, make sure to set `propertyOrdering[]` , and when you provide examples, make sure that the property
ordering in the examples matches the schema.
When you're working with JSON schemas in the Gemini API, the order of properties
is important. By default, the API orders properties alphabetically and does not
preserve the order in which the properties are defined (although the Google Gen AI SDKs may preserve this order). If you're
providing examples to the model with a schema configured, and the property
ordering of the examples is not consistent with the property ordering of the
schema, the output could be rambling or unexpected.

To ensure a consistent, predictable ordering of properties, you can use the
optional `propertyOrdering[]` field.

```
"propertyOrdering": ["recipeName", "ingredients"]
```

 `propertyOrdering[]` – not a standard field in the OpenAPI specification
– is an array of strings used to determine the order of properties in the
response. By specifying the order of properties and then providing examples with
properties in that same order, you can potentially improve the quality of
results. `propertyOrdering` is only supported when you manually create `types.Schema` .

### Schemas in Python

When you're using the Python library, the value of `response_schema` must be one
of the following:

- A type, as you would use in a type annotation (see the Python  `typing` module )
- An instance of  `genai.types.Schema`
- The `dict` equivalent of `genai.types.Schema`

The easiest way to define a schema is with a Pydantic type (as shown in the
previous example):

```
config={'response_mime_type': 'application/json',
        'response_schema': list[Recipe]}
```

When you use a Pydantic type, the Python library builds out a JSON schema for
you and sends it to the API. For additional examples, see the Python library docs .

The Python library supports schemas defined with the following types (where `AllowedType` is any allowed type):

- `int`
- `float`
- `bool`
- `str`
- `list[AllowedType]`
- `AllowedType|AllowedType|...`
- For structured types:- `dict[str, AllowedType]` . This annotation declares all dict values to
be the same type, but doesn't specify what keys should be included.
- User-defined Pydantic models . This
approach lets you specify the key names and define different types for the
values associated with each of the keys, including nested structures.


# Function Calling with the Gemini API

Function calling lets you connect models to external tools and APIs.
Instead of generating text responses, the model understands when to call specific
functions and provides the necessary parameters to execute real-world actions.
This allows the model to act as a bridge between natural language and real-world
actions and data. Function calling has 3 primary use cases:

- **Augment Knowledge:**Access information from external sources like databases, APIs, and knowledge bases.
- **Extend Capabilities:**Use external tools to perform computations and extend the limitations of the model, such as using a calculator or creating charts.
- **Take Actions:**Interact with external systems using APIs, such as scheduling appointments, creating invoices, sending emails, or controlling smart home devices

```
from google import genai
from google.genai import types

# Define the function declaration for the model
weather_function = {
    "name": "get_current_temperature",
    "description": "Gets the current temperature for a given location.",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "The city name, e.g. San Francisco",
            },
        },
        "required": ["location"],
    },
}

# Configure the client and tools
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
tools = types.Tool(function_declarations=[weather_function])
config = types.GenerateContentConfig(tools=[tools])

# Send request with function declarations
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What's the temperature in London?",
    config=config,
)

# Check for a function call
if response.candidates[].content.parts[0].function_call:
    function_call = response.candidates[0].content.parts[0].function_call
    print(f"Function to call: {function_call.name}")
    print(f"Arguments: {function_call.args}")
    #  In a real app, you would call your function here:
    #  result = get_current_temperature(**function_call.args)
else:
    print("No function call found in the response.")
    print(response.text)
```

## How Function Calling Works

Function calling involves a structured interaction between your application,
the model, and external functions. Here's a breakdown of the process:

- **Define Function Declaration:**Define the function declaration in your
application code. Function Declarations describe the function's name,
parameters, and purpose to the model.
- **Call LLM with function declarations:**Send user prompt along with the
function declaration(s) to the model. It analyzes the request and
determines if a function call would be helpful. If so, it responds with a
structured JSON object.
- **Execute Function Code (Your Responsibility):**The Model*does not*execute the function itself. It's your application's responsibility to
process the response and check for Function Call, if- **Yes**: Extract the name and args of the function and execute the
corresponding function in your application.
- **No:**The model has provided a direct text response to the prompt (this
flow is less emphasized in the example but is a possible outcome).
- **Create User friendly response:**If a function was executed, capture the
result and send it back to the model in a subsequent turn of the conversation.
It will use the result to generate a final, user-friendly response that
incorporates the information from the function call.

This process can be repeated over multiple turns, allowing for complex
interactions and workflows. The model also supports calling multiple functions in a
single turn ( parallel function calling ) and in sequence ( compositional function
calling ).

### Step 1: Define Function Declaration

Define a function and its declaration within your application code that allows
users to set light values and make an API request. This function could call
external services or APIs.

```
from google.genai import types

# Define a function that the model can call to control smart lights
set_light_values_declaration = {
    "name": "set_light_values",
    "description": "Sets the brightness and color temperature of a light.",
    "parameters": {
        "type": "object",
        "properties": {
            "brightness": {
                "type": "integer",
                "description": "Light level from 0 to 100. Zero is off and 100 is full brightness",
            },
            "color_temp": {
                "type": "string",
                "enum": ["daylight", "cool", "warm"],
                "description": "Color temperature of the light fixture, which can be `daylight`, `cool` or `warm`.",
            },
        },
        "required": ["brightness", "color_temp"],
    },
}

# This is the actual function that would be called based on the model's suggestion
def set_light_values(brightness: int, color_temp: str) -> dict[str, int | str]:
    """Set the brightness and color temperature of a room light. (mock API).

    Args:
        brightness: Light level from 0 to 100. Zero is off and 100 is full brightness
        color_temp: Color temperature of the light fixture, which can be `daylight`, `cool` or `warm`.

    Returns:
        A dictionary containing the set brightness and color temperature.
    """
    return {"brightness": brightness, "colorTemperature": color_temp}
```

### Step 2: Call the model with function declarations

Once you have defined your function declarations, you can prompt the model to use
the function. It analyzes the prompt and function declarations and decides
to respond directly or to call a function. If a function is called the response
object will contain a function call suggestion.

```
from google import genai

# Generation Config with Function Declaration
tools = types.Tool(function_declarations=[set_light_values_declaration])
config = types.GenerateContentConfig(tools=[tools])

# Configure the client
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

# Define user prompt
contents = [
    types.Content(
        role="user", parts=[types.Part(text="Turn the lights down to a romantic level")]
    )
]

# Send request with function declarations
response = client.models.generate_content(
    model="gemini-2.5-flash", config=config, contents=contents
)

print(response.candidates[].content.parts[0].function_call)
```

The model then returns a `functionCall` object in an OpenAPI compatible
schema specifying how to call one or more of the declared functions in order to
respond to the user's question.

```
id=None args={'color_temp': 'warm', 'brightness': } name='set_light_values'
```

### Step 3: Execute set_light_values function code

Extract the function call details from the model's response, parse the arguments
, and execute the `set_light_values` function in our code.

```
# Extract tool call details
tool_call = response.candidates[].content.parts[0].function_call

if tool_call.name == "set_light_values":
    result = set_light_values(**tool_call.args)
    print(f"Function execution result: {result}")
```

### Step 4: Create User friendly response with function result and call the model again

Finally, send the result of the function execution back to the model so it can incorporate this information into its final response to the user.

```
# Create a function response part
function_response_part = types.Part.from_function_response(
    name=tool_call.name,
    response={"result": result},
)

# Append function call and result of the function execution to contents
contents.append(types.Content(role="model", parts=[types.Part(function_call=tool_call)])) # Append the model's function call message
contents.append(types.Content(role="user", parts=[function_response_part])) # Append the function response

final_response = client.models.generate_content(
    model="gemini-2.5-flash",
    config=config,
    contents=contents,
)

print(final_response.text)
```

This completes the function calling flow. The Model successfully used the `set_light_values` function to perform the request action of the user.

## Function declarations

When you implement function calling in a prompt, you create a `tools` object, which contains one or more* `function declarations` *. You define functions using JSON, specifically with a select subset of the OpenAPI schema format. A single function declaration can include the following parameters:

- `name` (string): A unique name for the function ( `get_weather_forecast` , `send_email` ). Use descriptive names without spaces or special characters (use underscores or camelCase).
- `description` (string): A clear and detailed explanation of the function's purpose and capabilities. This is crucial for the model to understand when to use the function. Be specific and provide examples if helpful ("Finds theaters based on location and optionally movie title which is currently playing in theaters.").
- `parameters` (object): Defines the input parameters the function expects.- `type` (string): Specifies the overall data type, such as `object` .
- `properties` (object): Lists individual parameters, each with:- `type` (string): The data type of the parameter, such as `string` , `integer` , `boolean, array` .
- `description` (string): A description of the parameter's purpose and format. Provide examples and constraints ("The city and state, e.g., 'San Francisco, CA' or a zip code e.g., '95616'.").
- `enum` (array, optional): If the parameter values are from a fixed set, use "enum" to list the allowed values instead of just describing them in the description. This improves accuracy ("enum": ["daylight", "cool", "warm"]).
- `required` (array): An array of strings listing the parameter names that are mandatory for the function to operate.

## Parallel Function Calling

In addition to single turn function calling, you can also call multiple
functions at once. Parallel function calling lets you execute multiple
functions at once and is used when the functions are not dependent on each
other. This is useful in scenarios like gathering data from multiple
independent sources, such as retrieving customer details from different
databases or checking inventory levels across various warehouses or performing
multiple actions such as converting your apartment into a disco.

```
power_disco_ball = {
    "name": "power_disco_ball",
    "description": "Powers the spinning disco ball.",
    "parameters": {
        "type": "object",
        "properties": {
            "power": {
                "type": "boolean",
                "description": "Whether to turn the disco ball on or off.",
            }
        },
        "required": ["power"],
    },
}

start_music = {
    "name": "start_music",
    "description": "Play some music matching the specified parameters.",
    "parameters": {
        "type": "object",
        "properties": {
            "energetic": {
                "type": "boolean",
                "description": "Whether the music is energetic or not.",
            },
            "loud": {
                "type": "boolean",
                "description": "Whether the music is loud or not.",
            },
        },
        "required": ["energetic", "loud"],
    },
}

dim_lights = {
    "name": "dim_lights",
    "description": "Dim the lights.",
    "parameters": {
        "type": "object",
        "properties": {
            "brightness": {
                "type": "number",
                "description": "The brightness of the lights, 0.0 is off, 1.0 is full.",
            }
        },
        "required": ["brightness"],
    },
}
```

Call the model with an instruction that could use all of the specified tools. This example uses a `tool_config` . To learn more you can read about configuring function calling .

```
from google import genai
from google.genai import types

# Set up function declarations
house_tools = [
    types.Tool(function_declarations=[power_disco_ball, start_music, dim_lights])
]

config = {
    "tools": house_tools,
    "automatic_function_calling": {"disable": True},
    # Force the model to call 'any' function, instead of chatting.
    "tool_config": {"function_calling_config": {"mode": "any"}},
}

# Configure the client
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

chat = client.chats.create(model="gemini-2.5-flash", config=config)
response = chat.send_message("Turn this place into a party!")

# Print out each of the function calls requested from this single call
print("Example 1: Forced function calling")
for fn in response.function_calls:
    args = ", ".join(f"{key}={val}" for key, val in fn.args.items())
    print(f"{fn.name}({args})")
```

Each of the printed results reflects a single function call that the model has requested. To send the results back, include the responses in the same order as they were requested.

The Python SDK supports a feature called automatic function calling which converts the Python function to declarations, handles the function call execution and response cycle for you. Following is an example for our disco use case.

**Note:**Automatic Function Calling is a Python SDK only feature at the moment.

```
from google import genai
from google.genai import types

# Actual implementation functions
def power_disco_ball_impl(power: bool) -> dict:
    """Powers the spinning disco ball.

    Args:
        power: Whether to turn the disco ball on or off.

    Returns:
        A status dictionary indicating the current state.
    """
    return {"status": f"Disco ball powered {'on' if power else 'off'}"}

def start_music_impl(energetic: bool, loud: bool) -> dict:
    """Play some music matching the specified parameters.

    Args:
        energetic: Whether the music is energetic or not.
        loud: Whether the music is loud or not.

    Returns:
        A dictionary containing the music settings.
    """
    music_type = "energetic" if energetic else "chill"
    volume = "loud" if loud else "quiet"
    return {"music_type": music_type, "volume": volume}

def dim_lights_impl(brightness: float) -> dict:
    """Dim the lights.

    Args:
        brightness: The brightness of the lights, 0.0 is off, 1.0 is full.

    Returns:
        A dictionary containing the new brightness setting.
    """
    return {"brightness": brightness}

config = {
    "tools": [power_disco_ball_impl, start_music_impl, dim_lights_impl],
}

chat = client.chats.create(model="gemini-2.5-flash", config=config)
response = chat.send_message("Do everything you need to this place into party!")

print("\nExample 2: Automatic function calling")
print(response.text)
# I've turned on the disco ball, started playing loud and energetic music, and dimmed the lights to 50% brightness. Let's get this party started!
```

## Compositional Function Calling

Gemini 2.0 supports compositional function calling, meaning the model can chain multiple function calls together. For example, to answer "Get the temperature in my current location", the Gemini API might invoke both a `get_current_location()` function and a `get_weather()` function that takes the location as a parameter.

**Note:**Compositional function calling is a Live API only feature at the moment. The `run()` function declaration, which handles the asynchronous websocket setup, is omitted for brevity.

```
# Light control schemas
turn_on_the_lights_schema = {'name': 'turn_on_the_lights'}
turn_off_the_lights_schema = {'name': 'turn_off_the_lights'}

prompt = """
  Hey, can you write run some python code to turn on the lights, wait 10s and then turn off the lights?
  """

tools = [
    {'code_execution': {}},
    {'function_declarations': [turn_on_the_lights_schema, turn_off_the_lights_schema]}
]

await run(prompt, tools=tools, modality="AUDIO")
```

## Function calling modes

The Gemini API lets you control how the model uses the provided tools (function declarations). Specifically, you can set the mode within the `function_calling_config` .

- `AUTO (Default)` : The model decides whether to generate a natural language response or suggest a function call based on the prompt and context. This is the most flexible mode and recommended for most scenarios.
- `ANY` : The model is constrained to always predict a function call and guarantee function schema adherence. If `allowed_function_names` is not specified, the model can choose from any of the provided function declarations. If `allowed_function_names` is provided as a list, the model can only choose from the functions in that list. Use this mode when you require a function call in response to every prompt (if applicable).
- `NONE` : The model is*prohibited*from making function calls. This is equivalent to sending a request without any function declarations. Use this to temporarily disable function calling without removing your tool definitions.

```
from google.genai import types

# Configure function calling mode
tool_config = types.ToolConfig(
    function_calling_config=types.FunctionCallingConfig(
        mode="ANY", allowed_function_names=["get_current_temperature"]
    )
)

# Create the generation config
config = types.GenerateContentConfig(
    temperature=,
    tools=[tools],  # not defined here.
    tool_config=tool_config,
)
```

## Automatic Function Calling (Python Only)

When using the Python SDK, you can provide Python functions directly as tools. The SDK automatically converts the Python function to declarations, handles the function call execution and response cycle for you. The Python SDK then automatically:

- Detects function call responses from the model.
- Call the corresponding Python function in your code.
- Sends the function response back to the model.
- Returns the model's final text response.

To use this, define your function with type hints and a docstring, and then pass the function itself (not a JSON declaration) as a tool:

```
from google import genai
from google.genai import types

# Define the function with type hints and docstring
def get_current_temperature(location: str) -> dict:
    """Gets the current temperature for a given location.

    Args:
        location: The city and state, e.g. San Francisco, CA

    Returns:
        A dictionary containing the temperature and unit.
    """
    # ... (implementation) ...
    return {"temperature": , "unit": "Celsius"}

# Configure the client and model
client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))  # Replace with your actual API key setup
config = types.GenerateContentConfig(
    tools=[get_current_temperature]
)  # Pass the function itself

# Make the request
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="What's the temperature in Boston?",
    config=config,
)

print(response.text)  # The SDK handles the function call and returns the final text
```

You can disable automatic function calling with:

```
# To disable automatic function calling:
config = types.GenerateContentConfig(
    tools=[get_current_temperature],
    automatic_function_calling=types.AutomaticFunctionCallingConfig(disable=True)
)
```

### Automatic Function schema declaration

Automatic schema extraction from Python functions doesn't work in all cases. For example: it doesn't handle cases where you describe the fields of a nested  dictionary-object. The API is able to describe any of the following types:

```
AllowedType = (int | float | bool | str | list['AllowedType'] | dict[str, AllowedType])
```

To see what the inferred schema looks like, you can convert it using  `from_callable`  :

```
def multiply(a: float, b: float):
    """Returns a * b."""
    return a * b

fn_decl = types.FunctionDeclaration.from_callable(callable=multiply, client=client)

# to_json_dict() provides a clean JSON representation.
print(fn_decl.to_json_dict())
```

## Multi-tool use: Combine Native Tools with Function Calling

With Gemini 2.0, you can enable multiple tools combining native tools with function calling at the same time. Here's an example that enables two tools, Grounding with Google Search and code execution , in a request using the Live API .

**Note:**Multi-tool use is a Live API only feature at the moment. The `run()` function declaration, which handles the asynchronous websocket setup, is omitted for brevity.

```
# Multiple tasks example - combining lights, code execution, and search
prompt = """
  Hey, I need you to do three things for me.

    1.  Turn on the lights.
    2.  Then compute the largest prime palindrome under 100000.
    3.  Then use Google Search to look up information about the largest earthquake in California the week of Dec 5 2024.

  Thanks!
  """

tools = [
    {'google_search': {}},
    {'code_execution': {}},
    {'function_declarations': [turn_on_the_lights_schema, turn_off_the_lights_schema]} # not defined here.
]

# Execute the prompt with specified tools in audio modality
await run(prompt, tools=tools, modality="AUDIO")
```

Python developers can try this out in the Live API Tool Use notebook .

## Model Context Protocol (MCP)

 Model Context Protocol (MCP) is an open standard for connecting AI applications with external tools and data. MCP provides a common protocol for models to access context, such as functions (tools), data sources (resources), or predefined prompts.

The Gemini SDKs have built-in support for MCP, reducing boilerplate code and offering automatic tool calling for MCP tools. When the model generates an MCP tool call, the Python and JavaScript client SDK can automatically execute the MCP tool and send the response back to the model in a subsequent request, continuing this loop until no more tool calls are made by the model.

Here, you can find an example of how to use a local MCP server with Gemini and `mcp` SDK.

Make sure the latest version of the  `mcp` SDK is installed on your platform of choice.

```
pip install mcp
```

**Note:**Python supports automatic tool calling by passing in the `ClientSession` into the `tools` parameters. If you want to disable it you can provide `automatic_function_calling` with disabled `True` .

```
import os
import asyncio
from datetime import datetime
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
from google import genai

client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))

# Create server parameters for stdio connection
server_params = StdioServerParameters(
    command="npx",  # Executable
    args=["-y", "@philschmid/weather-mcp"],  # MCP Server
    env=None,  # Optional environment variables
)

async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # Prompt to get the weather for the current day in London.
            prompt = f"What is the weather in London in {datetime.now().strftime('%Y-%m-%d')}?"
            # Initialize the connection between client and server
            await session.initialize()
            # Send request to the model with MCP function declarations
            response = await client.aio.models.generate_content(
                model="gemini-2.5-flash",
                contents=prompt,
                config=genai.types.GenerateContentConfig(
                    temperature=,
                    tools=[session],  # uses the session, will automatically call the tool
                    # Uncomment if you **don't** want the sdk to automatically call the tool
                    # automatic_function_calling=genai.types.AutomaticFunctionCallingConfig(
                    #     disable=True
                    # ),
                ),
            )
            print(response.text)

# Start the asyncio event loop and run the main function
asyncio.run(run())
```

### Limitations with built-in MCP Support

Built-in MCP support is a experimental feature in our SDKs and has the following limitations:

- Only tools are supported, not resources or prompts
- It is available for the Python and JavaScript/TypeScript SDK.
- Breaking changes might occur in future releases.

Manual integration of MCP servers is always an option if these limit what you're building.

## Supported Models

Experimental models are not included. You can find their capabilities on the model overview page.

| Model | Function Calling | Parallel Function Calling | Compositional Function Calling(Live API only) |
| --- | --- | --- | --- |
| Gemini 2.0 Flash | ✔️ | ✔️ | ✔️ |
| Gemini 2.0 Flash-Lite | X | X | X |
| Gemini 1.5 Flash | ✔️ | ✔️ | ✔️ |
| Gemini 1.5 Pro | ✔️ | ✔️ | ✔️ |

## Best Practices

- **Function and Parameter Descriptions:**Be extremely clear and specific in your descriptions. The model relies on these to choose the correct function and provide appropriate arguments.
- **Naming:**Use descriptive function names (without spaces, periods, or dashes).
- **Strong Typing:**Use specific types (integer, string, enum) for parameters to reduce errors. If a parameter has a limited set of valid values, use an enum.
- **Tool Selection:**While the model can use an arbitrary number of tools, providing too many can increase the risk of selecting an incorrect or suboptimal tool. For best results, aim to provide only the relevant tools for the context or task, ideally keeping the active set to a maximum of 10-20. Consider dynamic tool selection based on conversation context if you have a large total number of tools.
- **Prompt Engineering:**- Provide context: Tell the model its role (e.g., "You are a helpful weather assistant.").
- Give instructions: Specify how and when to use functions (e.g., "Don't guess dates; always use a future date for forecasts.").
- Encourage clarification: Instruct the model to ask clarifying questions if needed.
- **Temperature:**Use a low temperature (e.g., 0) for more deterministic and reliable function calls.
- **Validation:**If a function call has significant consequences (e.g., placing an order), validate the call with the user before executing it.
- **Error Handling**: Implement robust error handling in your functions to gracefully handle unexpected inputs or API failures. Return informative error messages that the model can use to generate helpful responses to the user.
- **Security:**Be mindful of security when calling external APIs. Use appropriate authentication and authorization mechanisms. Avoid exposing sensitive data in function calls.
- **Token Limits:**Function descriptions and parameters count towards your input token limit. If you're hitting token limits, consider limiting the number of functions or the length of the descriptions, break down complex tasks into smaller, more focused function sets.

## Notes and Limitations

- Only a subset of the OpenAPI schema is supported.
- Supported parameter types in Python are limited.
- Automatic function calling is a Python SDK feature only.


# Files API

The Gemini family of artificial intelligence (AI) models is built to handle various types of input data, including text, images, and audio. Since these models can handle more than one type or *mode* of data, the Gemini models are called *multimodal models* or explained as having *multimodal capabilities*.

This guide shows you how to work with media files using the Files API. The basic operations are the same for audio files, images, videos, documents, and other supported file types.

For file prompting guidance, check out the File prompt guide section.

## Upload a file

You can use the Files API to upload a media file. Always use the Files API when the total request size (including the files, text prompt, system instructions, etc.) is larger than 20 MB.

The following code uploads a file and then uses the file in a call to `generateContent`.

```python
from google import genai

client = genai.Client(api_key="GOOGLE_API_KEY")

myfile = client.files.upload(file="path/to/sample.mp3")

response = client.models.generate_content(
    model="gemini-2.5-flash", contents=["Describe this audio clip", myfile]
)

print(response.text)
```

## Get metadata for a file

You can verify that the API successfully stored the uploaded file and get its metadata by calling `files.get`.

```python
myfile = client.files.upload(file='path/to/sample.mp3')
file_name = myfile.name
myfile = client.files.get(name=file_name)
print(myfile)
```

## List uploaded files

You can upload multiple files using the Files API. The following code gets a list of all the files uploaded:

```python
print('My files:')
for f in client.files.list():
    print(' ', f.name)
```

## Delete uploaded files

Files are automatically deleted after 48 hours. You can also manually delete an uploaded file:

```python
myfile = client.files.upload(file='path/to/sample.mp3')
client.files.delete(name=myfile.name)
```