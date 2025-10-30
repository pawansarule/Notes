<!-- Hyperion Architecture Overview
The Hyperion architecture is a system designed for observability that uses the Loki, Grafana, Tempo, and Mimir (LGTME) stack, though Mimir has been replaced by Victoria Metrics for the metrics backend. It leverages the OpenTelemetry (OTel) standard for collecting and processing telemetry data (logs, metrics, and traces). The entire system is built on Kubernetes and utilizes various Azure services, with core infrastructure management provided by Megatron, which offers the Azure Kubernetes Service (AKS).
________________________________________
🛣️ Data Flow and Key Components
The data flow within the Hyperion architecture is structured into distinct layers: Ingestion, Processing, Backend Storage, and Visualization/Egress.
1. ➡️ Ingestion Layer (OTel Collector Ingress)
This is the entry point for all telemetry data into Hyperion.
•	App Gateway (Hyperion): All client requests and data streams first enter through the custom-managed Application Gateway, which is used instead of the Megatron-provided one for greater customization (e.g., domain, TLS configuration).
•	NGINX Ingress (Megatron): The requests are then forwarded to the NGINX Ingress controller running on the Kubernetes cluster. The NGINX Ingress controller has defined ingress rules (endpoints) and routes the telemetry data to the OTel Collector Ingress.
•	OTel Collector Ingress: This component is the first stage of the OpenTelemetry processing. It receives all the telemetry data (logs, metrics, traces) and acts as a router. It filters the data (checking for allowed metrics) and forwards it to the Processing Layer.
•	Kafka (Failover/Cache): Kafka is deployed to act as a data-loss protection mechanism. If any of the downstream processing pipelines fail, the events are sent to Kafka, where they are temporarily stored and later pulled back to the OTel Collectors.
2. ⚙️ Processing Layer
This layer is responsible for transforming and filtering the raw telemetry data before storage.
The OTel Collector Ingress routes data to specific processors:
•	OTel Collector Metrics Processor: Handles metric data.
•	OTel Collector Logs Processor: Handles log data.
•	OTel Collector Trace Processor: Handles trace data. For traces, there is an additional layer with the OTel Collector Loadbalancer to stabilize the processing, especially when dealing with high-volume telemetry (e.g., 400k events/second).
3. 💾 Backend Storage Layer
This layer stores the processed telemetry data for a defined retention period.
•	Traces:
o	Grafana Tempo (Trace Backend): The trace data storage and indexing system.
o	Azure Blob Storage (Data Storage): Used by Tempo for persistent, long-term trace data storage.
o	ClickHouse (Analytics DB): A columnar database used to create materialized views for specific analysis and end-to-end monitoring of traces.
•	Metrics:
o	Grafana Mimir (Metrics Backend) / Victoria Metrics: The metrics storage system. The original plan to use Mimir was replaced by Victoria Metrics. This component uses Kubernetes PVCs (Persistent Volume Claims) for its primary storage (write and read operations) for a 14-day retention period.
o	Azure Blob Storage (Data Storage): Used as a backup option for Victoria Metrics in case of PVC deletion or unavailability.
•	Logs:
o	Grafana Loki (Logs Backend): The log aggregation system.
o	Azure Blob Storage (Data Storage): Used by Loki for persistent log storage.
4. 📊 Visualization and Egress Layer
This layer is the interface for users to view data and for forwarding data to external systems.
•	Grafana (WebUI): The main visualization tool. All stakeholders access Grafana to view dashboards and query the data stored in Tempo, Loki, and Victoria Metrics.
o	PostgreSQL (Grafana Config): A PostgreSQL database is used to store the state of the Grafana application, including dashboards, data source configurations, and alerts. This ensures persistence if the Grafana instance is deleted.
•	OTel Collector Egress: This component is responsible for downstream data forwarding to external consumers (like Checkpoint logging or the Poseidon brand) that need specific, filtered, or sampled Hyperion data. This process often involves sending data back to Kafka for the consuming system to retrieve.
________________________________________
🛠️ Megatron and Infrastructure
The entire Hyperion deployment runs on Azure with essential infrastructure services provided by Megatron, which is a high-level entity within Carrier responsible for the infrastructure for running all services.
•	AKS: Megatron provides the underlying Azure Kubernetes Service (AKS) clusters where Hyperion is deployed, including development (Dev) and production (Prod) environments.
•	Security and Policies: Megatron provides built-in security features, including Gatekeeper (kyverno) for checking YAML definitions against security policies before deployment.
•	Add-ons: Megatron also provides tools that Hyperion utilizes, such as ArgoCD for continuous deployment, Trivy for image scanning, and an Azure Ingress Firewall.


 -->
