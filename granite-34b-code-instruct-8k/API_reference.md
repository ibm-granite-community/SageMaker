# Input schema
The fields you can use to run this model with an API. If you don't give a value for a field its default value will be used.

| Command | Type|	Default value | Description |
| --- | --- | --- | --- |
| `prompt` | string | | Prompt
| `system_prompt` | String | 	You are a helpful assistant. | System prompt to send to the model. This is prepended to the prompt and helps guide system behavior. Ignored for non-chat models.
| `min_tokens` | integer | 0 | The minimum number of tokens the model should generate as output.
| `max_tokens` | integer | 512 | The maximum number of tokens the model should generate as output.
| `temperature` | number | 0.6 | The value used to modulate the next token probabilities.
| `top_p` | number | 0.9 | A probability threshold for generating the output. If < 1.0, only keep the top tokens with cumulative probability >= top_p (nucleus filtering).
| `top_k` | integer | 50 | The number of highest probability tokens to consider for generating the output. If > 0, only keep the top k tokens with highest probability (top-k filtering).
| `presence_penalty` | number | 0 | Presence penalty.
| `frequency_penalty` | number | 0 | Frequency penalty.
| `stop_sequences` | string |  | A comma-separated list of sequences to stop generation at. For example, '<end>,<stop>' will stop generation at the first instance of 'end' or '<stop>'.
| `prompt_template` | string |  | A template to format the prompt with. If not provided, the default prompt template will be used.
| `seed` | integer |  | Random seed. Leave blank to randomize the seed.
