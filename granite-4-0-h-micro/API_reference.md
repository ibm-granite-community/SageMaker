# API Reference

This document describes the input schema and example request/response formats for the model, based on the provided sample files (`chat_sample_*`, `real_time_sample_*`, and `tool_sample_*`). Use these examples to format requests and to parse responses from the model.

---

## Input schema

The fields you can use to run this model with an API. If you don't give a value for a field its default value will be used.

| Command                   |    Type | Default value                  | Description                                                                                     |
| ------------------------- | ------: | ------------------------------ | ----------------------------------------------------------------------------------------------- |
| `inputs`                  |  string |                                | Primary prompt or input text. For chat-style models this may contain a serialized chat context. |
| `system_prompt`           |  string | `You are a helpful assistant.` | System prompt prepended to the model context. Ignored for non-chat models.                      |
| `min_tokens`              | integer | `0`                            | Minimum number of tokens the model should generate.                                             |
| `max_tokens`              | integer | `512`                          | Maximum number of tokens the model should generate.                                             |
| `temperature`             |  number | `0.6`                          | Controls randomness of token sampling. Lower is more deterministic.                             |
| `top_p`                   |  number | `0.9`                          | Nucleus sampling cumulative probability threshold.                                              |
| `top_k`                   | integer | `50`                           | Top-k sampling: number of highest-probability tokens considered at each step.                   |
| `presence_penalty`        |  number | `0`                            | Penalizes new token choices based on whether they appear in the text so far.                    |
| `frequency_penalty`       |  number | `0`                            | Penalizes tokens that have already appeared to reduce repetition.                               |
| `stop_sequences`          |  string |                                | Comma-separated list of sequences where generation should stop (e.g. `"<end>,<stop>"`).         |
| `prompt_template`         |  string |                                | Optional template used to format the prompt before sending it to the model.                     |
| `seed`                    | integer |                                | Random seed for deterministic outputs. Leave blank to randomize.                                |
| `return_dict_in_generate` | boolean | `false`                        | If true, returns a dictionary with keys like `sequences`, `scores`, `attentions`.               |
| `return_full_text`        | boolean | `false`                        | If true, returns the entire prompt (input + generated text) rather than only generated text.    |
| `output_scores`           | boolean | `false`                        | If true, returns token-level probabilities/confidence scores for the generated output.          |

---

## Standard chat request (example)

Use this pattern when making chat-style requests. The `inputs` field can contain a combined or structured chat context depending on your API client.

```json
{
  "inputs": "<|start_of_role|>system<|end_of_role|> Knowledge Cutoff Date: April 2024. Today's Date: April 12, 2025. You are Granite, developed by IBM. You are a helpful AI assistant. Respond to every user query in a comprehensive and detailed way. ...<|end_of_text|><|start_of_role|>user<|end_of_role|>A farmer has 10 cows, 5 chickens, and 2 horses. If we count all the animals' legs together, how many legs are there in total?<|end_of_text|>",
  "temperature": 0.5,
  "top_p": 0.9,
  "top_k": 50,
  "max_tokens": 1024
}
```

### Corresponding chat response (structured)

A typical response object for chat-mode usage contains the generated text and associated metadata. Example fields you may see in returned JSON (fields vary by provider):

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "created": 1760694259,
  "model": "lmi",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "**Thought process**\n\nTo find the total number of legs...",
        "tool_calls": []
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 208,
    "completion_tokens": 172,
    "total_tokens": 380
  }
}
```

> Note: Some samples include an internal `reasoning_content` or `annotations` field. These are provider-specific and optional.

---

## Real-time (streaming / token-by-token) input example

Real-time clients may send a compact `inputs` string where role markers are embedded and specify generation parameters in `parameters`:

```json
{
  "inputs": "<|start_of_role|>system<|end_of_role|> Knowledge Cutoff Date: April 2024...<|end_of_text|><|start_of_role|>user<|end_of_role|>A farmer has 10 cows...",
  "parameters": {
    "do_sample": true,
    "top_p": 0.9,
    "temperature": 0.5,
    "top_k": 50,
    "max_new_tokens": 1024,
    "repetition_penalty": 1.00
  }
}
```

A real-time response may return a single `generated_text` string (or stream multiple partial fragments) — see provider docs for streaming protocols and chunked output formats.

---

## Tool / function-call integration

When the model is configured to call tools or functions, the request payload often includes a `tools` description array describing the callable functions. The model's final response will contain a `tool_calls` entry when a function is invoked.

### Example tool description (from samples)

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get the current weather in a given location",
        "parameters": {
          "type": "object",
          "properties": {
            "city": { "type": "string", "description": "The city to find the weather for" },
            "state": { "type": "string", "description": "Two-letter state code, e.g. 'TX'" },
            "unit": { "type": "string", "enum": ["celsius","fahrenheit"] }
          },
          "required": ["city","state","unit"]
        }
      }
    }
  ],
  "tool_choice": { "type": "function", "function": { "name": "get_current_weather" } }
}
```

### Example tool call returned in response

```json
"tool_calls": [
  {
    "id": "chatcmpl-tool-...",
    "type": "function",
    "function": {
      "name": "get_current_weather",
      "arguments": "{\n  \"city\": \"Dallas\",\n  \"state\": \"TX\",\n  \"unit\": \"fahrenheit\"\n}"
    }
  }
]
```