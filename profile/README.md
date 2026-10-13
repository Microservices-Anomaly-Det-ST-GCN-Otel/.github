# Aegis-RCA: A Clock-Immune Spatial-Temporal Graph Architecture for Real-Time Microservice Root-Cause Analysis

**Aegis-RCA** is an advanced AIOps observability and root-cause analysis (RCA) framework designed for cloud-native microservices deployed on Kubernetes and AWS EKS. 

By decoupling fast anomaly detection from computationally intensive root-cause isolation, Aegis-RCA solves two major flaws of traditional observability tools: **server clock drift** (NTP skew across nodes) and **unmonitored dependency blindness** (third-party APIs or managed cloud databases).

---

## 1. Executive Summary

Modern cloud-native applications rely on containerized microservices where a single localized failure (e.g., CPU throttling, memory leaks, or packet loss) quickly triggers cascading failure storms across inter-service call graphs. Traditional monitoring tools rely on physical host timestamps and static metric thresholds, causing "alert storms" that overwhelm on-call engineers while masking the true origin of the outage.

**Aegis-RCA** introduces a topology-aware, two-tier observability pipeline:
1. **Tier 1 (Gatekeeper):** A Spatial-Temporal Graph Convolutional Network (**ST-GCN**) acts as an offline-trained, zero-shot anomaly detector, continually monitoring graph matrix variations with ultra-low computational overhead.
2. **Tier 2 (The Investigator):** Upon anomaly confirmation, a **Personalized PageRank** trace walk algorithm traverses parent-child execution headers using OpenTelemetry Trace Span IDs. By ignoring server clocks entirely, it mathematically crawls backward across inter-service calls to isolate the true root cause within seconds.

---

## 2. System Architecture & End-to-End Workflow

### Phase 1: Testbed Setup & Telemetry Ingestion
* **Benchmark Testbed:** Deployed on Kubernetes (AWS EKS) using industry-standard microservice benchmark applications (**Sock Shop** and **Train-Ticket**).
* **Load & Fault Simulation:** **Locust** generates continuous virtual user workloads across realistic browsing and checkout flows, while **Chaos Mesh** injects controlled synthetic faults (CPU throttling, memory leaks, packet loss, network latency, and inter-node server clock skew).
* **Un-sampled Ingestion:** OpenTelemetry agents capture application metrics and distributed trace spans containing parent-child execution headers, streaming them directly into an in-memory **Redis Streams** buffer.

### Phase 2: Tier 1 — Topology-Aware Gatekeeping (ST-GCN)
* An ingestion worker cleans raw telemetry streams (handling pod restart metric gaps and network retry duplicate spans), standardizes scales via Z-score normalization, and feeds PyTorch tensor matrices into the engine.
* The **Spatial-Temporal Graph Convolutional Network (ST-GCN)** evaluates structural spatial relationships and temporal traffic flows simultaneously.
* It operates as a zero-shot model, detecting systemic anomaly patterns without requiring local retraining or fine-tuning on new cluster topologies.

### Phase 3: Tier 2 — Clock-Immune Root-Cause Isolation
* Once Tier 1 flags a structural failure, the entire execution graph passes to the **Personalized PageRank** traversal module.
* To eliminate the **Stopwatch Problem** (where unsynchronized server clocks corrupt causality sorting), the algorithm ignores physical timestamps entirely.
* It uses **OpenTelemetry Trace Span IDs** as a digital thread, walking backward along parent-child execution headers to isolate the root-cause microservice—even across unmonitored external dependencies.
* Immutable **Incident Snapshots** (metrics, trace graphs, and error states) are captured for historical auditing.

### Phase 4: Observability Console, Snapshotting & Incident Triage
* Results are delivered instantly to an interactive **Next.js** observability dashboard via persistent WebSockets.
* **D3.js / Vis.js** dynamically renders the live microservice topology graph, highlighting root-cause nodes with visual red indicators.
* On-call engineers receive immediate automated **Email Notifications** with direct links to incident snapshots for rapid code-level remediation.

---

## 3. Tech Stack & Infrastructure

| Component | Technology | Rationale / Description |
| :--- | :--- | :--- |
| **Cloud & Orchestration** | Kubernetes / AWS EKS | Production-grade container orchestration for distributed microservice deployments. |
| **Fault Engineering** | Chaos Mesh | Synthetic fault injection (CPU, memory, packet loss, clock skew) for abnormal dataset generation. |
| **User Load Testing** | Locust | High-concurrency virtual user traffic generator for realistic cluster load profiling. |
| **Telemetry Pipeline** | OpenTelemetry & Redis Streams | Vendor-agnostic distributed tracing paired with an in-memory RAM buffer for high-throughput ingestion. |
| **Core AI / ML Engine** | PyTorch Geometric & NetworkX | Tensor graph processing for ST-GCN anomaly detection and matrix math for Personalized PageRank walks. |
| **Frontend Observability** | Next.js, WebSockets, D3.js/Vis.js | Real-time SSR dashboard rendering interactive dependency graphs, snapshots, and live alerts. |

---

## 4. Evaluation Metrics

Aegis-RCA is evaluated against standard baseline RCA approaches using three primary benchmarks:
1. **Top-K Accuracy ($A@1, A@3, A@5$):** Evaluates whether the true root-cause microservice is located within the top $K$ predicted candidates.
2. **Mean Reciprocal Rank (MRR):** Measures the average statistical rank position of the actual root cause across evaluation runs.
3. **Computational Overhead:** Profiles CPU usage, memory footprint, and inference latency added by the telemetry ingestion and model evaluation pipelines.

---

## 5. High-Level Project Schedule & Milestones

The project is structured across a 32-week academic timeline:

| Phase | Duration | Focus Area | Assigned Resources | Key Deliverables |
| :--- | :--- | :--- | :--- | :--- |
| **A1** | Weeks 1–4 | Literature Review & Requirements | Full Team | Approved FYP Proposal & Technical Specification Document |
| **A2** | Weeks 5–8 | Testbed & Dataset Creation | DevOps & Data Lead | Sock Shop / Train-Ticket cluster, Locust scripts, Chaos Mesh dataset |
| **A3** | Weeks 9–12 | Telemetry Pipeline & Redis | Backend Engineer | OpenTelemetry-to-Redis streaming worker & PyTorch graph preprocessor |
| **A4** | Weeks 13–18 | Tier 1: ST-GCN Model Training | AI / ML Engineer | Offline pre-trained ST-GCN gatekeeper with zero-shot capabilities |
| **A5** | Weeks 19–22 | Tier 2: PageRank Trace Engine | Algorithmic Lead | Clock-immune parent-child trace walk module for root-cause isolation |
| **A6** | Weeks 23–26 | Next.js Dashboard & AWS Cloud | Frontend Lead & DevOps | Interactive D3.js topology UI, incident snapshotting, AWS EKS launch |
| **A7** | Weeks 27–29 | System Integration & Benchmarking | Full Team | Quantitative evaluation report ($A@K$, MRR, CPU/memory overhead) |
| **A8** | Weeks 30–32 | Final Thesis & Oral Defense | Full Team | Complete Thesis Report, GitHub Code Handover, and Final Defense |

> *Note: The complete `.mpp` schedule file, dependencies, and Work Breakdown Structure (WBS) are managed independently in Microsoft Project.*

---

## 6. Team & Responsibilities

* **Abdul Wahab** — *Project Manager / AI & Algorithmic Lead*
  * Responsible for Tier 1 ST-GCN model development, Tier 2 Personalized PageRank trace walk logic, overall research methodology, and MS Project scheduling.
* **Hassan Ali** — *DevOps, Backend & Frontend Lead*
  * Responsible for Kubernetes/AWS EKS cluster setup, Chaos Mesh/Locust datasets, OpenTelemetry/Redis ingestion pipelines, and Next.js/D3.js dashboard development.

---

## 7. License

This project is developed for undergraduate thesis research at the Department of Computer Science. All rights reserved.
