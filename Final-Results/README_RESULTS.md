# README — Experimental Results

This file explains the columns and metrics reported in `Final-Results.xlsx`. The Excel file contains the aggregated experimental results for the federated continual-learning forecasting experiments. Each row corresponds to one combination of forecasting target, prediction horizon, and method.

## 1. Experimental Unit

Each row in the results table represents:

- one forecasting target, such as `PM2.5`, `TEMP`, or `WSPM`;
- one prediction horizon, such as 6, 12, or 24 time steps;
- one evaluated method, such as `Static`, `Naive`, `NormalReplay`, `TwoTierReplay`, `EWC`, `LwF`, or `SI`.

The columns ending with `_mean` report the average value across repeated runs/seeds. The columns ending with `_std` report the standard deviation across repeated runs/seeds.

Lower values are better for RMSE, forgetting, stability-plasticity score, number of drift events, and CPU time, unless a specific diagnostic interpretation is stated below.

## 2. Column Definitions

### `target`
The forecasting target variable.

Examples:

- `PM2.5`: fine particulate matter concentration;
- `TEMP`: temperature;
- `WSPM`: wind speed.

### `horizon`
The prediction horizon, expressed in the number of future time steps predicted by the model. For hourly data, horizons 6, 12, and 24 correspond to 6-hour, 12-hour, and 24-hour forecasting horizons.

### `method`
The evaluated forecasting/adaptation method.

### `stream_rmse_mean`
The mean RMSE on the continual-learning stream, averaged across repeated runs/seeds.

This metric measures plasticity/adaptation performance. A lower value means the method predicts the stream data more accurately.

### `stream_rmse_std`
The standard deviation of `stream_rmse` across repeated runs/seeds.

A smaller value indicates more stable performance across seeds.

### `base_model_base_rmse_mean`
The mean RMSE of the original offline base model evaluated on the base-period retention data.

This is the reference value used to measure forgetting. It represents the base model's performance before continual-learning adaptation.

### `base_model_base_rmse_std`
The standard deviation of `base_model_base_rmse` across repeated runs/seeds.

### `final_model_base_rmse_mean`
The mean RMSE of the final model after stream adaptation, evaluated again on the base-period retention data.

This metric measures how well the model retains the knowledge learned during the base/offline phase after continual learning.

### `final_model_base_rmse_std`
The standard deviation of `final_model_base_rmse` across repeated runs/seeds.

### `af_base_retain_mean`
The mean signed forgetting score on the base-period retention data.

It is computed as:

```text
AF_base = final_model_base_rmse - base_model_base_rmse
```

Interpretation:

- `AF_base > 0`: forgetting occurred; the final model became worse on base-period data.
- `AF_base = 0`: no change relative to the base model.
- `AF_base < 0`: the final model improved on base-period data; this may indicate positive backward transfer or improved retention.

Lower values are better.

### `af_base_retain_std`
The standard deviation of `af_base_retain` across repeated runs/seeds.

### `stability_plasticity_score_mean`
The mean signed stability-plasticity score.

It is computed as:

```text
stability_plasticity_score = stream_rmse + alpha_base * af_base_retain
```

In the reported experiments:

```text
alpha_base = 1.0
```

This score combines:

- plasticity: performance on the continual-learning stream, measured by `stream_rmse`;
- stability: retention of base-period knowledge, measured by `af_base_retain`.

Lower values are better. A method can obtain a better score either by improving stream forecasting accuracy, reducing forgetting, or both.

### `stability_plasticity_score_std`
The standard deviation of the stability-plasticity score across repeated runs/seeds.

### `total_drifts_mean`
The mean total number of detected drift events across all clients during the stream.

This counts client-level drift detections accumulated over the full stream.

### `total_drifts_std`
The standard deviation of `total_drifts` across repeated runs/seeds.

### `avg_drifts_client_mean`
The mean number of detected drifts per client.

It is computed as:

```text
avg_drifts_client = total_drifts / number_of_clients
```

### `avg_drifts_client_std`
The standard deviation of `avg_drifts_client` across repeated runs/seeds.

### `global_drift_events_mean`
The mean number of global drift events.

A global drift event occurs when at least one client triggers drift detection at a stream chunk and the federated adaptation procedure is activated.

This is different from `total_drifts_mean`, which counts client-level drift detections. One global drift event may involve one or multiple drifting clients.

### `global_drift_events_std`
The standard deviation of `global_drift_events` across repeated runs/seeds.

### `cpu_sec_mean`
The mean wall-clock runtime in seconds for the evaluated method, averaged across repeated runs/seeds.

This measures the runtime of the method evaluation/adaptation stage for a given target and horizon. Runtime may vary depending on hardware, device selection, and implementation details.

### `cpu_sec_std`
The standard deviation of `cpu_sec` across repeated runs/seeds.

## 4. How to Read the Results

For each target and horizon, compare methods primarily using:

```text
stability_plasticity_score_mean
```

The method with the lowest `stability_plasticity_score_mean` gives the best overall trade-off between stream adaptation and base-period retention under the signed score used in the experiments.

For additional interpretation:

- use `stream_rmse_mean` to compare pure stream forecasting performance;
- use `af_base_retain_mean` to compare forgetting or retention;
- use `cpu_sec_mean` to compare computational cost;
- use `total_drifts_mean` and `global_drift_events_mean` to understand how often adaptation was triggered.


A concise paper table can focus on `stream_rmse_mean`, `af_base_retain_mean`, and `stability_plasticity_score_mean`, while the full Excel file provides the complete diagnostics.
