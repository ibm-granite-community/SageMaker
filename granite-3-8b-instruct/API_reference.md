# Input schema
The fields you can use to run this model with an API. If you don't give a value for a field its default value will be used.

| Command | Type|	Default value | Description |
| --- | --- | --- | --- |
| `inference_type` | string | | forecasting
| `model_id` | String | ttm-r2 | model id is being supported currently.
| `parameters` | String |  | Blank dictionary.
| `schema.timestamp_column` | timestamp | | Timestamp.
| `schema.id_columns` | String | | single timeseries doesn't require id_columns.
| `schema.target_columns` | String | | what we're generating a forecast for.
| `data` |  | 50 | input data.
| `future_data` |  |  | used for things like exogenous data.

