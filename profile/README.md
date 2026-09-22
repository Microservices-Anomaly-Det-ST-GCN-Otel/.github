# Project: Machine Learning Based Microservices Failure Root Cause Detector
**Domain:** AIOps, Distributed Systems Observability, Machine Learning

## 1. Executive Summary
Modern enterprise applications rely on distributed microservices, where a single localized failure (e.g., a database timeout) instantly triggers a cascading domino effect across the network. Traditional monitoring tools rely on raw timestamps and static metric thresholds, resulting in "alert storms" that overwhelm engineers with hundreds of false-positive warnings while masking the true origin of the crash. Project AMAD introduces a topology-aware observability pipeline. By utilizing a Spatial-Temporal Graph Convolutional Network (ST-GCN) as an intelligent anomaly gatekeeper and a Backward Random Walk algorithm as a deterministic trace investigator, the system autonomously filters out symptom alerts and isolates the exact root-cause microservice within seconds.

---

## 2. End-to-End Logic & System Flow

### Phase 1: Distributed Sandbox & Telemetry Ingestion
The system operates on a polyglot microservice architecture designed to handle standard user traffic and simulate real-world failure scenarios (via Chaos Mesh). 
*   As microservices communicate, **OpenTelemetry** agents silently intercept network packets, attaching unique **Trace IDs** and capturing health metrics (latency, HTTP codes) without altering business logic.
*   This telemetry data is continuously streamed into **Redis Streams**, acting as an ultra-fast, rolling in-memory buffer that retains only the latest time-window of data to prevent storage bloat.

### Phase 2: Topology-Aware Gatekeeping (Anomaly Detection)
A backend service continuously pulls the telemetry batches from Redis and feeds them into the anomaly detection engine.
*   The system evaluates the data using an offline-trained **Spatial-Temporal Graph Convolutional Network (ST-GCN)**. 
*   Instead of evaluating metrics in isolation (which causes false alarms for harmless CPU spikes), the ST-GCN analyzes the physical map of the system (Spatial) and the flow of traffic (Temporal). It recognizes the mathematical "ripple effect" of a true cascade failure.
*   When a structural failure is confirmed, the ST-GCN acts as a gatekeeper, triggering a system-wide alert to initiate the root-cause investigation.

### Phase 3: Root-Cause Isolation (The Detective)
Once the ST-GCN triggers the alarm, the system hands the entire broken network graph over to the **Backward Random Walk** algorithm.
*   To solve the "Stopwatch Problem" (where unsynchronized cloud server clocks make timestamp-based sorting inaccurate), this algorithm ignores timestamps entirely. 
*   It utilizes the OpenTelemetry Trace IDs as a continuous digital thread, mathematically crawling backward across the API connections—starting from the screaming user-facing symptom node and walking back until it hits the silent root cause.
*   The system packages the isolated faulty node, the propagation path, and the exact telemetry evidence into an Incident Snapshot and commits it to **MongoDB** for historical persistence.

### Phase 4: Observability Console & Engineer Triage
The final incident snapshot is pushed instantly via persistent WebSockets to a real-time observability dashboard.
*   The UI dynamically renders the microservice network topology. Downstream symptom nodes display warning colors, while the true root-cause service pulses in bright red.
*   Engineers can click the root-cause node to view the irrefutable evidence—the exact OpenTelemetry Trace IDs and raw error logs—allowing for immediate, targeted code remediation.

---

## 3. Technology Stack & Justification

### Infrastructure & Deployment
*   **Docker & AWS EC2/EKS:** Containerization ensures microservices remain isolated. Deploying on AWS introduces real-world network latency and proves the system's viability in true distributed cloud environments.

### Backend & Microservices
*   **NestJS (TypeScript):** Provides a highly structured, scalable framework for building the simulated operational microservices, perfectly suited for rapid API generation and local Chaos Mesh testing.
*   **Python (FastAPI):** Chosen for the AI bridge layer due to Python's unrivaled ecosystem for machine learning and graph mathematics, while FastAPI ensures high-throughput asynchronous data processing.

### Telemetry & State Management
*   **OpenTelemetry:** The industry standard for distributed tracing. It guarantees vendor-agnostic, standardized Trace IDs capable of traversing polyglot architectures.
*   **Redis Streams:** Crucial for high-speed log ingestion. Traditional databases would lock or crash under the write-heavy load of a telemetry flood; Redis operates entirely in RAM as a transient buffer.
*   **MongoDB:** A flexible NoSQL document store ideal for saving unstructured JSON incident snapshots and hierarchical graph data for post-mortem UI retrieval.

### Machine Learning & Graph Processing
*   **PyTorch Geometric:** Specifically designed for deep learning on irregular structures like graphs. It provides the native mathematical primitives required to build and train the ST-GCN efficiently.
*   **NetworkX:** Utilized within Python to structure the in-memory directed graphs and execute the Personalized PageRank matrix math required for the Backward Random Walk.

### Frontend Observability
*   **Next.js (React) & WebSockets:** Next.js provides a robust SSR foundation, while WebSockets replace traditional HTTP polling, ensuring the UI reacts to backend anomalies in real-time (sub-millisecond updates).
*   **D3.js / Vis.js:** Standard charting libraries cannot visualize complex node-edge relationships. Force-directed graph libraries are required to render dynamic, interactive system topologies.
