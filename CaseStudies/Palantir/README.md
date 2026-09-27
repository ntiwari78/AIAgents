
# Main Resource
- https://www.palantir.com/docs/foundry/architecture-center/aip-architecture
- https://www.palantir.com/docs/foundry/architecture-center/ontology-system/

---
---

![Palantir-aip](https://github.com/ntiwari78/AIAgents/blob/main/images/Palantir-aip.jpeg)

---
---

# Palantir AIP Architecture Layers

Palantir’s **Artificial Intelligence Platform (AIP)** sits atop a multi-layered architecture that tightly integrates enterprise data, operational logic, and AI models. At the heart of this stack is the **Ontology system**, which unifies data (“nouns”), processes (“verbs”), and security policies into a single semantic graph. On top of the Ontology are Foundry/Gotham applications and user interfaces, while below it live data pipelines and compute services. AIP then connects third-party LLMs and AI agents into this mix, enabling AI-driven workflows that respect all security and governance guardrails. The architecture can be described in roughly four layers:

## 1. Semantic Data Layer (Ontology & Data Services)
This layer is the **single source of truth** for the enterprise. Palantir’s Ontology system continuously ingests and organizes data from ERP, CRM, IoT sensors, documents, and other sources into a coherent graph of objects and relationships. The Ontology does **four-fold integration**: data, logic (business rules/ML models), actions (workflows, LLM-powered functions), and security, making it a true “digital twin” of the business. All data is stored in open formats (e.g. Apache Iceberg tables) and remains queryable via SQL, Python, or REST interfaces. This ensures that analytics and AI have real-time access to the freshest enterprise context.

- **Key components:** Ontology Manager, data services (ingestion, transformation, storage), virtual/physical tables (Iceberg, Parquet), change-data-capture pipelines.  
- **Best practices:** Model your ontology around real business concepts and processes, not just raw data. Define clear object types (“nouns”) and actions (“verbs”) and link them with security markings to enforce policy. Use Palantir’s tooling to capture data lineage and enforce mandatory/discretionary access labels at creation time. Whenever possible, keep data in open standards (Iceberg, CSV, S3) and avoid duplication by using virtual tables to connect existing data warehouses. Thoroughly test new data pipelines (e.g. with AI FDE) on sandbox branches and require code reviews before merging, as automated pipeline generation should be verified for correctness and performance.

## 2. Data Integration & Compute Layer (Multimodal Data Plane)
Underpinning the Ontology is the **Multimodal Data Plane (MMDP)** – an open, scalable data and compute fabric. Any type of enterprise data (tabular, streaming, geospatial, media, documents) can be ingested. Palantir leverages Apache Iceberg as the primary table format (compatible with Snowflake, Databricks, etc.), so Foundry/AIP can mix and match live data sources without copying them. On the compute side, a hardened Kubernetes mesh called **Rubix** provides all runtimes. MMDP supports **batch (Spark)**, **streaming (Flink)**, and lightweight engines (DuckDB, Polars, DataFusion) out of the box. Clients can also “bring your own” compute via Containerized **Compute Modules**, letting you plug in custom LLMs or specialized runtimes.

- **Key components:** Data connectors (change-data-capture, APIs, file ingestion), Iceberg catalogs (virtual or managed), compute engines (Spark, Flink, DuckDB, etc.), Rubix container cluster.  
- **Best practices:** Use the open data features: for example, register external data with virtual tables instead of copying, and configure pipelines with source-based transforms to minimize data movement. Design pipelines to be idempotent and test with representative data, since Palantir’s continuous deployment (via Apollo) will roll out changes across environments. In high-scale scenarios, partition data effectively (e.g. via Iceberg partitioning) and offload compute where possible (e.g. pushdown SQL to partner data engines). Because Rubix nodes are ephemeral (cycled regularly for security), ensure your jobs can tolerate worker restarts by writing fault-tolerant code and using retry mechanisms.

## 3. AI / Model Layer (LLM & Agent Orchestration)
Sitting atop the data stack is Palantir’s **AI Platform (AIP)**, which provides secure connectivity to large language models and agent frameworks. AIP’s “secure LLM integration” tunnels queries to commercial LLMs (GPT, Claude, Gemini, Grok, etc.) or on-prem models, **guaranteeing no retention or re-training** of your data by the provider. It also allows “bring your own model” (BYOM) via REST or container interfaces. Once models are accessible, AIP supplies the context: it feeds the Ontology data as retrieval-augmented prompts so agents can ground their reasoning in real-time enterprise facts. 

Behind the scenes, AIP implements **agent orchestration**: you can build workflows that mix LLM calls with code, actions on the Ontology (via Functions on Objects), and external API calls. Each agent run is fully logged and auditable. AIP also includes **AIP Evals**, a built-in test framework, to validate agent outputs against expected results and track performance over time.

- **Key components:** LLM connectors, AIP Logic (low-code workflow editor), Functions on Objects, “AIP Chatbots” (for chat interfaces), AIP Evals (evaluation), and security middleware.  
- **Best practices:** Keep AI interactions **focused**. Only pass to the model the relevant data and tools needed – too much context or too many functions can confuse the model. Rigorously **evaluate** any LLM output: use Palantir’s Evals framework to create test suites for your agents so you can compare model variants and catch regressions. Do **prompt engineering**: write clear, specific instructions and include examples where needed. Palantir’s docs advise iterative prompting (clarity, specificity, relevancy) to get the desired accuracy. Finally, remember that automated agents can strain infrastructure; monitor usage and employ rate limits or quotas as needed.

## 4. Developer & Application Layer (Foundry, Gotham, Workspaces)
Above the data and AI engines are the **user-facing applications and developer tools**. Palantir Foundry/Gotham provide rich UIs (liveboards, dashboards, code notebooks, etc.) that let analysts and operators interact with the Ontology and AI services. Developers use **Workshop (code repositories)**, the **Ontology SDK** and **Platform SDK**, or even Palantir’s VS Code plugin to build pipelines, apps, and integrations. AIP’s building blocks (AIP Chatbot Studio, AIP Document Intelligence, AIP Analyst, etc.) appear as Workshop modules or widgets in apps. These tools all run on top of the Ontology, meaning a BI visualization or a mobile app can call the same Ontology objects and permissions as an AI agent.

- **Key components:** Foundry/Gotham modules (Analytics, Operations, Crisis Management, etc.), Code Workspaces (Jupyter/RStudio), AIP widgets (chatbot/agent UI), global branching for projects.  
- **Best practices:** Build applications by composing predefined Ontology objects and logic, not by duplicating data models. Use standard DevOps workflows: package pipelines and apps in branches and promote through testing before release. For interactive apps, use the **AIP Chatbot widget** to embed AI assistants directly in Foundry apps – it can read and write to application state so AI and UI stay in sync. Keep analytics separate from operational apps: use dashboards (Liveboards) for reporting, and use Gateway APIs for any external front-end. Encourage analysts to work in code repos with version control enabled, so that any code (e.g. functions or notebooks) can be peer-reviewed and tested before deployment.

## 5. Deployment & Infrastructure Layer (Apollo, Rubix, Cloud)
Underpinning everything is a **zero-trust, microservices infrastructure** managed by Palantir’s **Apollo** platform and Rubix Kubernetes substrate. Apollo automates blue/green deployments and configures autoscaling for hundreds of services (Foundry, AIP, Gotham) so upgrades happen **day and night with zero downtime**. Rubix enforces strict security at the container level (auto-cycling nodes every ~48 hours, network isolation, pervasive encryption). The net result: Foundry/AIP run as a unified enterprise OS that can live on any cloud or on-prem (AWS, Azure, GCP, Oracle, IL-5/6 networks, etc.).

- **Key components:** Kubernetes cluster (Rubix), deployment control plane (Apollo), storage layers, and networking.  
- **Best practices:** Design services to be stateless and restartable (since Rubix kills nodes regularly for security). Use Apollo’s capabilities: define “plans” for each release, use blue/green deployments for major upgrades, and leverage out-of-the-box autoscaling policies. Monitor health via Palantir’s observability (see below) so that autoscaling can respond to demand. For *edge* or *disconnected* deployments, use Apollo’s federated mode and ensure connectivity rules are compliant. Lastly, if you bring your own containers (via Compute Modules), ensure they are small, stateless, and follow the same security best practices (open ports, secrets, etc.) as Palantir’s containers – Rubix treats them the same way.

## 6. Security & Governance (Cross-Cutting)
Security is woven through every layer. Palantir enforces **zero trust** at the infrastructure level and **object-level access control** at the data/model level. By default every data object and workflow action carries mandatory security markings and lineage. Users and agents only see the fields and functions they’re permitted to use. All data at rest and in transit is encrypted with modern cryptography, and multi-factor authentication and audit logging are mandatory features of the platform. 

- **Key components:** Role-, purpose-, and marking-based access controls, project/organization isolation, audit trail services, and integration with enterprise identity (SAML/AD).  
- **Best practices:** Adopt the least-privilege model: assign the minimal roles needed for each user or agent, and use Palantir’s *Markings* and *Classification-Based Access Controls* to protect sensitive columns or rows. Regularly review audit logs and lineage views to ensure no unexpected data flow occurs. Use Workflows or Code Workspaces to implement checks on sensitive actions (for example, require a review for any Ontology schema change). In short, plan your security scheme in advance: organize data into projects and organizations by trust boundary, tag data with appropriate sensitivity markings, and leverage Palantir’s built-in encryption and governance rather than trying to bolt them on later.

## 7. Observability & Monitoring
Robust observability ensures the stack runs smoothly. Every service emits metrics and logs to Palantir’s monitoring systems. Within AIP, **end-to-end observability** is provided for all AI workflows: you can trace every data flow into the Ontology, every prompt sent to an LLM, and every action taken by an agent or user. Token usage and compute costs are logged for budgeting. The integrated logging and audit features mean you can reconstruct any step of a workflow after the fact. 

- **Best practices:** Enable **AIP Evals** and **Metrics Dashboards** to catch model drift or performance issues early. Use Foundry’s integrated checkpointing and lineage diagrams to validate data pipelines. Set up alerts on infrastructure metrics (CPU, memory, queue lengths) via Apollo. In short, instrument everything: logs, events, costs – and review them regularly to spot anomalies.

## 8. Interoperability & Integration
Palantir is designed to play well with other systems. All layers use **open standards** and APIs. Data is accessible via REST, JDBC/ODBC, and S3, so BI tools like Tableau or Power BI can plug in with zero data copying. The Ontology is exposed via JSON/REST and even Palantir’s Model Context Protocol (MCP) for agent-driven sync. Developers can call AIP or Ontology functions via SDKs (Python, TypeScript) or use “Bring Your Own Container” for custom connectors. 

- **Key components:** Virtual tables (e.g. Databricks, BigQuery), BI connectors (JDBC to Liveviews), Code Workspaces for Jupyter/RStudio, and Palantir’s Compute Modules for custom services.  
- **Best practices:** When possible, **reuse existing enterprise assets**: use virtual tables to federate data, register LLMs via API hooks, and integrate with identity providers (SAML) so users get single sign-on. Use Palantir’s platform SDK to script interactions (e.g. ingest via REST APIs) rather than writing proprietary integrations. For reporting, use the native SQL/BI connectors so you aren’t moving data out of Ontology.

## References

Palantir’s official documentation provides in-depth details on each layer and component. In particular, see the [AIP Architecture Overview](https://www.palantir.com/docs/foundry/architecture-center/aip-architecture), the [Integrated Platforms overview](https://www.palantir.com/docs/foundry/architecture-center/platforms), the [Ontology system](https://www.palantir.com/docs/foundry/architecture-center/ontology-system), the [Multimodal Data Plane](https://www.palantir.com/docs/foundry/architecture-center/multimodal-data-plane), and the [Security and Governance overview](https://www.palantir.com/docs/foundry/security/overview) for authoritative guidance. These explain how the layers connect and offer further best-practice guidance for building secure, scalable AI-driven systems on Palantir’s platform.

---
---

You're right. The previous output did not surface the reference links. I checked Palantir's current documentation and mapped the **12 architecture categories in the diagram** to the most relevant official documentation pages. The Palantir architecture page itself confirms that these are the 12 general capability categories of AIP. ([Palantir][1])

Below is a link set you can use directly in the Markdown report.

# Palantir AIP Architecture — Authoritative Reference Links

## 0. Master architecture references

These are the best starting points because they explain how the whole architecture fits together.

| Topic                        | Authoritative reference                                                                                                                        | What it covers                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **AIP Architecture**         | [Palantir AIP Architecture Overview](https://www.palantir.com/docs/foundry/architecture-center/aip-architecture?utm_source=chatgpt.com)        | The 12 architecture categories shown in your diagram                              |
| **Architecture Center**      | [Palantir Architecture Center](https://www.palantir.com/docs/foundry/architecture-center/overview?utm_source=chatgpt.com)                      | Relationship between AIP, Foundry, Apollo, Ontology, data/logic/workflow services |
| **Foundry Documentation**    | [Palantir Foundry Documentation](https://www.palantir.com/docs/foundry?utm_source=chatgpt.com)                                                 | Complete official documentation index                                             |
| **Foundry Platform Summary** | [Foundry Platform Summary for LLMs](https://www.palantir.com/docs/foundry/getting-started/foundry-platform-summary-llm?utm_source=chatgpt.com) | Concise architecture description and terminology                                  |

The current Palantir documentation explicitly describes AIP as having **12 key capability categories**, matching the architecture diagram: secure LLM integration, observability, context engineering, Ontology, vector/compute/tool services, security/governance, agent lifecycle, operational automation, development environments, human+AI applications, package/release/deploy, and enterprise automation. ([Palantir][2])

---

# 1. Secure LLM Integration & Access

This corresponds to the bottom-right portion of your diagram:

> **Secure LLM Integration, Hosting, Access**

including:

* Commercial LLMs
* Open-source LLMs
* Private/custom models
* Model provider integration
* Infrastructure
* Smart caching
* Dynamic retry
* PII obfuscation
* Content detection/moderation
* Validation & oversight
* Model enablement
* Usage tracking
* Rate limiting

### Official Palantir references

**AIP Architecture — Secure LLM Integration**

[AIP Architecture](https://www.palantir.com/docs/foundry/architecture-center/aip-architecture?utm_source=chatgpt.com)

**Model Catalog**

[AIP Model Catalog](https://www.palantir.com/docs/foundry/model-catalog/overview?utm_source=chatgpt.com)

Model Catalog is specifically intended for discovering and selecting the LLMs available in AIP and provides sandbox/playground capabilities. ([Palantir][3])

**LLM integrations / AIP**

[Palantir AIP Documentation](https://www.palantir.com/docs/foundry/aip?utm_source=chatgpt.com)

**Foundry platform summary**

[LLM and AIP Platform Summary](https://www.palantir.com/docs/foundry/getting-started/foundry-platform-summary-llm?utm_source=chatgpt.com)

The current documentation also describes provider-compatible APIs for providers such as OpenAI and Anthropic, allowing external development tools to route requests through Foundry infrastructure while obtaining platform-level controls such as rate limiting, usage tracking and zero-data-retention capabilities. ([Palantir][2])

### Useful external references

**NIST AI Risk Management Framework**

[NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework?utm_source=chatgpt.com)

**OWASP LLM Top 10**

[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/?utm_source=chatgpt.com)

**NIST Generative AI Profile**

[NIST Generative AI Profile](https://www.nist.gov/itl/ai-risk-management-framework/ai-rmf-generative-ai-profile?utm_source=chatgpt.com)

These are particularly useful for the **validation, moderation, prompt injection, data leakage and model-risk** aspects of this layer.

---

# 2. End-to-End Observability

Your diagram shows:

> **End-to-End Observability**

and underneath it:

* Model Catalog
* execution monitoring
* telemetry
* workflow observability

### Official references

**Ontology & AIP Observability**

[Ontology and AIP Observability](https://www.palantir.com/docs/foundry/aip-observability/overview?utm_source=chatgpt.com)

This is probably the **single most important reference for Layer 2**.

Palantir documents metrics, execution history, distributed tracing, logging, token usage, prompts, error details and performance monitoring for AIP/Ontology workflows. ([Palantir][4])

**Foundry Observability**

[Foundry Observability](https://www.palantir.com/docs/foundry/observability/overview?utm_source=chatgpt.com)

This covers:

* metrics
* health checks
* logs
* traces
* alerts
* operational monitoring
* telemetry export

([Palantir][5])

### External references

**OpenTelemetry**

[OpenTelemetry](https://opentelemetry.io/docs/?utm_source=chatgpt.com)

**OpenTelemetry Traces**

[OpenTelemetry Tracing](https://opentelemetry.io/docs/concepts/signals/traces/?utm_source=chatgpt.com)

**OpenTelemetry Metrics**

[OpenTelemetry Metrics](https://opentelemetry.io/docs/concepts/signals/metrics/?utm_source=chatgpt.com)

**Google SRE Monitoring**

[Google SRE — Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/?utm_source=chatgpt.com)

---

# 3. Context Engineering

This is the top-left area of the diagram.

It contains:

### Contextual Data

* External Logic
* Model Building
* Functions
* Data/context integration

### Contextual Logic

### Systems of Action

* Event-driven
* Streaming
* Edge integration

Palantir explicitly describes context engineering as integrating **data, logic and action** into the Ontology through batch, streaming and CDC mechanisms. ([Palantir][1])

### Official references

**AIP Architecture — Context Engineering**

[Context Engineering in AIP Architecture](https://www.palantir.com/docs/foundry/architecture-center/aip-architecture?utm_source=chatgpt.com)

**Data Integration**

[Foundry Data Integration](https://www.palantir.com/docs/foundry/data-integration/overview?utm_source=chatgpt.com)

**Data Pipelines**

[What is a Data Pipeline?](https://www.palantir.com/docs/foundry/data-integration/data-pipeline?utm_source=chatgpt.com)

**Streaming**

[Foundry Streaming](https://www.palantir.com/docs/foundry/data-integration/streaming?utm_source=chatgpt.com)

**Change Data Capture**

[Change Data Capture](https://www.palantir.com/docs/foundry/data-integration/change-data-capture/overview?utm_source=chatgpt.com)

**Functions**

[Foundry Functions](https://www.palantir.com/docs/foundry/functions/overview?utm_source=chatgpt.com)

**Transforms**

[Foundry Transforms](https://www.palantir.com/docs/foundry/transforms/overview?utm_source=chatgpt.com)

### External references

**Apache Kafka**

[Apache Kafka Documentation](https://kafka.apache.org/documentation/?utm_source=chatgpt.com)

**Apache Flink**

[Apache Flink Documentation](https://nightlies.apache.org/flink/flink-docs-stable/?utm_source=chatgpt.com)

**Change Data Capture — Debezium**

[Debezium Documentation](https://debezium.io/documentation/?utm_source=chatgpt.com)

---

# 4. Ontology

This is arguably the **central architectural layer**.

The diagram shows:

* Ontology Core
* Semantic
* Kinetic
* Dynamic
* Human + AI Decision Model
* Media & Vector Services
* Tool Services
* Data
* Logic
* Actions
* OSDK

Palantir describes the Ontology as integrating **data + logic + action + security** and representing the operational world of an enterprise. ([Palantir][6])

### Essential references

**Ontology Core Concepts**

[Ontology Core Concepts](https://www.palantir.com/docs/foundry/ontology/core-concepts?utm_source=chatgpt.com)

This explains:

* Object Types
* Properties
* Link Types
* Action Types
* Functions
* Interfaces
* Roles
* Object Views

([Palantir][7])

**Ontology System Architecture**

[The Ontology System](https://www.palantir.com/docs/foundry/architecture-center/ontology-system?utm_source=chatgpt.com)

This is especially important for your architecture paper because it explains the:

* Ontology Language
* Ontology Engine
* Ontology Toolchain
* Data
* Logic
* Actions
* Security

([Palantir][6])

**Why Ontology?**

[Why Create an Ontology?](https://www.palantir.com/docs/foundry/ontology/why-ontology?utm_source=chatgpt.com)

### Ontology development

[Object Types](https://www.palantir.com/docs/foundry/ontology/object-types/overview?utm_source=chatgpt.com)

[Link Types](https://www.palantir.com/docs/foundry/ontology/link-types/overview?utm_source=chatgpt.com)

[Action Types](https://www.palantir.com/docs/foundry/ontology/action-types/overview?utm_source=chatgpt.com)

[Functions](https://www.palantir.com/docs/foundry/functions/overview?utm_source=chatgpt.com)

[Ontology SDK](https://www.palantir.com/docs/foundry/ontology-sdk/overview?utm_source=chatgpt.com)

---

# 5. Vector, Compute & Tool Services

The diagram shows:

### Media & Vector Services

* Documents
* Images
* Videos
* Geospatial
* Audio
* Vector capabilities

### Multimodal Compute Services

* Interactive
* Batch
* Serverless
* Streaming
* BYO compute

### Tool Services

* Data
* Logic
* Actions
* OSDK

### Official references

**Compute Modules**

[Compute Modules](https://www.palantir.com/docs/foundry/compute-modules/overview?utm_source=chatgpt.com)

Compute Modules allow existing code and containers to run inside Foundry, including custom functions, APIs, models and real-time processing. ([Palantir][8])

**Media Sets**

[Media Sets](https://www.palantir.com/docs/foundry/data-integration/media-sets/overview?utm_source=chatgpt.com)

**Virtual Media**

[Virtual Media](https://www.palantir.com/docs/foundry/data-integration/virtual-media/overview?utm_source=chatgpt.com)

**Ontology Toolchain**

[Ontology SDK](https://www.palantir.com/docs/foundry/ontology-sdk/overview?utm_source=chatgpt.com)

---

# 6. Security & Governance

The diagram explicitly lists:

* Role-based controls
* Marking-based controls
* Purpose-based controls
* System-wide branching
* Approvals
* Checkpoints

This is a **cross-cutting layer**, rather than merely another box in the stack.

### Official references

**Foundry Security Overview**

[Palantir Security Overview](https://www.palantir.com/docs/foundry/security/overview?utm_source=chatgpt.com)

Palantir documents authentication, authorization, mandatory controls, discretionary controls, markings, organizations, resource-level roles and row/column policies. ([Palantir][9])

**Access Control Propagation**

[Access Control Propagation](https://www.palantir.com/docs/foundry/security/access-control-propagation?utm_source=chatgpt.com)

This is an especially useful reference for understanding how security policies propagate through data and derived artifacts. ([Palantir][10])

**AI Ethics & Governance**

[AIP Ethics and Governance](https://www.palantir.com/docs/foundry/aip/ethics-governance?utm_source=chatgpt.com)

**Approvals**

[Approvals](https://www.palantir.com/docs/foundry/approvals/overview?utm_source=chatgpt.com)

**Checkpoints**

[Checkpoints](https://www.palantir.com/docs/foundry/checkpoints/overview?utm_source=chatgpt.com)

The official application reference describes Approvals as managing requests/approvals and Checkpoints as governance mechanisms that can require justification during sensitive interactions. ([Palantir][11])

### External standards

**NIST Cybersecurity Framework**

[NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework?utm_source=chatgpt.com)

**NIST Zero Trust Architecture**

[NIST SP 800-207 — Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final?utm_source=chatgpt.com)

**OWASP ASVS**

[OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/?utm_source=chatgpt.com)

---

# 7. Agent Lifecycle

Your diagram contains:

* Agent Building
* Agent Orchestration
* Evaluation Suites

This is the **build → test → evaluate → deploy → monitor** lifecycle.

### Official references

**AIP Architecture**

[AIP Architecture — Agent Lifecycle](https://www.palantir.com/docs/foundry/architecture-center/aip-architecture?utm_source=chatgpt.com)

**AIP Chatbot Studio**

[AIP Chatbot Studio](https://www.palantir.com/docs/foundry/aip/chatbot-studio/overview?utm_source=chatgpt.com)

**AIP Logic**

[AIP Logic](https://www.palantir.com/docs/foundry/aip/logic/overview?utm_source=chatgpt.com)

**AIP Evals**

[AIP Evals](https://www.palantir.com/docs/foundry/aip-evals/overview?utm_source=chatgpt.com)

The current Palantir platform summary describes AIP Evals as supporting test cases, debugging, iteration and comparison of agent performance across models and executions. ([Palantir][2])

### Agent security

[AI FDE Security & Governance](https://www.palantir.com/docs/foundry/ai-fde/security-and-governance?utm_source=chatgpt.com)

This is particularly useful because it demonstrates Palantir's approach to agent permissions, approvals, branch-aware controls and auditability. ([Palantir][12])

---

# 8. Operational Automation

Your diagram shows three modes:

### Scheduled Automation

### Event-driven Automation

### API-driven Automation

### Official references

[Foundry Automations](https://www.palantir.com/docs/foundry/automate/overview?utm_source=chatgpt.com)

[AIP Automate](https://www.palantir.com/docs/foundry/aip/automate/overview?utm_source=chatgpt.com)

[Workflow Authoring](https://www.palantir.com/docs/foundry/workshop/workflows/overview?utm_source=chatgpt.com)

The architecture documentation describes Workflow Services as supporting interactive compute, event-driven automations, scheduled automations and both pro-code and low-code workflow authoring. ([Palantir][13])

---

# 9. Development Environments

The diagram shows:

* Integrated VS Code
* Integrated Jupyter
* Compute Modules
* MCP
* IDE extensions

### Official references

**Developer Toolchain**

[Palantir Developer Toolchain](https://www.palantir.com/docs/foundry/dev-toolchain/overview?utm_source=chatgpt.com)

**VS Code**

[VS Code Workspaces](https://www.palantir.com/docs/foundry/code-workspaces/overview?utm_source=chatgpt.com)

**Code Workspaces**

[Code Workspaces](https://www.palantir.com/docs/foundry/code-workspaces/overview?utm_source=chatgpt.com)

**Palantir MCP**

[Palantir MCP](https://www.palantir.com/docs/foundry/palantir-mcp/overview?utm_source=chatgpt.com)

This is now particularly important. Palantir documents MCP as allowing AI IDEs and agents to build, modify and review applications across data integration, Ontology configuration and application development. ([Palantir][14])

**Ontology MCP**

[Ontology MCP](https://www.palantir.com/docs/foundry/ontology-mcp/overview?utm_source=chatgpt.com)

### External MCP specification

[Model Context Protocol — Official Specification](https://modelcontextprotocol.io/specification/latest?utm_source=chatgpt.com)

This is the appropriate external reference when explaining MCP itself rather than Palantir's implementation.

---

# 10. Human + AI Applications

The diagram shows:

* No-code / low-code AIP applications
* Object-oriented analytics
* Real-time analytics
* Workflow management
* Resource management

### Official references

**Ontology-aware Applications**

[Ontology-aware Applications](https://www.palantir.com/docs/foundry/ontology/applications?utm_source=chatgpt.com)

Palantir identifies Object Views, Object Explorer, Quiver and Workshop as important Ontology-aware applications. ([Palantir][15])

**Workshop**

[Workshop](https://www.palantir.com/docs/foundry/workshop/overview?utm_source=chatgpt.com)

**Slate**

[Slate](https://www.palantir.com/docs/foundry/slate/overview?utm_source=chatgpt.com)

**Object Views**

[Object Views](https://www.palantir.com/docs/foundry/ontology/object-views/overview?utm_source=chatgpt.com)

**Quiver**

[Quiver](https://www.palantir.com/docs/foundry/quiver/overview?utm_source=chatgpt.com)

**OSDK Applications**

[Ontology SDK Applications](https://www.palantir.com/docs/foundry/ontology-sdk/overview?utm_source=chatgpt.com)

Palantir describes applications as consuming the data and object layers to support operational workflows, with Workshop, Slate, analytics tools and custom OSDK applications forming major application surfaces. ([Palantir][2])

---

# 11. Package, Release & Deploy

The diagram shows:

* Product packaging
* Dependency management
* Environment promotion
* Release channels

This is essentially the **DevOps / product-delivery layer**.

### Official references

**Foundry DevOps**

[Foundry DevOps](https://www.palantir.com/docs/foundry/devops/overview?utm_source=chatgpt.com)

**Projects**

[Foundry Projects](https://www.palantir.com/docs/foundry/foundry-projects/overview?utm_source=chatgpt.com)

**Global Branching**

[Global Branching](https://www.palantir.com/docs/foundry/building-pipelines/global-branching/overview?utm_source=chatgpt.com)

**Release Management**

[Foundry Release Management](https://www.palantir.com/docs/foundry/release-management/overview?utm_source=chatgpt.com)

**Marketplace**

[Foundry Marketplace](https://www.palantir.com/docs/foundry/marketplace/overview?utm_source=chatgpt.com)

Palantir's architecture documentation specifically describes packaging data pipelines, Ontology definitions, automations and applications and promoting them across heterogeneous target environments. ([Palantir][1])

### External DevOps references

**DORA**

[DORA Research](https://dora.dev/?utm_source=chatgpt.com)

**CNCF**

[Cloud Native Computing Foundation](https://www.cncf.io/?utm_source=chatgpt.com)

**Kubernetes**

[Kubernetes Documentation](https://kubernetes.io/docs/?utm_source=chatgpt.com)

---

# 12. Enterprise Automation

The final layer shows:

* AI FDE
* AIP Analyst
* AIP-based development
* Code assistance
* AI-enabled APIs
* Enterprise-wide autonomous workflows

### Official references

**AI FDE**

[Palantir AI FDE](https://www.palantir.com/docs/foundry/ai-fde/overview?utm_source=chatgpt.com)

**AI FDE Security**

[AI FDE Security & Governance](https://www.palantir.com/docs/foundry/ai-fde/security-and-governance?utm_source=chatgpt.com)

**AIP Analyst**

[AIP Analyst](https://www.palantir.com/docs/foundry/aip/analyst/overview?utm_source=chatgpt.com)

**Palantir MCP**

[Palantir MCP](https://www.palantir.com/docs/foundry/palantir-mcp/overview?utm_source=chatgpt.com)

The current developer documentation explicitly describes MCP and AI FDE as mechanisms through which AI agents can participate in development and operational workflows while remaining within the platform's permission and governance framework. ([Palantir][14])

---

# Cross-cutting references

These are worth adding to the report because they explain architectural principles that span multiple boxes.

## Ontology as the central control plane

[The Ontology System](https://www.palantir.com/docs/foundry/architecture-center/ontology-system?utm_source=chatgpt.com)

Palantir's architecture documentation describes the Ontology as integrating **data, logic, action and security**, with applications, agents and automations operating above it. ([Palantir][6])

## Platform architecture

[Foundry Platform Overview](https://www.palantir.com/docs/foundry/platform-overview?utm_source=chatgpt.com)

This is useful for explaining why AIP isn't simply an "LLM platform." Palantir positions it as operating together with Foundry's data/ontology/workflow capabilities.

## Architecture Center

[Palantir Architecture Center](https://www.palantir.com/docs/foundry/architecture-center/overview?utm_source=chatgpt.com)

This is probably the best **single external link to put at the top of the report**.

---

# External standards worth citing

For the "best practices" portion of your report, I would not rely exclusively on Palantir's own documentation. These provide independent architectural references.

| Area                    | Authoritative source                                                                                                                    |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| AI risk                 | [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework?utm_source=chatgpt.com)                       |
| Generative AI risk      | [NIST Generative AI Profile](https://www.nist.gov/itl/ai-risk-management-framework/ai-rmf-generative-ai-profile?utm_source=chatgpt.com) |
| LLM security            | [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/?utm_source=chatgpt.com)                  |
| Application security    | [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/?utm_source=chatgpt.com)                          |
| Zero Trust              | [NIST SP 800-207](https://csrc.nist.gov/pubs/sp/800/207/final?utm_source=chatgpt.com)                                                   |
| Observability           | [OpenTelemetry](https://opentelemetry.io/docs/?utm_source=chatgpt.com)                                                                  |
| Container orchestration | [Kubernetes Documentation](https://kubernetes.io/docs/?utm_source=chatgpt.com)                                                          |
| Cloud native            | [CNCF](https://www.cncf.io/?utm_source=chatgpt.com)                                                                                     |
| Event streaming         | [Apache Kafka](https://kafka.apache.org/documentation/?utm_source=chatgpt.com)                                                          |
| Stream processing       | [Apache Flink](https://nightlies.apache.org/flink/flink-docs-stable/?utm_source=chatgpt.com)                                            |
| CDC                     | [Debezium](https://debezium.io/documentation/?utm_source=chatgpt.com)                                                                   |
| DevOps                  | [DORA](https://dora.dev/?utm_source=chatgpt.com)                                                                                        |
| MCP                     | [Model Context Protocol Specification](https://modelcontextprotocol.io/specification/latest?utm_source=chatgpt.com)                     |

## One important correction for the report

Some labels in the image should **not** be treated as exact Palantir product names.

For example, labels such as **"Semantic", "Kinetic", "Dynamic", "Human + AI Decision Model", "Smart Cache", "Dynamic Retry", "Opinionated", "Multimodal Compute Services"** are useful architectural concepts shown in Palantir's diagram, but they do not necessarily correspond one-to-one with a publicly documented product or API.

The safest approach in the research report is therefore:

**Image terminology → Palantir documented capability → specific documentation → independent industry practice**

rather than assuming that every box represents a separately deployable Palantir component.

That distinction is particularly important if you're using this analysis for an **enterprise architecture review or designing an AIP-like architecture**, because otherwise the diagram can easily be mistaken for a literal product/component dependency diagram. Palantir itself describes the diagram as **12 "general categories of capability"**, which supports treating it as a conceptual architecture rather than a complete component-level deployment diagram. ([Palantir][1])

[1]: https://www.palantir.com/docs/foundry/architecture-center/aip-architecture?utm_source=chatgpt.com "AIP architecture overview • Palantir"
[2]: https://www.palantir.com/docs/foundry/getting-started/foundry-platform-summary-llm?utm_source=chatgpt.com "Foundry platform summary for LLMs • Palantir"
[3]: https://www.palantir.com/docs/foundry/model-catalog/overview?utm_source=chatgpt.com "AIP Model Catalog • Overview • Palantir"
[4]: https://www.palantir.com/docs/foundry/aip-observability/overview?utm_source=chatgpt.com "Ontology and AIP observability • Overview • Palantir"
[5]: https://www.palantir.com/docs/foundry/observability/overview?utm_source=chatgpt.com "Overview • Observability • Palantir"
[6]: https://www.palantir.com/docs/foundry/architecture-center/ontology-system?utm_source=chatgpt.com "The Ontology system • Palantir"
[7]: https://www.palantir.com/docs/foundry/ontology/core-concepts?utm_source=chatgpt.com "Core concepts • Palantir"
[8]: https://www.palantir.com/docs/foundry/announcements/2026-02?utm_source=chatgpt.com "February 2026 • Announcements • Palantir"
[9]: https://www.palantir.com/docs/foundry/security/overview?utm_source=chatgpt.com "Overview • Security • Palantir"
[10]: https://www.palantir.com/docs/foundry/security/access-control-propagation?utm_source=chatgpt.com "Concepts • Access control propagation • Palantir"
[11]: https://www.palantir.com/docs/foundry/getting-started/application-reference?utm_source=chatgpt.com "Application reference • Palantir"
[12]: https://www.palantir.com/docs/foundry/ai-fde/security-and-governance?utm_source=chatgpt.com "AI FDE • Security and governance • Palantir"
[13]: https://www.palantir.com/docs/foundry/architecture-center/overview?utm_source=chatgpt.com "Overview • Architecture center • Palantir"
[14]: https://www.palantir.com/docs/foundry/palantir-mcp/overview?utm_source=chatgpt.com "Palantir MCP • Overview • Palantir"
[15]: https://www.palantir.com/docs/foundry/ontology/applications?utm_source=chatgpt.com "Ontology-aware applications • Palantir"
