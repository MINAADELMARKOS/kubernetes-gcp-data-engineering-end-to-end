## Architecture Overview

The high‑level architecture of this end‑to‑end data engineering solution combines infrastructure provisioning, container orchestration and data‑pipeline logic on Google Cloud Platform.  The design prioritises scalability, resilience and compliance with European data‑protection regulations.

### Components

| Component | Description |
| --- | --- |
| **Terraform** | Defines and provisions all cloud resources: GKE cluster, Pub/Sub topic & subscription, BigQuery dataset/table, IAM roles and networking. |
| **Google Kubernetes Engine (GKE)** | Hosts the Python microservice that consumes messages from Pub/Sub, pseudonymises sensitive fields and writes to BigQuery.  Each pod runs a container built from the `data_pipeline.py` service. |
| **Pub/Sub** | Serves as the ingestion layer for raw transaction events.  Messages are published by the upstream application and consumed by the worker service. |
| **BigQuery** | Stores cleansed and pseudonymised data.  Tables are partitioned by ingestion date to improve query performance and support retention policies. |
| **Secret Manager** | Securely stores service‑account credentials, database URIs and other secrets. |
| **Cloud Logging & Monitoring** | Provides observability into pipeline health, throughput and errors. Logs are audited to satisfy GDPR accountability requirements. |

### Data Flow

1. **Event Publishing:** The retail application publishes a JSON event (containing user ID, transaction amount, timestamp and item category) to the Pub/Sub topic.
2. **Message Consumption:** The Python microservice running on GKE subscribes to the topic and pulls messages in real time.
3. **Pseudonymisation & Processing:** The service applies a SHA‑256 hash to the user ID, removes unnecessary fields (data minimisation) and adds an ingestion timestamp. Additional transformations (e.g., currency conversion) can be applied here.
4. **Storage:** The processed record is written to the BigQuery table.  Optionally, the raw message is archived to Cloud Storage for auditing.
5. **Monitoring:** Cloud Logging records processing status, errors and throughput metrics.  Alerts can be configured on failure rates or latency.

### Compliance Notes

* **Data minimisation:** Only required fields are persisted.  Identifiers are hashed to prevent re‑identification.
* **Retention:** BigQuery’s table partitioning and GCS lifecycle rules enforce automatic deletion of data after a defined period.
* **Security:** All data is encrypted at rest and in transit.  Service accounts are granted minimal roles (principle of least privilege).  Secrets are not stored in code.
* **Transparency & accountability:** The README and this document describe data handling practices clearly.  Logging and audit trails support investigations or GDPR subject‑access requests.