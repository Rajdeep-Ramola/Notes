# 🗣️ Colang

> Colang is an event-driven interaction modeling language interpreted by a Python runtime, used to define guardrail flows in NeMo Guardrails.

---

## 📑 Table of Contents

1. [Event Streams](#1-event-streams)
2. [Flows](#2-flows)
3. [Testing](#3-testing)
4. [Dialog Rails](#4-dialog-rails)
5. [LLM Integration](#5-llm-integration)
6. [Multimodal Rails](#6-multimodal-rails)
7. [Input Rails](#7-input-rails)
8. [Interaction Loop](#8-interaction-loop)

---

## 1. Event Streams

Examples of events include: user saying something, the LLM generating a response, triggering an action, the result of an action, the retrieval of additional info, and the triggering of a guardrail.

![Guardrails Events Stream](https://docs.nvidia.com/nemo/guardrails/latest/_images/guardrails_events_stream.png)

---

## 2. Flows

The entry point for a Colang script is the `main` flow.

```colang
import core

flow main
  user said "hi"
  bot say "Hello World!"
```

In this example, `user said` and `bot say` are predefined flows. These flows are located in the `core` module included in the Colang standard library.

---

## 3. Testing

To test the above script, you can use the NeMo Guardrails CLI:

```bash
$ nemoguardrails chat --config=examples/v2_x/tutorial/hello_world_1
> hi
Hello World!
> something else is ignored
>
```

---

## 4. Dialog Rails

Enforce the path that dialog between the user and the bot should take. It usually consists of three components: definition of user messages, definition of bot messages, and definition of flows connecting user messages and bot messages.

```colang
import core

flow main
  user expressed greeting
  bot express greeting

flow user expressed greeting
  user said "hi" or user said "hello"

flow bot express greeting
  bot say "Hello world!"
```

---

## 5. LLM Integration

To use the LLM to drive the interaction for inputs that are not matched exactly by flows, you have to *activate* the `llm continuation` flow, which is part of the `llm` module in the Colang Standard Library.

```colang
import core
import llm

flow main
  activate llm continuation
  activate greeting

flow greeting
  user expressed greeting
  bot express greeting

flow user expressed greeting
  user said "hi" or user said "hello"

flow bot express greeting
  bot say "Hello world!"
```

> **Note:** If a flow is not activated (or called explicitly by another flow), it will not be used.

---

## 6. Multimodal Rails

A rail that handles multiple types of input/output modalities (e.g., text, voice, gestures, posture, image).

Colang provides an `avatars` module with flows for multimodal events and actions to implement interactive avatars use cases.

```colang
import core
import avatars

flow main
  user expressed greeting
  bot express greeting

flow user expressed greeting
  user expressed verbal greeting
    or user gestured "Greeting gesture"

flow user expressed verbal greeting
  user said "hi"
    or user said "hello"

flow bot express greeting
  bot express verbal greeting
    and bot gesture "Smile and wave with one hand."

flow bot express verbal greeting
  bot say "Hi there!"
    or bot say "Welcome!"
    or bot say "Hello!"
```

Here `user gestured` and `bot gesture` are predefined flows which match user gestures and control bot gestures.

---

## 7. Input Rails

A rail that checks the input from the user.

```colang
import core
import guardrails
import llm

flow main
  activate llm continuation
  activate greeting

flow greeting
  user expressed greeting
  bot express greeting

flow user expressed greeting
  user said "hi" or user said "hello"

flow bot express greeting
  bot say "Hello world!"

flow input rails $input_text
  $input_safe = await check user utterance $input_text
  if not $input_safe
    bot say "I'm sorry, I can't respond to that."
    abort

flow check user utterance $input_text -> $input_safe
  $is_safe = ..."Consider the following user utterance: '{$input_text}'. Assign 'True' if appropriate, 'False' if inappropriate."
  print $is_safe
  return $is_safe
```

---

## 8. Interaction Loop

When the LLM needs to interact with the user in a continuous interaction loop.

```colang
import core
import llm
import avatars
import timing

flow main
  activate automating intent detection
  activate generating user intent for unhandled user utterance

  while True
    when unhandled user intent
      $response = ..."Response to what user said."
      bot say $response

    or when user was silent 12.0
      bot inform about service

    or when user expressed greeting
      bot say "Hi there!"

    or when user expressed goodbye
      bot inform "That was fun. Goodbye"

flow user expressed greeting
  user said "hi"
    or user said "hello"

flow user expressed goodbye
  user said "goodbye"
    or user said "I am done"
    or user said "I have to go"

flow bot inform about service
  bot say "You can ask me anything!"
    or bot say "Just ask me something!"
```

> **Note:** This example uses `when / or when` syntax, which is a mechanism for branching a flow on multiple paths.
