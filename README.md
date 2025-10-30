<!-- Megatron: The Cloud Foundation
Megatron is the underlying platform that provides the foundational infrastructure and a suite of tools for deploying and managing services within Carrier. It essentially serves as a managed Platform-as-a-Service (PaaS) layer, primarily leveraging Azure and Kubernetes.
Key Megatron Components
Category	Component	Purpose
Source & Pipelines	GitHub	Source code repository and hosting for GitHub Actions (used for CI/CD, deployment pipelines).
Orchestration	Kubernetes	Uses Azure Kubernetes Service (AKS) to manage containerized applications, including multiple clusters (Dev, QA, Prod).
Security & Policy	GitOps with ArgoCD	Enables continuous deployment by automatically syncing the desired state from Git to the Kubernetes clusters.
	Security built-in, Central Policy management	Provides a security framework with tools like Giverno (for checking Kubernetes YAML definitions against security compliance) and Trivy (for image scanning).
Azure Services	Load Balancer, App Gateway, Egress Firewall, Virtual Networks, DDoS Protection, DNS	Standard managed cloud infrastructure components to ensure network security, traffic routing, and domain name resolution.
________________________________________
Hyperion: The Observability Architecture
Hyperion is a dedicated observability platform built on top of Megatron's infrastructure. Its primary goal is to collect, process, and visualize telemetry data (logs, metrics, and traces) to provide deep insights into application performance and state.
The LGTM Stack and Data Flow
Hyperion is centered around the LGTM stack (Loki, Grafana, Tempo, and Mimir—though Mimir is replaced by Victoria Metrics), with OpenTelemetry Collector as the central data pipeline.
1.	Ingestion (Entry Points):
o	Application Gateway (Hyperion's Own): The custom entry point for external requests, handling routing and security before forwarding to Kubernetes.
o	NGINX Ingress: Handles routing of incoming requests within the Kubernetes cluster.
2.	Collection & Processing:
o	OpenTelemetry (OTel) Collector: The critical component that receives all telemetry data (metrics, logs, traces). It operates in a multi-layered pipeline:
	Ingress Collector: Filters and routes incoming telemetry data.
	Processing Layer (Metrics/Logs/Trace Processor): Transforms and processes the data before storage.
	Egress Collector: Used for downstream data forwarding to consumer teams (like Checkpoint Logging or Porsche) after filtering and sampling.
o	Kafka: Acts as a failover buffer to prevent data loss. If any collector pipeline fails, the events are sent to Kafka and can be pulled back later for reprocessing.
3.	Storage (Backend Layer):
o	Grafana Tempo (for Traces): Stores trace data, primarily in Azure Blob Storage for long-term persistence (40 days retention).
o	Grafana Loki (for Logs): Stores log data, primarily in Azure Blob Storage (7 days retention).
o	Victoria Metrics (replaces Grafana Mimir for Metrics): Stores metrics data. Unlike Tempo and Loki, it stores its data primarily on Kubernetes PVCs (Persistent Volume Claims) for 14 days, with a separate Azure Storage backup for disaster recovery.
o	ClickHouse: An analytical database currently being tested/developed for use with traces to enable end-to-end monitoring and create materialized views.
4.	Visualization and State:
o	Grafana: The unified visualization layer where all stakeholders view dashboards, alerts, and operational data from Tempo, Loki, and Victoria Metrics.
o	PostgreSQL: Used as a persistent database to store Grafana's internal state (dashboards, data source configurations, alerts) so they aren't lost if Grafana itself is deleted or restarted.
Key Concepts
•	Observability Pillars: The architecture collects the three pillars of observability: Logs (low emphasis), Metrics, and Traces (high emphasis).
•	Security Tools: ArgoCD for GitOps, Giverno for compliance checking, Trivy for image scanning, and Falco for runtime security and networking controls.
•	Persistent Storage: Azure Blob Storage is utilized for logs and traces, while Kubernetes PVCs are primarily used for metrics (Victoria Metrics), backed up by Azure Storage.

 -->
