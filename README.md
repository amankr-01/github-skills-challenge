# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


## AIOps Monitoring Scenario

This project simulates monitoring for a `payment-service`. The service produces operational telemetry, including response times, CPU usage, memory usage, log levels, and log messages.

The operational problem is identifying unusual service behaviour, such as slow payment requests, high CPU or memory utilization, and warning or error conditions in the logs. These anomalies may indicate service degradation or a database connectivity problem.

AIOps is used in this assessment to analyze operational data, detect anomalies, generate structured events, and process those events automatically. The simulated workflow is:

**Operational Data → Anomaly Detection → Event Generation → Producer → Event Topic → Consumer → AIOps Output**

The main components are:

- `data/service_data.json`: Contains simulated service metrics and logs.
- `src/anomaly_detector.py`: Detects unusual response times, CPU usage, memory usage, and log conditions.
- `src/event_producer.py`: Publishes detected anomaly events.
- `src/event_topic.py`: Simulates an event topic for passing events.
- `src/event_consumer.py`: Consumes anomaly events.
- `src/aiops_pipeline.py`: Connects the complete workflow and displays the final AIOps results.

## Task 2: Operational Data Analysis

The data in `data/service_data.json` contains 10 observations for the `payment-service`, covering the period from `2026-09-20T10:00:00` through `2026-09-20T10:09:00`.

### Metrics

The metric fields are:

- `response_time_ms`: Payment request response time in milliseconds.
- `cpu_percent`: CPU utilization percentage.
- `memory_percent`: Memory utilization percentage.

The `service` field identifies which application service produced the measurements.

### Log Information

The log fields are:

- `log_level`: Log severity, such as `INFO` or `ERROR`.
- `message`: Text describing the service activity or failure.

### Timestamps

The `timestamp` field records when each observation occurred in ISO-style date-time format. The records are in chronological order and are sampled at one-minute intervals, which makes it possible to relate metric changes to log events over time.

### Normal Behaviour

The observations at `10:00` through `10:04` and `10:07` through `10:09` appear normal. They show successful payment processing, `INFO` logs, response times between 120 and 150 ms, CPU utilization between 42% and 50%, and memory utilization between 51% and 57%.

### Unusual Behaviour

- At `10:05`, the response time increases to 610 ms and the log reports `Payment service timeout` at `ERROR` level. This is unusual even though CPU and memory utilization remain below the anomaly thresholds.
- At `10:06`, the response time increases to 640 ms, CPU utilization reaches 94%, memory utilization reaches 91%, and the log reports `Database connection timeout` at `ERROR` level. This is the strongest indication of service degradation and may explain the slow payment requests.

## Task 3: Anomaly Detection Report

The provided `AnomalyDetector` was used with its default thresholds: response time above 500 ms, CPU above 80%, and memory above 80%. Warning and error log levels are also treated as concerning events. The pipeline processed all 10 operational records and distinguished the normal records from two anomalous records.

### Detected Anomalies

- `2026-09-20T10:05:00`: Flagged for high response time (`610 ms`) and a concerning `ERROR` log. The message was `Payment service timeout`.
- `2026-09-20T10:06:00`: Flagged for high response time (`640 ms`), high CPU utilization (`94%`), high memory utilization (`91%`), and a concerning `ERROR` log. The message was `Database connection timeout`.

The producer publishes both events to the `anomaly-events` topic, and the consumer receives both events. The final result is 10 records processed, 2 anomalies detected, and 2 events consumed. The eight records with `INFO` logs and normal metric values were not flagged.

### Detection Review

Both expected anomalies were detected, including the timeout at `10:05` and the combined performance and database problem at `10:06`. No normal event was incorrectly flagged in this dataset, and no expected anomaly was missed after including concerning `ERROR` log levels in the detector.

One limitation is that the detector uses fixed thresholds and does not learn the service's normal baseline or account for trends. A possible improvement would be to calculate a baseline from historical data and detect gradual changes or service-specific deviations automatically.

## Task 4: AIOps Event Flow Verification

The event flow was verified with the anomaly at `2026-09-20T10:05:00`:

1. `AnomalyDetector` examined the record and created an `ANOMALY` event because the response time was `610 ms` and the log level was `ERROR`.
2. `EventProducer.publish()` accepted the event and passed it to the producer's topic.
3. `EventTopic` stored the event in the in-memory `anomaly-events` topic.
4. `EventConsumer.consume()` read the event from that same topic.
5. The consumer returned the received event, including its timestamp, service, type, source record, and detection reasons.
6. `run_pipeline()` returned the consumed event in `events_consumed`, which is the downstream AIOps output displayed by the pipeline.

### Component Roles

- **Event/message**: A structured anomaly record containing the service, timestamp, original source data, and reasons for detection.
- **Producer**: Publishes each detected anomaly event to the event topic.
- **Topic**: Provides the in-memory channel that stores and transfers published events.
- **Consumer**: Reads events from the topic and makes them available to the downstream pipeline result.

The single-event verification passed from detector to producer, topic, consumer, and AIOps result. The complete dataset verification also passed: 10 records processed, 2 anomaly events produced, and the same 2 events consumed and returned by the pipeline.

## Task 5: Troubleshooting and Corrections

The original assessment code contained three workflow problems. Each was reproduced or identified from the component code, corrected within the existing architecture, and then verified by executing the affected component again.

### 1. Error logs were missed

- **Affected component:** `src/anomaly_detector.py`
- **Cause:** The detector checked only for `log_level == "WARNING"`, but the supplied anomalous records use `ERROR`.
- **Correction:** Treat both `WARNING` and `ERROR` as concerning log levels and include that reason in the generated event.
- **Verification:** An `ERROR`-only test record produced an anomaly with the reason `Concerning log level`. The dataset anomalies at `10:05` and `10:06` now include this reason.

### 2. The producer used the wrong topic

- **Affected component:** `src/aiops_pipeline.py` and the producer/topic connection.
- **Cause:** The producer published to a topic named `service-events`, while the consumer read from a separate topic named `anomaly-events`.
- **Correction:** Create one shared `anomaly-events` topic and pass that same topic instance to both `EventProducer` and `EventConsumer`.
- **Verification:** The full pipeline published both detected events to the shared topic, and the consumer received both events.

### 3. No events reached the downstream result

- **Affected component:** `src/aiops_pipeline.py` and the consumer-to-pipeline handoff.
- **Cause:** Because the consumer was attached to the empty second topic, `events_consumed` was empty even though the detector found two anomalies.
- **Correction:** Keep the consumer on the producer's shared topic so `consumer.consume()` returns the published events to `run_pipeline()`.
- **Verification:** Running `PYTHONPATH=.:src python3 src/aiops_pipeline.py` processed 10 records, detected 2 anomalies, and consumed 2 events. The final output included both timestamps and their detection reasons.

No unrelated components were replaced. The existing detector, producer, topic, consumer, and pipeline classes continue to provide the workflow architecture.

## Task 6: End-to-End Pipeline Execution

The corrected workflow was executed from the repository terminal with:

```bash
PYTHONPATH=.:src python3 src/aiops_pipeline.py
```

The execution verified the complete path:

1. **Operational data:** 10 records were loaded from `data/service_data.json`.
2. **Anomaly detection:** 2 abnormal observations were identified.
3. **Event generation:** Each abnormal observation produced a structured `ANOMALY` event with timestamp, service, source data, and detection reasons.
4. **Producer and topic:** The 2 events were published to the shared `anomaly-events` topic.
5. **Consumer:** Both published events were consumed successfully.
6. **AIOps processing:** The consumed events were returned by `run_pipeline()` and displayed by the command-line output.
The final output identified two operational issues in `payment-service`: a payment service timeout at `10:05` and a database connection timeout at `10:06`. The result was `10` records processed, `2` anomalies detected, and `2` events consumed.

## Task 7: Reproduction Steps

Another user can reproduce the demonstration as follows:

1. Open the repository in a Python-enabled environment and change to the repository root.
2. Install the test dependency if needed:

	```bash
	pip install -r requirements.txt
	```

3. Run the complete AIOps workflow:

	```bash
	PYTHONPATH=.:src python3 src/aiops_pipeline.py
	```

4. Confirm the output reports 10 records processed, 2 anomalies detected, and 2 events consumed. The output should identify the anomalies at `2026-09-20T10:05:00` and `2026-09-20T10:06:00`.
5. Run the automated tests to verify the individual workflow components:

	```bash
	PYTHONPATH=.:src python3 -m pytest -q
	```

The expected test result is 8 passing tests. The `PYTHONPATH=.:src` setting supports both the package-style imports used by the tests and the direct module imports used by `src/aiops_pipeline.py`.