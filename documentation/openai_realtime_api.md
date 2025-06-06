# Realtime APIBeta

Build low-latency, multi-modal experiences with the Realtime API.

The OpenAI Realtime API enables low-latency, multimodal interactions including speech-to-speech conversational experiences and real-time transcription.

This API works with natively multimodal models such as GPT-4o and GPT-4o mini , offering capabilities such as real-time text and audio processing, function calling, and speech generation, and with the latest transcription models GPT-4o Transcribe and GPT-4o mini Transcribe .

## Get started with the Realtime API

You can connect to the Realtime API in two ways:

- Using WebRTC , which is ideal for client-side applications (for example, a web app)
- Using WebSockets , which is great for server-to-server applications (from your backend or if you're building a voice agent over phone for example)


## Use cases

The most common use case for the Realtime API is to build a real-time, speech-to-speech, conversational experience. This is great for building voice agents and other voice-enabled applications.

The Realtime API can also be used independently for transcription and turn detection use cases. A client can stream audio in and have Realtime API produce streaming transcripts when speech is detected.

Both use-cases benefit from built-in voice activity detection (VAD) to automatically detect when a user is done speaking. This can be helpful to seamlessly handle conversation turns, or to analyze transcriptions one phrase at a time.

Depending on your use case (conversation or transcription), you should initialize a session in different ways.
Use the switcher below to see the details for each case.

## Connect with WebRTC

 WebRTC is a powerful set of standard interfaces for building real-time applications. The OpenAI Realtime API supports connecting to realtime models through a WebRTC peer connection. Follow this guide to learn how to configure a WebRTC connection to the Realtime API.

### Overview

In scenarios where you would like to connect to a Realtime model from an insecure client over the network (like a web browser), we recommend using the WebRTC connection method. WebRTC is better equipped to handle variable connection states, and provides a number of convenient APIs for capturing user audio inputs and playing remote audio streams from the model.

Connecting to the Realtime API from the browser should be done with an ephemeral API key, generated via the OpenAI REST API . The process for initializing a WebRTC connection is as follows (assuming a web browser client):

- A browser makes a request to a developer-controlled server to mint an ephemeral API key.
- The developer's server uses a standard API key to request an ephemeral key from the OpenAI REST API , and returns that new key to the browser. Note that ephemeral keys currently expire one minute after being issued.
- The browser uses the ephemeral key to authenticate a session directly with the OpenAI Realtime API as a WebRTC peer connection .

While it is technically possible to use a standard API key to authenticate client-side WebRTC sessions,**this is a dangerous and insecure practice**because it leaks your secret key. Standard API keys grant access to your full OpenAI API account, and should only be used in secure server-side environments. We recommend ephemeral keys in client-side applications whenever possible.

### Connection details

Connecting via WebRTC requires the following connection information:

| URL | https://api.openai.com/v1/realtime |
| --- | --- |
| Query Parameters | model  Realtime model ID to connect to, like gpt-4o-realtime-preview-2024-12-17 |
| Headers | Authorization: Bearer EPHEMERAL_KEY  Substitute EPHEMERAL_KEY with an ephemeral API token - see below for details on how to generate one. |

The following example shows how to initialize a WebRTC session (including the data channel to send and receive Realtime API events). It assumes you have already fetched an ephemeral API token (example server code for this can be found in the next section ).

```
async function init() {
  // Get an ephemeral key from your server - see server code below
  const tokenResponse = await fetch("/session");
  const data = await tokenResponse.json();
  const EPHEMERAL_KEY = data.client_secret.value;

  // Create a peer connection
  const pc = new RTCPeerConnection();

  // Set up to play remote audio from the model
  const audioEl = document.createElement("audio");
  audioEl.autoplay = true;
  pc.ontrack = e => audioEl.srcObject = e.streams[0];

  // Add local audio track for microphone input in the browser
  const ms = await navigator.mediaDevices.getUserMedia({
    audio: true
  });
  pc.addTrack(ms.getTracks()[0]);

  // Set up data channel for sending and receiving events
  const dc = pc.createDataChannel("oai-events");
  dc.addEventListener("message", (e) => {
    // Realtime server events appear here!
    console.log(e);
  });

  // Start the session using the Session Description Protocol (SDP)
  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);

  const baseUrl = "https://api.openai.com/v1/realtime";
  const model = "gpt-4o-realtime-preview-2024-12-17";
  const sdpResponse = await fetch(`${baseUrl}?model=${model}`, {
    method: "POST",
    body: offer.sdp,
    headers: {
      Authorization: `Bearer ${EPHEMERAL_KEY}`,
      "Content-Type": "application/sdp"
    },
  });

  const answer = {
    type: "answer",
    sdp: await sdpResponse.text(),
  };
  await pc.setRemoteDescription(answer);
}

init();
```

The WebRTC APIs provide rich controls for handling media streams and input devices. For more guidance on building user interfaces on top of WebRTC, refer to the docs on MDN .

### Creating an ephemeral token

To create an ephemeral token to use on the client-side, you will need to build a small server-side application (or integrate with an existing one) to make an OpenAI REST API request for an ephemeral key. You will use a standard API key to authenticate this request on your backend server.

Below is an example of a simple Node.js express server which mints an ephemeral API key using the REST API:

```
import express from "express";

const app = express();

// An endpoint which would work with the client code above - it returns
// the contents of a REST API request to this protected endpoint
app.get("/session", async (req, res) => {
  const r = await fetch("https://api.openai.com/v1/realtime/sessions", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      model: "gpt-4o-realtime-preview-2024-12-17",
      voice: "verse",
    }),
  });
  const data = await r.json();

  // Send back the JSON we received from the OpenAI REST API
  res.send(data);
});

app.listen(3000);
```

You can create a server endpoint like this one on any platform that can send and receive HTTP requests. Just ensure that**you only use standard OpenAI API keys on the server, not in the browser.**

### Sending and receiving events

To learn how to send and receive events over the WebRTC data channel, refer to the Realtime conversations guide .

## Connect with WebSockets

 WebSockets are a broadly supported API for realtime data transfer, and a great choice for connecting to the OpenAI Realtime API in server-to-server applications. For browser and mobile clients, we recommend connecting via WebRTC .

### Overview

In a server-to-server integration with Realtime, your backend system will connect via WebSocket directly to the Realtime API. You can use a standard API key to authenticate this connection, since the token will only be available on your secure backend server.

WebSocket connections can also be authenticated with an ephemeral client token ( as shown above in the WebRTC section ) if you choose to connect to the Realtime API via WebSocket on a client device.

Standard OpenAI API tokens**should only be used in secure server-side environments**.

### Connection details

Connecting via WebSocket requires the following connection information:

| URL | wss://api.openai.com/v1/realtime |
| --- | --- |
| Query Parameters | model  Realtime model ID to connect to, like gpt-4o-realtime-preview-2024-12-17 |
| Headers | Authorization: Bearer YOUR_API_KEY  Substitute YOUR_API_KEY with a standard API key on the server, or an ephemeral token on insecure clients (note that WebRTC is recommended for this use case).OpenAI-Beta: realtime=v1  This header is required during the beta period. |

Below are several examples of using these connection details to initialize a WebSocket connection to the Realtime API.

Connect using the ws module (Node.js)

```
import WebSocket from "ws";

const url = "wss://api.openai.com/v1/realtime?model=gpt-4o-realtime-preview-2024-12-17";
const ws = new WebSocket(url, {
  headers: {
    "Authorization": "Bearer " + process.env.OPENAI_API_KEY,
    "OpenAI-Beta": "realtime=v1",
  },
});

ws.on("open", function open() {
  console.log("Connected to server.");
});

ws.on("message", function incoming(message) {
  console.log(JSON.parse(message.toString()));
});
```

### Sending and receiving events

To learn how to send and receive events over Websockets, refer to the Realtime conversations guide .

-----------------

# Realtime conversationsBeta

Learn how to manage Realtime speech-to-speech conversations.

Once you have connected to the Realtime API through either WebRTC or WebSocket , you can call a Realtime model (such as gpt-4o-realtime-preview ) to have speech-to-speech conversations. Doing so will require you to**send client events**to initiate actions, and**listen for server events**to respond to actions taken by the Realtime API.

This guide will walk through the event flows required to use model capabilities like audio and text generation and function calling, and how to think about the state of a Realtime Session.

If you do not need to have a conversation with the model, meaning you don't expect any response, you can use the Realtime API in transcription mode .

## Realtime speech-to-speech sessions

A Realtime Session is a stateful interaction between the model and a connected client. The key components of the session are:

- The**Session**object, which controls the parameters of the interaction, like the model being used, the voice used to generate output, and other configuration.
- A**Conversation**, which represents user input Items and model output Items generated during the current session.
- **Responses**, which are model-generated audio or text Items that are added to the Conversation.

**Input audio buffer and WebSockets**

If you are using WebRTC, much of the media handling required to send and receive audio from the model is assisted by WebRTC APIs.

If you are using WebSockets for audio, you will need to manually interact with the**input audio buffer**by sending audio to the server, sent with JSON events with base64-encoded audio.

All these components together make up a Realtime Session. You will use client events to update the state of the session, and listen for server events to react to state changes within the session.

## Session lifecycle events

After initiating a session via either WebRTC or WebSockets , the server will send a `session.created` event indicating the session is ready. On the client, you can update the current session configuration with the `session.update` event. Most session properties can be updated at any time, except for the`voice`the model uses for audio output, after the model has responded with audio once during the session. The maximum duration of a Realtime session is**30 minutes**.

The following example shows updating the session with a`session.update`client event. See the WebRTC or WebSocket guide for more on sending client events over these channels.

Update the system instructions used by the model in this session

```
const event = {
  type: "session.update",
  session: {
    instructions: "Never use the word 'moist' in your responses!"
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

When the session has been updated, the server will emit a `session.updated` event with the new state of the session.

| Related client events | Related server events |
| --- | --- |
| session.update | session.createdsession.updated |

## Text inputs and outputs

To generate text with a Realtime model, you can add text inputs to the current conversation, ask the model to generate a response, and listen for server-sent events indicating the progress of the model's response. In order to generate text, the session must be configured with the`text`modality (this is true by default).

Create a new text conversation item using the `conversation.item.create` client event. This is similar to sending a user message (prompt) in Chat Completions in the REST API.

Create a conversation item with user input

```
const event = {
  type: "conversation.item.create",
  item: {
    type: "message",
    role: "user",
    content: [
      {
        type: "input_text",
        text: "What Prince album sold the most copies?",
      }
    ]
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

After adding the user message to the conversation, send the `response.create` event to initiate a response from the model. If both audio and text are enabled for the current session, the model will respond with both audio and text content. If you'd like to generate text only, you can specify that when sending the`response.create`client event, as shown below.

Generate a text-only response

```
const event = {
  type: "response.create",
  response: {
    modalities: [ "text" ]
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

When the response is completely finished, the server will emit the `response.done` event. This event will contain the full text generated by the model, as shown below.

Listen for response.done to see the final results

```
function handleEvent(e) {
  const serverEvent = JSON.parse(e.data);
  if (serverEvent.type === "response.done") {
    console.log(serverEvent.response.output[0]);
  }
}

// Listen for server messages (WebRTC)
dataChannel.addEventListener("message", handleEvent);

// Listen for server messages (WebSocket)
// ws.on("message", handleEvent);
```

While the model response is being generated, the server will emit a number of lifecycle events during the process. You can listen for these events, such as `response.text.delta` , to provide realtime feedback to users as the response is generated. A full listing of the events emitted by there server are found below under**related server events**. They are provided in the rough order of when they are emitted, along with relevant client-side events for text generation.

| Related client events | Related server events |
| --- | --- |
| conversation.item.createresponse.create | conversation.item.createdresponse.createdresponse.output_item.addedresponse.content_part.addedresponse.text.deltaresponse.text.doneresponse.content_part.doneresponse.output_item.doneresponse.donerate_limits.updated |

## Audio inputs and outputs

One of the most powerful features of the Realtime API is voice-to-voice interaction with the model, without an intermediate text-to-speech or speech-to-text step. This enables lower latency for voice interfaces, and gives the model more data to work with around the tone and inflection of voice input.

### Voice options

Realtime sessions can be configured to use one of several built‑in voices when producing audio output. You can set the`voice`on session creation (or on a`response.create`) to control how the model sounds. Current voice options are`alloy`,`ash`,`ballad`,`coral`,`echo`,`sage`,`shimmer`, and`verse`. Once the model has emitted audio in a session, the`voice`cannot be modified for that session.

### Handling audio with WebRTC

If you are connecting to the Realtime API using WebRTC, the Realtime API is acting as a peer connection to your client. Audio output from the model is delivered to your client as a remote media stream . Audio input to the model is collected using audio devices ( `getUserMedia` ), and media streams are added as tracks to to the peer connection.

The example code from the WebRTC connection guide shows a basic example of configuring both local and remote audio using browser APIs:

```
// Create a peer connection
const pc = new RTCPeerConnection();

// Set up to play remote audio from the model
const audioEl = document.createElement("audio");
audioEl.autoplay = true;
pc.ontrack = e => audioEl.srcObject = e.streams[0];

// Add local audio track for microphone input in the browser
const ms = await navigator.mediaDevices.getUserMedia({
  audio: true
});
pc.addTrack(ms.getTracks()[0]);
```

The snippet above enables simple interaction with the Realtime API, but there's much more that can be done. For more examples of different kinds of user interfaces, check out the WebRTC samples repository. Live demos of these samples can also be found here .

Using media captures and streams in the browser enables you to do things like mute and unmute microphones, select which device to collect input from, and more.

### Client and server events for audio in WebRTC

By default, WebRTC clients don't need to send any client events to the Realtime API before sending audio inputs. Once a local audio track is added to the peer connection, your users can just start talking!

However, WebRTC clients still receive a number of server-sent lifecycle events as audio is moving back and forth between client and server over the peer connection. Examples include:

- When input is sent over the local media track, you will receive `input_audio_buffer.speech_started` events from the server.
- When local audio input stops, you'll receive the `input_audio_buffer.speech_stopped` event.
- You'll receive delta events for the in-progress audio transcript .
- You'll receive a `response.done` event when the model has transcribed and completed sending a response.

Manipulating WebRTC APIs for media streams may give you all the control you need. However, it may occasionally be necessary to use lower-level interfaces for audio input and output. Refer to the WebSockets section below for more information and a listing of events required for granular audio input handling.

### Handling audio with WebSockets

When sending and receiving audio over a WebSocket, you will have a bit more work to do in order to send media from the client, and receive media from the server. Below, you'll find a table describing the flow of events during a WebSocket session that are necessary to send and receive audio over the WebSocket.

The events below are given in lifecycle order, though some events (like the`delta`events) may happen concurrently.

| Lifecycle stage | Client events | Server events |
| --- | --- | --- |
| Session initialization | session.update | session.createdsession.updated |
| User audio input | conversation.item.create  (send whole audio message)input_audio_buffer.append  (stream audio in chunks)input_audio_buffer.commit  (used when VAD is disabled)response.create  (used when VAD is disabled) | input_audio_buffer.speech_startedinput_audio_buffer.speech_stoppedinput_audio_buffer.committed |
| Server audio output | input_audio_buffer.clear  (used when VAD is disabled) | conversation.item.createdresponse.createdresponse.output_item.createdresponse.content_part.addedresponse.audio.deltaresponse.audio_transcript.deltaresponse.text.deltaresponse.audio.doneresponse.audio_transcript.doneresponse.text.doneresponse.content_part.doneresponse.output_item.doneresponse.donerate_limits.updated |

### Streaming audio input to the server

To stream audio input to the server, you can use the `input_audio_buffer.append` client event. This event requires you to send chunks of**Base64-encoded audio bytes**to the Realtime API over the socket. Each chunk cannot exceed 15 MB in size.

The format of the input chunks can be configured either for the entire session, or per response.

- Session:`session.input_audio_format`in `session.update` 
- Response:`response.input_audio_format`in `response.create` 
Append audio input bytes to the conversation

```
import fs from 'fs';
import decodeAudio from 'audio-decode';

// Converts Float32Array of audio data to PCM16 ArrayBuffer
function floatTo16BitPCM(float32Array) {
  const buffer = new ArrayBuffer(float32Array.length * 2);
  const view = new DataView(buffer);
  let offset = 0;
  for (let i = 0; i < float32Array.length; i++, offset += 2) {
    let s = Math.max(-1, Math.min(1, float32Array[i]));
    view.setInt16(offset, s < 0 ? s * 0x8000 : s * 0x7fff, true);
  }
  return buffer;
}

// Converts a Float32Array to base64-encoded PCM16 data
base64EncodeAudio(float32Array) {
  const arrayBuffer = floatTo16BitPCM(float32Array);
  let binary = '';
  let bytes = new Uint8Array(arrayBuffer);
  const chunkSize = 0x8000; // 32KB chunk size
  for (let i = 0; i < bytes.length; i += chunkSize) {
    let chunk = bytes.subarray(i, i + chunkSize);
    binary += String.fromCharCode.apply(null, chunk);
  }
  return btoa(binary);
}

// Fills the audio buffer with the contents of three files,
// then asks the model to generate a response.
const files = [
  './path/to/sample1.wav',
  './path/to/sample2.wav',
  './path/to/sample3.wav'
];

for (const filename of files) {
  const audioFile = fs.readFileSync(filename);
  const audioBuffer = await decodeAudio(audioFile);
  const channelData = audioBuffer.getChannelData(0);
  const base64Chunk = base64EncodeAudio(channelData);
  ws.send(JSON.stringify({
    type: 'input_audio_buffer.append',
    audio: base64Chunk
  }));
});

ws.send(JSON.stringify({type: 'input_audio_buffer.commit'}));
ws.send(JSON.stringify({type: 'response.create'}));
```

### Send full audio messages

It is also possible to create conversation messages that are full audio recordings. Use the `conversation.item.create` client event to create messages with`input_audio`content.

Create full audio input conversation items

```
const fullAudio = "<a base64-encoded string of audio bytes>";

const event = {
  type: "conversation.item.create",
  item: {
    type: "message",
    role: "user",
    content: [
      {
        type: "input_audio",
        audio: fullAudio,
      },
    ],
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

### Working with audio output from a WebSocket

**To play output audio back on a client device like a web browser, we recommend using WebRTC rather than WebSockets**. WebRTC will be more robust sending media to client devices over uncertain network conditions.

But to work with audio output in server-to-server applications using a WebSocket, you will need to listen for `response.audio.delta` events containing the Base64-encoded chunks of audio data from the model. You will either need to buffer these chunks and write them out to a file, or maybe immediately stream them to another source like a phone call with Twilio .

Note that the `response.audio.done` and `response.done` events won't actually contain audio data in them - just audio content transcriptions. To get the actual bytes, you'll need to listen for the `response.audio.delta` events.

The format of the output chunks can be configured either for the entire session, or per response.

- Session:`session.output_audio_format`in `session.update` 
- Response:`response.output_audio_format`in `response.create` 
Listen for response.audio.delta events

```
function handleEvent(e) {
  const serverEvent = JSON.parse(e.data);
  if (serverEvent.type === "response.audio.delta") {
    // Access Base64-encoded audio chunks
    // console.log(serverEvent.delta);
  }
}

// Listen for server messages (WebSocket)
ws.on("message", handleEvent);
```

## Voice activity detection

By default, Realtime sessions have**voice activity detection (VAD)**enabled, which means the API will determine when the user has started or stopped speaking and respond automatically.

Read more about how to configure VAD in our voice activity detection guide.

### Disable VAD

VAD can be disabled by setting`turn_detection`to`null`with the `session.update` client event. This can be useful for interfaces where you would like to take granular control over audio input, like push to talk interfaces.

When VAD is disabled, the client will have to manually emit some additional client events to trigger audio responses:

- Manually send `input_audio_buffer.commit` , which will create a new user input item for the conversation.
- Manually send `response.create` to trigger an audio response from the model.
- Send `input_audio_buffer.clear` before beginning a new user input.

### Keep VAD, but disable automatic responses

If you would like to keep VAD mode enabled, but would just like to retain the ability to manually decide when a response is generated, you can set`turn_detection.interrupt_response`and`turn_detection.create_response`to`false`with the `session.update` client event. This will retain all the behavior of VAD but not automatically create new Responses. Clients can trigger these manually with a `response.create` event.

This can be useful for moderation or input validation or RAG patterns, where you're comfortable trading a bit more latency in the interaction for control over inputs.

## Create responses outside the default conversation

By default, all responses generated during a session are added to the session's conversation state (the "default conversation"). However, you may want to generate model responses outside the context of the session's default conversation, or have multiple responses generated concurrently. You might also want to have more granular control over which conversation items are considered while the model generates a response (e.g. only the last N number of turns).

Generating "out-of-band" responses which are not added to the default conversation state is possible by setting the`response.conversation`field to the string`none`when creating a response with the `response.create` client event.

When creating an out-of-band response, you will probably also want some way to identify which server-sent events pertain to this response. You can provide`metadata`for your model response that will help you identify which response is being generated for this client-sent event.

Create an out-of-band model response

```
const prompt = `
Analyze the conversation so far. If it is related to support, output
"support". If it is related to sales, output "sales".
`;

const event = {
  type: "response.create",
  response: {
    // Setting to "none" indicates the response is out of band
    // and will not be added to the default conversation
    conversation: "none",

    // Set metadata to help identify responses sent back from the model
    metadata: { topic: "classification" },

    // Set any other available response fields
    modalities: [ "text" ],
    instructions: prompt,
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

Now, when you listen for the `response.done` server event, you can identify the result of your out-of-band response.

Create an out-of-band model response

```
function handleEvent(e) {
  const serverEvent = JSON.parse(e.data);
  if (
    serverEvent.type === "response.done" &&
    serverEvent.response.metadata?.topic === "classification"
  ) {
    // this server event pertained to our OOB model response
    console.log(serverEvent.response.output[0]);
  }
}

// Listen for server messages (WebRTC)
dataChannel.addEventListener("message", handleEvent);

// Listen for server messages (WebSocket)
// ws.on("message", handleEvent);
```

### Create a custom context for responses

You can also construct a custom context that the model will use to generate a response, outside the default/current conversation. This can be done using the`input`array on a `response.create` client event. You can use new inputs, or reference existing input items in the conversation by ID.

Listen for out-of-band model response with custom context

```
const event = {
  type: "response.create",
  response: {
    conversation: "none",
    metadata: { topic: "pizza" },
    modalities: [ "text" ],

    // Create a custom input array for this request with whatever context
    // is appropriate
    input: [
      // potentially include existing conversation items:
      {
        type: "item_reference",
        id: "some_conversation_item_id"
      },
      {
        type: "message",
        role: "user",
        content: [
          {
            type: "input_text",
            text: "Is it okay to put pineapple on pizza?",
          },
        ],
      },
    ],
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

### Create responses with no context

You can also insert responses into the default conversation, ignoring all other instructions and context. Do this by setting`input`to an empty array.

Insert no-context model responses into the default conversation

```
const prompt = `
Say exactly the following:
I'm a little teapot, short and stout! 
This is my handle, this is my spout!
`;

const event = {
  type: "response.create",
  response: {
    // An empty input array removes existing context
    input: [],
    instructions: prompt,
  },
};

// WebRTC data channel and WebSocket both have .send()
dataChannel.send(JSON.stringify(event));
```

## Function calling

The Realtime models also support**function calling**, which enables you to execute custom code to extend the capabilities of the model. Here's how it works at a high level:

- When updating the session or creating a response , you can specify a list of available functions for the model to call.
- If when processing input, the model determines it should make a function call, it will add items to the conversation representing arguments to a function call.
- When the client detects conversation items that contain function call arguments, it will execute custom code using those arguments
- When the custom code has been executed, the client will create new conversation items that contain the output of the function call, and ask the model to respond.

Let's see how this would work in practice by adding a callable function that will provide today's horoscope to users of the model. We'll show the shape of the client event objects that need to be sent, and what the server will emit in turn.

### Configure callable functions

First, we must give the model a selection of functions it can call based on user input. Available functions can be configured either at the session level, or the individual response level.

- Session:`session.tools`property in `session.update` 
- Response:`response.tools`property in `response.create` 

Here's an example client event payload for a`session.update`that configures a horoscope generation function, that takes a single argument (the astrological sign for which the horoscope should be generated):

```
{
  "type": "session.update",
  "session": {
    "tools": [
      {
        "type": "function",
        "name": "generate_horoscope",
        "description": "Give today's horoscope for an astrological sign.",
        "parameters": {
          "type": "object",
          "properties": {
            "sign": {
              "type": "string",
              "description": "The sign for the horoscope.",
              "enum": [
                "Aries",
                "Taurus",
                "Gemini",
                "Cancer",
                "Leo",
                "Virgo",
                "Libra",
                "Scorpio",
                "Sagittarius",
                "Capricorn",
                "Aquarius",
                "Pisces"
              ]
            }
          },
          "required": ["sign"]
        }
      }
    ],
    "tool_choice": "auto",
  }
}
```

The`description`fields for the function and the parameters help the model choose whether or not to call the function, and what data to include in each parameter. If the model receives input that indicates the user wants their horoscope, it will call this function with a`sign`parameter.

### Detect when the model wants to call a function

Based on inputs to the model, the model may decide to call a function in order to generate the best response. Let's say our application adds the following conversation item and attempts to generate a response:

```
{
  "type": "conversation.item.create",
  "item": {
    "type": "message",
    "role": "user",
    "content": [
      {
        "type": "input_text",
        "text": "What is my horoscope? I am an aquarius."
      }
    ]
  }
}
```

Followed by a client event to generate a response:

```
{
  "type": "response.create"
}
```

Instead of immediately returning a text or audio response, the model will instead generate a response that contains the arguments that should be passed to a function in the developer's application. You can listen for realtime updates to function call arguments using the `response.function_call_arguments.delta` server event, but`response.done`will also have the complete data we need to call our function.

```
{
  "type": "response.done",
  "event_id": "event_AeqLA8iR6FK20L4XZs2P6",
  "response": {
    "object": "realtime.response",
    "id": "resp_AeqL8XwMUOri9OhcQJIu9",
    "status": "completed",
    "status_details": null,
    "output": [
      {
        "object": "realtime.item",
        "id": "item_AeqL8gmRWDn9bIsUM2T35",
        "type": "function_call",
        "status": "completed",
        "name": "generate_horoscope",
        "call_id": "call_sHlR7iaFwQ2YQOqm",
        "arguments": "{\"sign\":\"Aquarius\"}"
      }
    ],
    "usage": {
      "total_tokens": 541,
      "input_tokens": 521,
      "output_tokens": 20,
      "input_token_details": {
        "text_tokens": 292,
        "audio_tokens": 229,
        "cached_tokens": 0,
        "cached_tokens_details": { "text_tokens": 0, "audio_tokens": 0 }
      },
      "output_token_details": {
        "text_tokens": 20,
        "audio_tokens": 0
      }
    },
    "metadata": null
  }
}
```

In the JSON emitted by the server, we can detect that the model wants to call a custom function:

| Property | Function calling purpose |
| --- | --- |
| response.output[0].type | When set to function_call, indicates this response contains arguments for a named function call. |
| response.output[0].name | The name of the configured function to call, in this case generate_horoscope |
| response.output[0].arguments | A JSON string containing arguments to the function. In our case, "{\"sign\":\"Aquarius\"}". |
| response.output[0].call_id | A system-generated ID for this function call - you will need this ID to pass a function call result back to the model. |

Given this information, we can execute code in our application to generate the horoscope, and then provide that information back to the model so it can generate a response.

### Provide the results of a function call to the model

Upon receiving a response from the model with arguments to a function call, your application can execute code that satisfies the function call. This could be anything you want, like talking to external APIs or accessing databases.

Once you are ready to give the model the results of your custom code, you can create a new conversation item containing the result via the`conversation.item.create`client event.

```
{
  "type": "conversation.item.create",
  "item": {
    "type": "function_call_output",
    "call_id": "call_sHlR7iaFwQ2YQOqm",
    "output": "{\"horoscope\": \"You will soon meet a new friend.\"}"
  }
}
```

- The conversation item type is`function_call_output`
- `item.call_id`is the same ID we got back in the`response.done`event above
- `item.output`is a JSON string containing the results of our function call

Once we have added the conversation item containing our function call results, we again emit the`response.create`event from the client. This will trigger a model response using the data from the function call.

```
{
  "type": "response.create"
}
```

## Error handling

The `error` event is emitted by the server whenever an error condition is encountered on the server during the session. Occasionally, these errors can be traced to a client event that was emitted by your application.

Unlike HTTP requests and responses, where a response is implicitly tied to a request from the client, we need to use an`event_id`property on client events to know when one of them has triggered an error condition on the server. This technique is shown in the code below, where the client attempts to emit an unsupported event type.

```
const event = {
  event_id: "my_awesome_event",
  type: "scooby.dooby.doo",
};

dataChannel.send(JSON.stringify(event));
```

This unsuccessful event sent from the client will emit an error event like the following:

```
{
  "type": "invalid_request_error",
  "code": "invalid_value",
  "message": "Invalid value: 'scooby.dooby.doo' ...",
  "param": "type",
  "event_id": "my_awesome_event"
}
```

-----------------

# Realtime transcriptionBeta

Learn how to transcribe audio in real-time with the Realtime API.

You can use the Realtime API for transcription-only use cases, either with input from a microphone or from a file. For example, you can use it to generate subtitles or transcripts in real-time.
With the transcription-only mode, the model will not generate responses.

If you want the model to produce responses, you can use the Realtime API in speech-to-speech conversation mode .

## Realtime transcription sessions

To use the Realtime API for transcription, you need to create a transcription session, connecting via WebSockets or WebRTC .

Unlike the regular Realtime API sessions for conversations, the transcription sessions typically don't contain responses from the model.

The transcription session object is also different from regular Realtime API sessions:

```
{
  object: "realtime.transcription_session",
  id: string,
  input_audio_format: string,
  input_audio_transcription: [{
    model: string,
    prompt: string,
    language: string
  }],
  turn_detection: {
    type: "server_vad",
    threshold: float,
    prefix_padding_ms: integer,
    silence_duration_ms: integer,
  } | null,
  input_audio_noise_reduction: {
    type: "near_field" | "far_field"
  },
  include: list[string] | null
}
```

Some of the additional properties transcription sessions support are:

- `input_audio_transcription.model`: The transcription model to use, currently`gpt-4o-transcribe`,`gpt-4o-mini-transcribe`, and`whisper-1`are supported
- `input_audio_transcription.prompt`: The prompt to use for the transcription, to guide the model (e.g. "Expect words related to technology")
- `input_audio_transcription.language`: The language to use for the transcription, ideally in ISO-639-1 format (e.g. "en", "fr"...) to improve accuracy and latency
- `input_audio_noise_reduction`: The noise reduction configuration to use for the transcription
- `include`: The list of properties to include in the transcription events

Possible values for the input audio format are:`pcm16`(default),`g711_ulaw`and`g711_alaw`.

You can find more information about the transcription session object in the API reference .

## Handling transcriptions

When using the Realtime API for transcription, you can listen for the`conversation.item.input_audio_transcription.delta`and`conversation.item.input_audio_transcription.completed`events.

For`whisper-1`the`delta`event will contain full turn transcript, same as`completed`event. For`gpt-4o-transcribe`and`gpt-4o-mini-transcribe`the`delta`event will contain incremental transcripts as they are streamed out from the model.

Here is an example transcription delta event:

```
{
  "event_id": "event_2122",
  "type": "conversation.item.input_audio_transcription.delta",
  "item_id": "item_003",
  "content_index": 0,
  "delta": "Hello,"
}
```

Here is an example transcription completion event:

```
{
  "event_id": "event_2122",
  "type": "conversation.item.input_audio_transcription.completed",
  "item_id": "item_003",
  "content_index": 0,
  "transcript": "Hello, how are you?"
}
```

Note that ordering between completion events from different speech turns is not guaranteed. You should use`item_id`to match these events to the`input_audio_buffer.committed`events and use`input_audio_buffer.committed.previous_item_id`to handle the ordering.

To send audio data to the transcription session, you can use the`input_audio_buffer.append`event.

You have 2 options:

- Use a streaming microphone input
- Stream data from a wav file

## Voice activity detection

The Realtime API supports automatic voice activity detection (VAD). Enabled by default, VAD will control when the input audio buffer is committed, therefore when transcription begins.

Read more about configuring VAD in our Voice Activity Detection guide.

You can also disable VAD by setting the`turn_detection`property to`null`, and control when to commit the input audio on your end.

## Additional configurations

### Noise reduction

You can use the`input_audio_noise_reduction`property to configure how to handle noise reduction in the audio stream.

The possible values are:

- `near_field`: Use near-field noise reduction.
- `far_field`: Use far-field noise reduction.
- `null`: Disable noise reduction.

The default value is`near_field`, and you can disable noise reduction by setting the property to`null`.

### Using logprobs

You can use the`include`property to include logprobs in the transcription events, using`item.input_audio_transcription.logprobs`.

Those logprobs can be used to calculate the confidence score of the transcription.

```
{
  "type": "transcription_session.update",
  "input_audio_format": "pcm16",
  "input_audio_transcription": {
    "model": "gpt-4o-transcribe",
    "prompt": "",
    "language": ""
  },
  "turn_detection": {
    "type": "server_vad",
    "threshold": 0.5,
    "prefix_padding_ms": 300,
    "silence_duration_ms": 500,
  },
  "input_audio_noise_reduction": {
    "type": "near_field"
  },
  "include": [ 
    "item.input_audio_transcription.logprobs",
  ],
}
```

Was this page useful?

-----------------

# Voice activity detection (VAD)Beta

Learn about automatic voice activity detection in the Realtime API.

Voice activity detection (VAD) is a feature available in the Realtime API allowing to automatically detect when the user has started or stopped speaking.
It is enabled by default in speech-to-speech or transcription Realtime sessions, but is optional and can be turned off.

## Overview

When VAD is enabled, the audio is chunked automatically and the Realtime API sends events to indicate when the user has started or stopped speaking:

- `input_audio_buffer.speech_started`: The start of a speech turn
- `input_audio_buffer.speech_stopped`: The end of a speech turn

You can use these events to handle speech turns in your application. For example, you can use them to manage conversation state or process transcripts in chunks.

You can use the`turn_detection`property of the`session.update`event to configure how audio is chunked within each speech-to-text sample.

There are two modes for VAD:

- `server_vad`: Automatically chunks the audio based on periods of silence.
- `semantic_vad`: Chunks the audio when the model believes based on the words said by the user that they have completed their utterance.

The default value is`server_vad`.

Read below to learn more about the different modes.

## Server VAD

Server VAD is the default mode for Realtime sessions, and uses periods of silence to automatically chunk the audio.

You can adjust the following properties to fine-tune the VAD settings:

- `threshold`: Activation threshold (0 to 1). A higher threshold will require louder audio to activate the model, and thus might perform better in noisy environments.
- `prefix_padding_ms`: Amount of audio (in milliseconds) to include before the VAD detected speech.
- `silence_duration_ms`: Duration of silence (in milliseconds) to detect speech stop. With shorter values turns will be detected more quickly.

Here is an example VAD configuration:

```
{
  "type": "session.update",
  "session": {
    "turn_detection": {
      "type": "server_vad",
      "threshold": 0.5,
      "prefix_padding_ms": 300,
      "silence_duration_ms": 500,
      "create_response": true, // only in conversation mode
      "interrupt_response": true, // only in conversation mode
    }
  }
}
```

## Semantic VAD

Semantic VAD is a new mode that uses a semantic classifier to detect when the user has finished speaking, based on the words they have uttered.
This classifier scores the input audio based on the probability that the user is done speaking. When the probability is low, the model will wait for a timeout, whereas when it is high, there is no need to wait.
For example, user audio that trails off with an "ummm..." would result in a longer timeout than a definitive statement.

With this mode, the model is less likely to interrupt the user during a speech-to-speech conversation, or chunk a transcript before the user is done speaking.

Semantic VAD can be activated by setting`turn_detection.type`to`semantic_vad`in a `session.update` event.

It can be configured like this:

```
{
  "type": "session.update",
  "session": {
    "turn_detection": {
      "type": "semantic_vad",
      "eagerness": "low" | "medium" | "high" | "auto", // optional
      "create_response": true, // only in conversation mode
      "interrupt_response": true, // only in conversation mode
    }
  }
}
```

The optional`eagerness`property is a way to control how eager the model is to interrupt the user, tuning the maximum wait timeout. In transcription mode, even if the model doesn't reply, it affects how the audio is chunked.

- `auto`is the default value, and is equivalent to`medium`.
- `low`will let the user take their time to speak.
- `high`will chunk the audio as soon as possible.

If you want the model to respond more often in conversation mode, or to return transcription events faster in transcription mode, you can set`eagerness`to`high`.

On the other hand, if you want to let the user speak uninterrupted in conversation mode, or if you would like larger transcript chunks in transcription mode, you can set`eagerness`to`low`.
