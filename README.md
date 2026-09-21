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