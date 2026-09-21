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