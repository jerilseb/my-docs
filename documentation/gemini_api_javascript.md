# Gemini API quickstart

This quickstart shows you how to install our libraries and make your first Gemini API request.

**Note:**All our code snippets use**Google Gen AI SDK**, a new set of libraries we have been rolling out since early 2025. You can find out more about this change at our Libraries page.

## Before you begin

You need a Gemini API key. If you don't already have one, you can get it for free in Google AI Studio .

## Install the Google GenAI SDK

Using Node.js v18+ ,
install the Google Gen AI SDK for TypeScript and JavaScript using the following npm command :

```
npm install @google/genai
```

## Make your first request

Use the  `generateContent`  method
to send a request to the Gemini API.

```
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: "YOUR_API_KEY" });

async function main() {
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: "Explain how AI works in a few words",
  });
  console.log(response.text);
}

main();
```

-----------

# Text generation

The Gemini API can generate text output from various inputs, including text,
images, video, and audio, leveraging Gemini models.

Here's a basic example that takes a single text input:

```
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: "GEMINI_API_KEY" });

async function main() {
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: "How does AI work?",
  });
  console.log(response.text);
}

await main();
```

## System instructions and configurations

You can guide the behavior of Gemini models with system instructions. To do so,
pass a  `GenerateContentConfig`  object.

```
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: "GEMINI_API_KEY" });

async function main() {
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: "Hello there",
    config: {
      systemInstruction: "You are a cat. Your name is Neko.",
    },
  });
  console.log(response.text);
}

await main();
```

The  `GenerateContentConfig`  object also lets you override default generation parameters, such as temperature .

```
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: "GEMINI_API_KEY" });

async function main() {
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: "Explain how AI works",
    config: {
      maxOutputTokens: ,
      temperature: 0.1,
    },
  });
  console.log(response.text);
}

await main();
```

Refer to the  `GenerateContentConfig`  in our API reference for a complete list of configurable parameters and their
descriptions.

## Multimodal inputs

The Gemini API supports multimodal inputs, allowing you to combine text with
media files. The following example demonstrates providing an image:

```
import {
  GoogleGenAI,
  createUserContent,
  createPartFromUri,
} from "@google/genai";

const ai = new GoogleGenAI({ apiKey: "GEMINI_API_KEY" });

async function main() {
  const image = await ai.files.upload({
    file: "/path/to/organ.png",
  });
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: [
      createUserContent([
        "Tell me about this instrument",
        createPartFromUri(image.uri, image.mimeType),
      ]),
    ],
  });
  console.log(response.text);
}

await main();
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
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: "GEMINI_API_KEY" });

async function main() {
  const response = await ai.models.generateContentStream({
    model: "gemini-2.0-flash",
    contents: "Explain how AI works",
  });

  for await (const chunk of response) {
    console.log(chunk.text);
  }
}

await main();
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
- Consider fine-tuning for advanced use cases.

Consult our prompt engineering guide for
more tips.

### Structured output

In some cases, you may need structured output, such as JSON. Refer to our structured output guide to learn how.

--------------------

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

### Configuring a schema

To constrain the model to generate JSON, configure a `responseSchema` . The model
will then respond to any prompt with JSON-formatted output.

```
import { GoogleGenAI, Type } from "@google/genai";

const ai = new GoogleGenAI({ "GOOGLE_API_KEY" });

async function main() {
  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents:
      "List a few popular cookie recipes, and include the amounts of ingredients.",
    config: {
      responseMimeType: "application/json",
      responseSchema: {
        type: Type.ARRAY,
        items: {
          type: Type.OBJECT,
          properties: {
            recipeName: {
              type: Type.STRING,
            },
            ingredients: {
              type: Type.ARRAY,
              items: {
                type: Type.STRING,
              },
            },
          },
          propertyOrdering: ["recipeName", "ingredients"],
        },
      },
    },
  });

  console.log(response.text);
}

main();
```

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

----------

# Audio understanding

Gemini can analyze and understand audio input, enabling use cases like the
following:

- Describe, summarize, or answer questions about audio content.
- Provide a transcription of the audio.
- Analyze specific segments of the audio.

This guide shows you how to use the Gemini API to generate a text response to
audio input.


## Input audio

You can provide audio data to Gemini in the following ways:

- Pass inline audio data with the request to `generateContent` .

### Pass audio data inline

Instead of uploading an audio file, you can pass inline audio data in the
request to `generateContent` :

```
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({ apiKey: "GOOGLE_API_KEY" });
const base64AudioFile = fs.readFileSync("path/to/small-sample.mp3", {
  encoding: "base64",
});

const contents = [
  { text: "Please summarize the audio." },
  {
    inlineData: {
      mimeType: "audio/mp3",
      data: base64AudioFile,
    },
  },
];

const response = await ai.models.generateContent({
  model: "gemini-2.0-flash",
  contents: contents,
});
console.log(response.text);
```

A few things to keep in mind about inline audio data:

- The maximum request size is 20 MB, which includes text prompts,
system instructions, and files provided inline. If your file's
size will make the*total request size*exceed 20 MB, then
use the Files API to upload an audio file for use in
the request.
- If you're using an audio sample multiple times, it's more efficient
to upload an audio file .

## Get a transcript

To get a transcript of audio data, just ask for it in the prompt:


## Refer to timestamps

You can refer to specific sections of an audio file using timestamps of the form `MM:SS` . For example, the following prompt requests a transcript that

- Starts at 2 minutes 30 seconds from the beginning of the file.
- Ends at 3 minutes 29 seconds from the beginning of the file.

```
// Create a prompt containing timestamps.
const prompt = "Provide a transcript of the speech from 02:30 to 03:29."
```

## Supported audio formats

Gemini supports the following audio format MIME types:

- WAV - `audio/wav`
- MP3 - `audio/mp3`
- AIFF - `audio/aiff`
- AAC - `audio/aac`
- OGG Vorbis - `audio/ogg`
- FLAC - `audio/flac`

## Technical details about audio

- Gemini represents each second of audio as 32 tokens; for example,
one minute of audio is represented as 1,920 tokens.
- Gemini can "understand" non-speech components, such as birdsong or sirens.
- The maximum supported length of audio data in a single prompt is 9.5 hours.
Gemini doesn't limit the*number*of audio files in a single prompt; however,
the total combined length of all audio files in a single prompt can't exceed
9.5 hours.
- Gemini downsamples audio files to a 16 Kbps data resolution.
- If the audio source contains multiple channels, Gemini combines those channels
into a single channel.


----------

# Live API

**Preview:**The Live API is in preview.
The Live API enables low-latency bidirectional voice and
video interactions with Gemini, letting you talk to Gemini live while also
streaming video input or sharing your screen. Using the
Live API, you can provide end users with the experience of
natural, human-like voice conversations.

You can try the Live API in Google AI Studio . To use the Live API in
Google AI Studio, select**Stream**.

## How the Live API works

### Streaming

The Live API uses a streaming model over a WebSocket connection. When you interact with the API, a persistent connection is created.
Your input (audio, video, or text) is streamed continuously to the model, and
the model's response (text or audio) is streamed back in real-time over the same
connection.

This bidirectional streaming ensures low latency and supports features such as
voice activity detection, tool usage, and speech generation.

For more information about the underlying WebSockets API, see the WebSockets API reference .

**Warning:**It is unsafe to insert your API key into client-side JavaScript or
TypeScript code. Use server-side deployments for accessing the Live API in
production.
### Output generation

The Live API processes multimodal input (text, audio, video)
to generate text or audio in real-time. It comes with a built-in mechanism to
generate audio and depending on the model version you use, it uses one of the
two audio generation methods:

- **Half cascade**: The model receives native audio input and uses a specialized
model cascade of distinct models to process the input and to generate audio output.
- **Native**: Gemini 2.5 introduces native audio generation ,
which directly generates audio output, providing a more natural sounding audio, more expressive voices, more
awareness of additional context, e.g., tone, and more proactive responses.

## Building with Live API

Before you begin building with the Live API, choose the audio generation approach that best fits your needs.

### Establishing a connection

The following example shows how to create a connection with an API key:

```
import { GoogleGenAI } from '@google/genai';

const ai = new GoogleGenAI({ apiKey: "GOOGLE_API_KEY" });
const model = 'gemini-2.0-flash-live-001';
const config = { responseModalities: [Modality.TEXT] };

async function main() {

    const session = await ai.live.connect({
        model: model,
        callbacks: {
            onopen: function () {
                console.debug('Opened');
            },
            onmessage: function (message) {
                console.debug(message);
            },
            onerror: function (e) {
                console.debug('Error:', e.message);
            },
            onclose: function (e) {
                console.debug('Close:', e.reason);
            },
        },
        config: config,
    });

    // Send content...

    session.close();
}

main();
```

**Note:**You can only set one modality in the `response_modalities` field. This means that you can configure the model
to respond with either text or audio, but not both in the same session.
### Sending and receiving text

Here's how you can send and receive text:

```
import { GoogleGenAI, Modality } from '@google/genai';

const ai = new GoogleGenAI({ apiKey: "GOOGLE_API_KEY" });
const model = 'gemini-2.0-flash-live-001';
const config = { responseModalities: [Modality.TEXT] };

async function live() {
    const responseQueue = [];

    async function waitMessage() {
        let done = false;
        let message = undefined;
        while (!done) {
            message = responseQueue.shift();
            if (message) {
                done = true;
            } else {
                await new Promise((resolve) => setTimeout(resolve, ));
            }
        }
        return message;
    }

    async function handleTurn() {
        const turns = [];
        let done = false;
        while (!done) {
            const message = await waitMessage();
            turns.push(message);
            if (message.serverContent && message.serverContent.turnComplete) {
                done = true;
            }
        }
        return turns;
    }

    const session = await ai.live.connect({
        model: model,
        callbacks: {
            onopen: function () {
                console.debug('Opened');
            },
            onmessage: function (message) {
                responseQueue.push(message);
            },
            onerror: function (e) {
                console.debug('Error:', e.message);
            },
            onclose: function (e) {
                console.debug('Close:', e.reason);
            },
        },
        config: config,
    });

    const simple = 'Hello how are you?';
    session.sendClientContent({ turns: simple });

    const turns = await handleTurn();
    for (const turn of turns) {
        if (turn.text) {
            console.debug('Received text: %s\n', turn.text);
        }
        else if (turn.data) {
            console.debug('Received inline data: %s\n', turn.data);
        }
    }

    session.close();
}

async function main() {
    await live().catch((e) => console.error('got error', e));
}

main();
```

### Sending and receiving audio

You can send audio by converting it to 16-bit PCM, 16kHz, mono format. This
example reads a WAV file and sends it in the correct format:

```
// Test file: https://storage.googleapis.com/generativeai-downloads/data/16000.wav
// Install helpers for converting files: npm install wavefile
import { GoogleGenAI, Modality } from '@google/genai';
import * as fs from "node:fs";
import pkg from 'wavefile';
const { WaveFile } = pkg;

const ai = new GoogleGenAI({ apiKey: "GOOGLE_API_KEY" });
const model = 'gemini-2.0-flash-live-001';
const config = { responseModalities: [Modality.TEXT] };

async function live() {
    const responseQueue = [];

    async function waitMessage() {
        let done = false;
        let message = undefined;
        while (!done) {
            message = responseQueue.shift();
            if (message) {
                done = true;
            } else {
                await new Promise((resolve) => setTimeout(resolve, ));
            }
        }
        return message;
    }

    async function handleTurn() {
        const turns = [];
        let done = false;
        while (!done) {
            const message = await waitMessage();
            turns.push(message);
            if (message.serverContent && message.serverContent.turnComplete) {
                done = true;
            }
        }
        return turns;
    }

    const session = await ai.live.connect({
        model: model,
        callbacks: {
            onopen: function () {
                console.debug('Opened');
            },
            onmessage: function (message) {
                responseQueue.push(message);
            },
            onerror: function (e) {
                console.debug('Error:', e.message);
            },
            onclose: function (e) {
                console.debug('Close:', e.reason);
            },
        },
        config: config,
    });

    // Send Audio Chunk
    const fileBuffer = fs.readFileSync("sample.wav");

    // Ensure audio conforms to API requirements (16-bit PCM, 16kHz, mono)
    const wav = new WaveFile();
    wav.fromBuffer(fileBuffer);
    wav.toSampleRate(16000);
    wav.toBitDepth("16");
    const base64Audio = wav.toBase64();

    // If already in correct format, you can use this:
    // const fileBuffer = fs.readFileSync("sample.pcm");
    // const base64Audio = Buffer.from(fileBuffer).toString('base64');

    session.sendRealtimeInput(
        {
            audio: {
                data: base64Audio,
                mimeType: "audio/pcm;rate=16000"
            }
        }

    );

    const turns = await handleTurn();
    for (const turn of turns) {
        if (turn.text) {
            console.debug('Received text: %s\n', turn.text);
        }
        else if (turn.data) {
            console.debug('Received inline data: %s\n', turn.data);
        }
    }

    session.close();
}

async function main() {
    await live().catch((e) => console.error('got error', e));
}

main();
```

You can receive audio by setting `AUDIO` as response modality. This example
saves the received data as WAV file:

```
import { GoogleGenAI, Modality } from '@google/genai';

const ai = new GoogleGenAI({ apiKey: "GOOGLE_API_KEY" });
const model = 'gemini-2.0-flash-live-001';
const config = { responseModalities: [Modality.AUDIO] };

async function live() {
    const responseQueue = [];

    async function waitMessage() {
        let done = false;
        let message = undefined;
        while (!done) {
            message = responseQueue.shift();
            if (message) {
                done = true;
            } else {
                await new Promise((resolve) => setTimeout(resolve, ));
            }
        }
        return message;
    }

    async function handleTurn() {
        const turns = [];
        let done = false;
        while (!done) {
            const message = await waitMessage();
            turns.push(message);
            if (message.serverContent && message.serverContent.turnComplete) {
                done = true;
            }
        }
        return turns;
    }

    const session = await ai.live.connect({
        model: model,
        callbacks: {
            onopen: function () {
                console.debug('Opened');
            },
            onmessage: function (message) {
                responseQueue.push(message);
            },
            onerror: function (e) {
                console.debug('Error:', e.message);
            },
            onclose: function (e) {
                console.debug('Close:', e.reason);
            },
        },
        config: config,
    });

    const simple = 'Hello how are you?';
    session.sendClientContent({ turns: simple });

    const turns = await handleTurn();

    // Combine audio data strings and save as wave file
    const combinedAudio = turns.reduce((acc, turn) => {
        if (turn.data) {
            const buffer = Buffer.from(turn.data, 'base64');
            const intArray = new Int16Array(buffer.buffer, buffer.byteOffset, buffer.byteLength / Int16Array.BYTES_PER_ELEMENT);
            return acc.concat(Array.from(intArray));
        }
        return acc;
    }, []);

    const audioBuffer = new Int16Array(combinedAudio);

    const wf = new WaveFile();
    wf.fromScratch(1, 24000, '16', audioBuffer);
    fs.writeFileSync('output.wav', wf.toBuffer());

    session.close();
}

async function main() {
    await live().catch((e) => console.error('got error', e));
}

main();
```

#### Audio formats

Audio data in the Live API is always raw, little-endian,
16-bit PCM. Audio output always uses a sample rate of 24kHz. Input audio
is natively 16kHz, but the Live API will resample if needed
so any sample rate can be sent. To convey the sample rate of input audio, set
the MIME type of each audio-containing Blob to a value
like `audio/pcm;rate=16000` .

#### Receiving audio transcriptions

You can enable transcription of the model's audio output by sending `output_audio_transcription` in the setup config. The transcription language is
inferred from the model's response.

```
import asyncio
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")
model = "gemini-2.0-flash-live-001"

config = {"response_modalities": ["AUDIO"],
          "output_audio_transcription": {}
}

async def main():
    async with client.aio.live.connect(model=model, config=config) as session:
        message = "Hello? Gemini are you there?"

        await session.send_client_content(
            turns={"role": "user", "parts": [{"text": message}]}, turn_complete=True
        )

        async for response in session.receive():
            if response.server_content.model_turn:
                print("Model turn:", response.server_content.model_turn)
            if response.server_content.output_transcription:
                print("Transcript:", response.server_content.output_transcription.text)

if __name__ == "__main__":
    asyncio.run(main())
```

You can enable transcription of the audio input by sending `input_audio_transcription` in setup config.

```
import asyncio
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")
model = "gemini-2.0-flash-live-001"

config = {"response_modalities": ["TEXT"],
    "realtime_input_config": {
        "automatic_activity_detection": {"disabled": True},
        "activity_handling": "NO_INTERRUPTION",
    },
    "input_audio_transcription": {},
}

async def main():
    async with client.aio.live.connect(model=model, config=config) as session:
        audio_data = Path("sample.pcm").read_bytes()

        await session.send_realtime_input(activity_start=types.ActivityStart())
        await session.send_realtime_input(
            audio=types.Blob(data=audio_data, mime_type='audio/pcm;rate=16000')
        )
        await session.send_realtime_input(activity_end=types.ActivityEnd())

        async for msg in session.receive():
            if msg.server_content.input_transcription:
                print('Transcript:', msg.server_content.input_transcription.text)

if __name__ == "__main__":
    asyncio.run(main())
```

### System instructions

System instructions let you steer the behavior of a model based on your specific
needs and use cases. System instructions can be set in the setup configuration
and will remain in effect for the entire session.

```
from google.genai import types

config = {
    "system_instruction": types.Content(
        parts=[
            types.Part(
                text="You are a helpful assistant and answer in a friendly tone."
            )
        ]
    ),
    "response_modalities": ["TEXT"],
}
```

### Incremental content updates

Use incremental updates to send text input, establish session context, or
restore session context. For short contexts you can send turn-by-turn
interactions to represent the exact sequence of events:

```
turns = [
    {"role": "user", "parts": [{"text": "What is the capital of France?"}]},
    {"role": "model", "parts": [{"text": "Paris"}]},
]

await session.send_client_content(turns=turns, turn_complete=False)

turns = [{"role": "user", "parts": [{"text": "What is the capital of Germany?"}]}]

await session.send_client_content(turns=turns, turn_complete=True)
```

For longer contexts it's recommended to provide a single message summary to free
up the context window for subsequent interactions.

### Changing voice and language

The Live API supports the following voices: Puck, Charon,
Kore, Fenrir, Aoede, Leda, Orus, and Zephyr.

To specify a voice, set the voice name within the `speechConfig` object as part
of the session configuration:

```
from google.genai import types

config = types.LiveConnectConfig(
    response_modalities=["AUDIO"],
    speech_config=types.SpeechConfig(
        voice_config=types.VoiceConfig(
            prebuilt_voice_config=types.PrebuiltVoiceConfig(voice_name="Kore")
        )
    )
)
```

**Note:**If you're using the `generateContent` API, the set of available voices is
slightly different. See the audio generation guide for `generateContent` audio generation voices.
The Live API supports multiple languages .

To change the language, set the language code within the `speechConfig` object
as part of the session configuration:

```
from google.genai import types

config = types.LiveConnectConfig(
    response_modalities=["AUDIO"],
    speech_config=types.SpeechConfig(
        language_code="de-DE",
    )
)
```

**Note:** Native audio output models automatically choose
the appropriate language and don't support explicitly setting the language
code.

## Native audio output

Through the Live API, you can also access models that
allow for native audio output in addition to native audio input. This allows for
higher quality audio outputs with better pacing, voice naturalness,
verbosity, and mood.

Native audio output is supported by the following native audio models :

- `gemini-2.5-flash-preview-native-audio-dialog`
- `gemini-2.5-flash-exp-native-audio-thinking-dialog`
**Note:**Native audio models currently have limited tool use support. See Overview of supported tools for details.
### How to use native audio output

To use native audio output, configure one of the native audio models and set `response_modalities` to `AUDIO` :

```
model = "gemini-2.5-flash-preview-native-audio-dialog"
config = types.LiveConnectConfig(response_modalities=["AUDIO"])

async with client.aio.live.connect(model=model, config=config) as session:
    # Send audio input and receive audio
```

### Affective dialog

This feature lets Gemini adapt its response style to the input expression and
tone.

To use affective dialog, set `enable_affective_dialog` to `true` in the setup
message:

```
config = types.LiveConnectConfig(
    response_modalities=["AUDIO"],
    enable_affective_dialog=True
)
```

Note that affective dialog is currently only supported by the native audio
output models.

### Proactive audio

When this feature is enabled, Gemini can proactively decide not to respond
if the content is not relevant.

To use it, configure the `proactivity` field in the setup message and set `proactive_audio` to `true` :

```
config = types.LiveConnectConfig(
    response_modalities=["AUDIO"],
    proactivity={'proactive_audio': True}
)
```

Note that proactive audio is currently only supported by the native audio output
models.

### Native audio output with thinking

Native audio output supports thinking capabilities ,
available via a separate model `gemini-2.5-flash-exp-native-audio-thinking-dialog` .

```
model = "gemini-2.5-flash-exp-native-audio-thinking-dialog"
config = types.LiveConnectConfig(response_modalities=["AUDIO"])

async with client.aio.live.connect(model=model, config=config) as session:
    # Send audio input and receive audio
```

## Tool use with Live API

You can define tools such as Function calling , Code execution , and Google Search with the
Live API.

To see examples of all tools in the Live API,
      run the "Live API Tools" cookbook:

 View
        on GitHub 

### Overview of supported tools

Here's a brief overview of the available tools for each model:

| Tool | Cascaded modelsgemini-2.0-flash-live-001 | gemini-2.5-flash-preview-native-audio-dialog | gemini-2.5-flash-exp-native-audio-thinking-dialog |
| --- | --- | --- | --- |
| Search | Yes | Yes | Yes |
| Function calling | Yes | Yes | No |
| Code execution | Yes | No | No |
| Url context | Yes | No | No |

### Function calling

You can define function declarations as part of the session configuration.
See the Function calling tutorial to learn more.

After receiving tool calls, the client should respond with a list of `FunctionResponse` objects using the `session.send_tool_response` method.

**Note:**Unlike the `generateContent` API, the Live API
doesn't support automatic tool response handling. You must handle tool responses
manually in your client code.

```
import asyncio
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")
model = "gemini-2.0-flash-live-001"

# Simple function definitions
turn_on_the_lights = {"name": "turn_on_the_lights"}
turn_off_the_lights = {"name": "turn_off_the_lights"}

tools = [{"function_declarations": [turn_on_the_lights, turn_off_the_lights]}]
config = {"response_modalities": ["TEXT"], "tools": tools}

async def main():
    async with client.aio.live.connect(model=model, config=config) as session:
        prompt = "Turn on the lights please"
        await session.send_client_content(turns={"parts": [{"text": prompt}]})

        async for chunk in session.receive():
            if chunk.server_content:
                if chunk.text is not None:
                    print(chunk.text)
            elif chunk.tool_call:
                function_responses = []
                for fc in tool_call.function_calls:
                    function_response = types.FunctionResponse(
                        id=fc.id,
                        name=fc.name,
                        response={ "result": "ok" } # simple, hard-coded function response
                    )
                    function_responses.append(function_response)

                await session.send_tool_response(function_responses=function_responses)

if __name__ == "__main__":
    asyncio.run(main())
```

From a single prompt, the model can generate multiple function calls and the
code necessary to chain their outputs. This code executes in a sandbox
environment, generating subsequent BidiGenerateContentToolCall messages.

### Asynchronous function calling

By default, the execution pauses until the results of each function
call are available, which ensures sequential processing. It means you won't be
able to continue interacting with the model while the functions are being run.

If you don't want to block the conversation, you can tell the model to run the
functions asynchronously.

To do so, you first need to add a `behavior` to the function
definitions:

```
# Non-blocking function definitions
  turn_on_the_lights = {"name": "turn_on_the_lights", "behavior": "NON_BLOCKING"} # turn_on_the_lights will run asynchronously
  turn_off_the_lights = {"name": "turn_off_the_lights"} # turn_off_the_lights will still pause all interactions with the model
```

 `NON-BLOCKING` will ensure the function will run asynchronously while you can
continue interacting with the model.

Then you need to tell the model how to behave when it receives the `FunctionResponse` using the `scheduling` parameter. It can either:

- Interrupt what it's doing and tell you about the response it got right away
( `scheduling="INTERRUPT"` ),
- Wait until it's finished with what it's currently doing
( `scheduling="WHEN_IDLE"` ),
- Or do nothing and use that knowledge later on in the discussion
( `scheduling="SILENT"` )

```
# Non-blocking function definitions
  function_response = types.FunctionResponse(
      id=fc.id,
      name=fc.name,
      response={
          "result": "ok",
          "scheduling": "INTERRUPT" # Can also be WHEN_IDLE or SILENT
      }
  )
```

### Grounding with Google Search

You can enable Grounding with Google Search as part of the session
configuration. See the Grounding tutorial to learn more.

```
import asyncio
from google import genai
from google.genai import types

client = genai.Client(api_key="GEMINI_API_KEY")
model = "gemini-2.0-flash-live-001"

tools = [{'google_search': {}}]
config = {"response_modalities": ["TEXT"], "tools": tools}

async def main():
    async with client.aio.live.connect(model=model, config=config) as session:
        prompt = "When did the last Brazil vs. Argentina soccer match happen?"
        await session.send_client_content(turns={"parts": [{"text": prompt}]})

        async for chunk in session.receive():
            if chunk.server_content:
                if chunk.text is not None:
                    print(chunk.text)

                # The model might generate and execute Python code to use Search
                model_turn = chunk.server_content.model_turn
                if model_turn:
                    for part in model_turn.parts:
                      if part.executable_code is not None:
                        print(part.executable_code.code)

                      if part.code_execution_result is not None:
                        print(part.code_execution_result.output)

if __name__ == "__main__":
    asyncio.run(main())
```

### Combining multiple tools

You can combine multiple tools within the Live API:

```
prompt = """
Hey, I need you to do three things for me.

1. Compute the largest prime palindrome under 100000.
2. Then use Google Search to look up information about the largest earthquake in California the week of Dec 5 2024?
3. Turn on the lights

Thanks!
"""

tools = [
    {"google_search": {}},
    {"code_execution": {}},
    {"function_declarations": [turn_on_the_lights, turn_off_the_lights]},
]

config = {"response_modalities": ["TEXT"], "tools": tools}
```

## Handling interruptions

Users can interrupt the model's output at any time. When Voice activity
detection (VAD) detects an interruption, the ongoing
generation is canceled and discarded. Only the information already sent to the
client is retained in the session history. The server then sends a BidiGenerateContentServerContent message to report the interruption.

In addition, the Gemini server discards any pending function calls and sends
a `BidiGenerateContentServerContent` message with the IDs of the canceled calls.

```
async for response in session.receive():
    if response.server_content.interrupted is True:
        # The generation was interrupted
```

### Voice activity detection (VAD)

You can configure or disable voice activity detection (VAD).

#### Using automatic VAD

By default, the model automatically performs VAD on
a continuous audio input stream. VAD can be configured with the  `realtimeInputConfig.automaticActivityDetection`  field of the setup configuration .

## Token count

You can find the total number of consumed tokens in the usageMetadata field of the returned server message.

```
async for message in session.receive():
    # The server will periodically send messages that include UsageMetadata.
    if message.usage_metadata:
        usage = message.usage_metadata
        print(
            f"Used {usage.total_token_count} tokens in total. Response token breakdown:"
        )
        for detail in usage.response_tokens_details:
            match detail:
                case types.ModalityTokenCount(modality=modality, token_count=count):
                    print(f"{modality}: {count}")
```

## Extending the session duration

The maximum session duration can be extended to
unlimited with two mechanisms:

Furthermore, you'll receive a GoAway message before the
session ends, allowing you to take further actions.

### Context window compression

To enable longer sessions, and avoid abrupt connection termination, you can
enable context window compression by setting the contextWindowCompression field as part of the session configuration.

In the ContextWindowCompressionConfig , you
can configure a sliding-window mechanism and the number of tokens that triggers compression.

```
from google.genai import types

config = types.LiveConnectConfig(
    response_modalities=["AUDIO"],
    context_window_compression=(
        # Configures compression with default parameters.
        types.ContextWindowCompressionConfig(
            sliding_window=types.SlidingWindow(),
        )
    ),
)
```

### Receiving a message before the session disconnects

The server sends a GoAway message that signals that the current
connection will soon be terminated. This message includes the timeLeft ,
indicating the remaining time and lets you take further action before the
connection will be terminated as ABORTED.

```
async for response in session.receive():
    if response.go_away is not None:
        # The connection will soon be terminated
        print(response.go_away.time_left)
```

### Receiving a message when the generation is complete

The server sends a generationComplete message that signals that the model finished generating the response.

```
async for response in session.receive():
    if response.server_content.generation_complete is True:
        # The generation is complete
```

## Media resolution

You can specify the media resolution for the input media by setting the `mediaResolution` field as part of the session configuration:

```
from google.genai import types

config = types.LiveConnectConfig(
    response_modalities=["AUDIO"],
    media_resolution=types.MediaResolution.MEDIA_RESOLUTION_LOW,
)
```

## Limitations

Consider the following limitations of the Live API
when you plan your project.

### Response modalities

You can only set one response modality ( `TEXT` or `AUDIO` ) per session in the
session configuration. Setting both results in a config error message. This
means that you can configure the model to respond with either text or audio,
but not both in the same session.

### Client authentication

The Live API only provides server to server authentication
and isn't recommended for direct client use. Client input should be routed
through an intermediate application server for secure authentication with
the Live API.

### Session duration

Session duration can be extended to unlimited by enabling session compression .
Without compression, audio-only sessions are limited to 15 minutes,
and audio plus video sessions are limited to 2 minutes. Exceeding these limits
without compression will terminate the connection.

Additionally, you can configure session resumption to
allow the client to resume a session that was terminated.

### Context window

A session has a context window limit of:

- 128k tokens for native audio output models
- 32k tokens for other Live API models

## Supported languages

Live API supports the following languages.

**Note:** Native audio output models automatically choose
the appropriate language and don't support explicitly setting the language
code.
| Language | BCP-47 Code |
| --- | --- |
| German (Germany) | de-DE |
| English (Australia) | en-AU |
| English (United Kingdom) | en-GB |
| English (India) | en-IN |
| English (US) | en-US |
| Spanish (United States) | es-US |
| French (France) | fr-FR |
| Hindi (India) | hi-IN |
| Portuguese (Brazil) | pt-BR |
| Arabic (Generic) | ar-XA |
| Spanish (Spain) | es-ES |
| French (Canada) | fr-CA |
| Indonesian (Indonesia) | id-ID |
| Italian (Italy) | it-IT |
| Japanese (Japan) | ja-JP |
| Turkish (Turkey) | tr-TR |
| Vietnamese (Vietnam) | vi-VN |
| Bengali (India) | bn-IN |
| Gujarati (India) | gu-IN |
| Kannada (India) | kn-IN |
| Malayalam (India) | ml-IN |
| Marathi (India) | mr-IN |
| Tamil (India) | ta-IN |
| Telugu (India) | te-IN |
| Dutch (Netherlands) | nl-NL |
| Korean (South Korea) | ko-KR |
| Mandarin Chinese (China) | cmn-CN |
| Polish (Poland) | pl-PL |
| Russian (Russia) | ru-RU |
| Thai (Thailand) | th-TH |


# Example of Live API tool calling

```
const changeBackgroundColorFunctionDeclaration = {
  name: 'change_background_color',
  behavior: Behavior.NON_BLOCKING,
  description: 'Change the background color of the page',
  parameters: {
    type: Type.OBJECT,
    properties: {
      color: {
        type: Type.STRING,
        description: 'The hex value of the color code (eg: #121212)',
      },
    },
    required: ['color'],
  },
};

function changeBackgroundColor(args) {
  document.body.style.background = args.color;
}

const functionDefs = {
  'change_background_color': changeBackgroundColor
}

let genAIConfig = {
  responseModalities: [Modality.AUDIO], 
  tools: [
    googleSearchTool, // Use grounding with Google Search
    {
      functionDeclarations: [
        changeBackgroundColorFunctionDeclaration
      ]
    }
  ]
};

session = await ai.live.connect({
  model: model,
  callbacks: {
    onmessage: async function (message) {
      if (message.toolCall) {
        const functionCall = message.toolCall.functionCalls[0];
        console.log(`Function to call: ${functionCall.name}, Arguments: ${functionCall.args}`);
        functionDefs[functionCall.name].call(null, functionCall.args);
      }

      if (message.serverContent?.modelTurn?.parts) {
        for (const part of message.serverContent.modelTurn.parts) {
          if (part.text) {
            // Process Text
          } else if (part.inlineData?.mimeType.startsWith('audio/')) {
            // Process audio
          }
        }
      }
    },
  },
  config: genAIConfig,
});
```