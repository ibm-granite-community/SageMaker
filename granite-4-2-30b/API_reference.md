# API Reference

This document describes the input schema and example request/response formats for IBM Granite 4.2 30B on Amazon SageMaker, based on the provided sample files (`chat_sample_*`, `real_time_sample_*`, and `tool_sample_*`). Use these examples to format requests and to parse responses from the model.

The endpoint runs the vLLM inference server. Requests are sent to the SageMaker `InvokeEndpoint` API (`ContentType` and `Accept` of `application/json`) and use the OpenAI-style chat completions format: a JSON body with a `messages` array. Generation parameters are top-level fields of the body. The older `inputs` / `parameters` format used by some other Granite packages is **not** used by this model.

Model facts:

| Item | Value |
| ---- | ----- |
| Architecture | Dense transformer, about 30B parameters |
| Precision | BF16 |
| License | Apache 2.0 |
| Context window | 128K tokens (131,072), prompt and generated tokens combined |
| Tested real-time instance | `ml.g7.24xlarge` (the only instance type tested) |
| Host image version | `al2023-ami-sagemaker-inference-gpu-4-1` |
| Batch transform | Not available for this model |

---

## Input schema

The fields you can use to run this model with an API. If you don't give a value for a field its default value will be used.

| Command                   |    Type | Default value | Description                                                                                                  |
| ------------------------- | ------: | ------------- | ------------------------------------------------------------------------------------------------------------ |
| `messages`                |   array |               | Required. Chat messages, each `{"role": "system" \| "user" \| "assistant" \| "tool", "content": "..."}`.     |
| `max_tokens`              | integer | model limit   | Maximum number of tokens to generate. Thinking tokens count toward this limit, so leave room for them.       |
| `temperature`             |  number | `1.0`         | Controls randomness of sampling. IBM recommends `1.0` for this model. Use `0` for deterministic output.      |
| `top_p`                   |  number | `1.0`         | Nucleus sampling threshold. IBM recommends `0.95` for this model.                                            |
| `top_k`                   | integer | `-1` (off)    | Top-k sampling: number of highest-probability tokens considered at each step.                                |
| `presence_penalty`        |  number | `0`           | Penalizes tokens based on whether they appear in the text so far.                                            |
| `frequency_penalty`       |  number | `0`           | Penalizes tokens that have already appeared to reduce repetition.                                            |
| `stop`                    | string or array |       | Sequence(s) where generation should stop.                                                                    |
| `seed`                    | integer |               | Random seed for repeatable sampling.                                                                         |
| `n`                       | integer | `1`           | Number of completions to generate.                                                                           |
| `stream`                  | boolean | `false`       | Streaming is not covered by the samples; use non-streaming `invoke_endpoint` unless you have tested streaming. |
| `tools`                   |   array |               | Function definitions the model may call (see below).                                                         |
| `tool_choice`             | string or object | `auto` when `tools` is set | `"auto"`, `"none"`, `"required"`, or `{"type": "function", "function": {"name": "..."}}`.   |
| `chat_template_kwargs`    |  object |               | Extra options for the chat template. `{"enable_thinking": false}` turns thinking off.                        |

Other standard vLLM OpenAI-compatible chat fields (for example `logprobs`, `response_format`) are passed to the server; they are not exercised in the samples.

---

## Standard chat request (example)

Thinking is on by default. See `chat_sample_input_data.json`.

```json
{
  "messages": [
    { "role": "user", "content": "A farmer has 10 cows, 5 chickens, and 2 horses. How many legs are there in total?" }
  ],
  "temperature": 1.0,
  "top_p": 0.95,
  "max_tokens": 4096
}
```

### Corresponding chat response (structured)

The final answer is in `message.content`. The thinking text is returned separately in `message.reasoning` (some builds name this field `reasoning_content`, so read both if you need to be portable). `usage.reasoning_tokens` reports how many completion tokens were spent on thinking.

```json
{
  "id": "chatcmpl-...",
  "object": "chat.completion",
  "created": 1794268800,
  "model": "/opt/ml/model",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "**Leg count** ... Total: 40 + 10 + 8 = **58 legs**.",
        "tool_calls": [],
        "reasoning": "Okay, let's see. The farmer has 10 cows, 5 chickens, and 2 horses ..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 39,
    "total_tokens": 659,
    "completion_tokens": 620,
    "reasoning_tokens": 327
  }
}
```

> Note: The three sample output files are complete, unedited responses captured on 2026-11-11 from the `vllm-sm4` container running the production configuration (bfloat16, tensor-parallel 4, default 128K context) on a 4-GPU instance (`g7.24xlarge`). The `model` field is the path `/opt/ml/model`. `id` and `created` change on every request, and the generated text varies from run to run. The thinking-off and tool sample inputs are the exact requests that produced the outputs. Fields such as `refusal`, `annotations`, `audio` and `function_call` are typically `null`.

---

## Thinking on / off

Granite 4.2 thinks before answering by default. To skip the thinking phase (faster and cheaper, suitable for simple requests), send:

```json
{
  "messages": [{ "role": "user", "content": "..." }],
  "temperature": 1.0,
  "top_p": 0.95,
  "max_tokens": 1024,
  "chat_template_kwargs": { "enable_thinking": false }
}
```

With thinking off, `message.reasoning` is empty/`null` and only `content` is returned. See `real_time_sample_input_data.json` and `real_time_sample_output_data.json`.

---

## Tool / function-call integration

The endpoint is started with automatic tool choice enabled and a tool-call parser, so function calling works through the standard `tools` array. The model's response contains a `tool_calls` entry when a function is invoked, with `finish_reason` `tool_calls`. Run the function yourself, then send its result back as a message with `"role": "tool"` and the matching `tool_call_id`.

### Example tool description (from samples)

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get the current weather in a city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": { "type": "string" }
          },
          "required": ["city"]
        }
      }
    }
  ],
  "tool_choice": "auto"
}
```

### Example tool call returned in response

`function.arguments` is a JSON string. Parse it with `json.loads`.

```json
"tool_calls": [
  {
    "id": "chatcmpl-tool-88e1d1e6f3fd4350",
    "type": "function",
    "function": {
      "name": "get_current_weather",
      "arguments": "{\"city\": \"Boston\"}"
    }
  }
]
```

When tools are available the model may also place its thinking in `message.reasoning` (for example, "Okay, the user is asking for the current weather in Boston. I need to use the get_current_weather tool ...").

---

## Context length

The endpoint is configured for the model's full 128K token (131,072) context window. The sum of prompt tokens and `max_tokens` must not exceed it. A test with a prompt of about 100,000 tokens (a hidden code `MANGO-7421` in filler text) was answered correctly. Long prompts need more time and GPU memory; the tested instance (`ml.g7.24xlarge`) handled one such request. Concurrency at full context length has not been characterized.

---

## Instances and batch transform

- Real-time inference: `ml.g7.24xlarge` is the only instance type this model package has been tested on. The deployment needs `inference_ami_version="al2023-ami-sagemaker-inference-gpu-4-1"`. Other instance types listed on the Marketplace page have not been tested.
- Batch transform: not available for this model (see `batch_sample_*_data.json`).
