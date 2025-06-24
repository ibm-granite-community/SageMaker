# Input Schema

The fields you can use to run this model with an API. If a field is not provided, its default value will be used.

| Command                   | Type     | Default Value                  | Description |
|---------------------------|----------|--------------------------------|-------------|
| `inputs`                  | string   |                                | Deprecated when using `messages`. Simple prompt string. |
| `messages`                | array    |                                | List of message objects for chat/multimodal interactions. Each contains a `role` and `content`. |
| `system_prompt`           | string   | "You are a helpful assistant." | Prepended to guide model behavior (used if no `system` role in `messages`). |
| `min_tokens`              | integer  | 0                              | Minimum number of tokens to generate. |
| `max_tokens`              | integer  | 512                            | Maximum number of tokens to generate. |
| `temperature`             | number   | 0.6                            | Controls randomness in output. Higher = more diverse. |
| `top_p`                   | number   | 0.9                            | Nucleus sampling; keeps top tokens with cumulative probability >= `top_p`. |
| `top_k`                   | integer  | 50                             | Top-K sampling; limits next token choices to top `k` probabilities. |
| `presence_penalty`        | number   | 0                              | Penalizes token selection based on whether it has appeared already. |
| `frequency_penalty`       | number   | 0                              | Penalizes token selection based on how frequently it has appeared. |
| `repetition_penalty`      | number   | 1                              | Penalty for repeated tokens. Higher = less repetition. |
| `stop_sequences`          | string   |                                | Comma-separated list of strings where generation will stop (e.g. "end,<stop>"). |
| `prompt_template`         | string   |                                | Custom prompt format; ignored if `messages` is used. |
| `seed`                    | integer  |                                | Random seed for deterministic output. Leave blank for random. |
| `return_dict_in_generate` | boolean  | false                          | If true, returns a dictionary with fields like sequences, scores, etc. |
| `return_full_text`        | boolean  | false                          | If true, includes prompt in output; otherwise, returns only generated part. |
| `output_scores`           | boolean  | false                          | If true, includes token-level confidence scores. |