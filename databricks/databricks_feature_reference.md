# Azure Databricks — Product Feature Reference

_A high-level guide to every major product capability — March 2026 Edition_

---

## 1. Lakehouse Platform

The foundational architecture that unifies data lakes and data warehouses into a single, governed platform. These are the core building blocks that everything else in Databricks sits on.

| Product Feature             | What It Is & Why It Matters                                                                                                                                                                                                                                                                                                       |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Delta Lake**              | The open-source storage layer at the heart of the Lakehouse. Brings ACID transactions, time travel, schema enforcement, and scalable metadata to cloud data lakes. Every table in Databricks is a Delta table by default, enabling reliability guarantees previously only available in traditional warehouses.                    |
| **Unity Catalog**           | The unified governance layer for all data and AI assets across every Databricks workspace. Provides a three-level namespace (catalog.schema.object), fine-grained access control, automated data lineage, and centralized auditing. It governs tables, views, volumes, models, functions, and connections as first-class objects. |
| **Delta Sharing**           | An open protocol for secure, live data sharing across organizations without copying data. Recipients can consume shared datasets using Databricks, Apache Spark, pandas, Power BI, or any Delta Sharing–compatible client — regardless of whether they use Databricks.                                                            |
| **Lakehouse Federation**    | Query external databases such as PostgreSQL, MySQL, SQL Server, Snowflake, and BigQuery directly from Databricks without moving or copying data. Foreign catalogs register external sources as Unity Catalog assets, giving you governed, federated access through a single SQL interface.                                        |
| **UniForm**                 | Automatically generates Apache Iceberg and Apache Hudi metadata alongside Delta Lake metadata, enabling other engines (Trino, Presto, Snowflake, BigQuery) to read Delta tables natively. Eliminates format lock-in while keeping Delta as the write format.                                                                      |
| **Liquid Clustering**       | A next-generation data layout optimization that replaces traditional partitioning and Z-Ordering. Incrementally and automatically clusters data based on specified columns, adapting to changing query patterns without requiring full rewrites or manual OPTIMIZE commands.                                                      |
| **Predictive Optimization** | An autopilot for Delta table maintenance. Automatically identifies and runs OPTIMIZE, VACUUM, and file compaction operations based on table usage patterns and data characteristics. Eliminates the need for scheduled maintenance jobs and reduces storage costs.                                                                |

---

## 2. Serverless Compute

Fully managed, instant-start compute that eliminates the need to configure, manage, or wait for clusters. Databricks handles all infrastructure behind the scenes — you just run your workloads.

| Product Feature               | What It Is & Why It Matters                                                                                                                                                                                                                                                   |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Serverless SQL Warehouses** | Instant-on, auto-scaling SQL compute endpoints optimized for BI queries and dashboards. Warehouses start in seconds, scale elastically with query concurrency, and stop automatically when idle. This is the default and recommended compute for all SQL analytics workloads. |
| **Serverless Notebooks**      | On-demand compute that attaches automatically when you open a notebook — no cluster selection or startup wait. Ideal for interactive exploration, ad-hoc analysis, and development. Compute is reclaimed when the notebook is idle.                                           |
| **Serverless Jobs**           | Managed compute for production workflows and scheduled pipelines. Eliminates job cluster startup overhead and removes the need to size clusters manually. You define the workload; Databricks provisions the optimal infrastructure.                                          |
| **Serverless DLT Pipelines**  | Delta Live Tables pipelines that run on fully managed serverless infrastructure. Removes all cluster configuration from pipeline definitions, letting you focus entirely on transformation logic and data quality rules.                                                      |
| **Serverless Model Serving**  | Auto-scaling, pay-per-request infrastructure for deploying ML models and LLMs as REST endpoints. Scales to zero when not in use and bursts automatically under load. Supports custom models, foundation models, and external model proxies.                                   |

---

## 3. Mosaic AI

The integrated AI and machine learning suite for building, training, deploying, and monitoring models and GenAI applications — from traditional ML to compound AI systems with retrieval, tool use, and agents.

### 3.1 Model Development & Training

| Product Feature                  | What It Is & Why It Matters                                                                                                                                                                                                                                                       |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Foundation Model Fine-tuning** | Fine-tune open-source LLMs (Llama, Mistral, DBRX, etc.) on your proprietary data using managed infrastructure. Supports supervised fine-tuning and continued pre-training with built-in experiment tracking, evaluation, and one-click deployment to serving endpoints.           |
| **AutoML**                       | Automated machine learning that evaluates multiple algorithms, performs hyperparameter tuning, and generates transparent, editable notebooks for the best-performing models. Provides a baseline model with minimal effort for classification, regression, and forecasting tasks. |
| **Distributed Training**         | Multi-node, multi-GPU training with native integration into Spark. Supports PyTorch, TensorFlow, DeepSpeed, and Hugging Face Trainer out of the box via pre-configured Databricks Runtime for ML and GPU-optimized clusters.                                                      |
| **Feature Engineering**          | Unity Catalog–governed feature store for creating, discovering, and serving features. Includes Feature Serving for real-time inference and Online Tables that auto-sync batch features to a low-latency serving layer.                                                            |

### 3.2 MLflow & Model Lifecycle

| Product Feature                     | What It Is & Why It Matters                                                                                                                                                                                                                     |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MLflow Tracking**                 | Open-source experiment tracking fully integrated into Databricks. Automatically logs parameters, metrics, code versions, and artifacts for every training run. Provides a searchable, sortable experiment UI for comparing runs across teams.   |
| **Model Registry in Unity Catalog** | Centralized model versioning and lifecycle management governed by Unity Catalog. Models are first-class assets with access control, lineage, tagging, and aliases (Champion, Challenger) for promoting models across environments.              |
| **MLflow Evaluate**                 | Systematic model evaluation framework with built-in metrics for both traditional ML (accuracy, F1) and LLMs (toxicity, relevance, groundedness). Supports custom metrics and comparison across model versions for informed promotion decisions. |

### 3.3 AI Agent Framework

| Product Feature              | What It Is & Why It Matters                                                                                                                                                                                                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AI Playground**            | Interactive sandbox for experimenting with LLMs, RAG configurations, and tool-calling setups before writing any code. Compare models side-by-side, test system prompts, and prototype agent architectures. Export working configurations directly to notebooks.                 |
| **Agent Builder**            | A guided, low-code interface for assembling production-ready AI agents. Connect retrieval sources, define tools, configure guardrails, and evaluate quality — then deploy as a governed serving endpoint with a single click.                                                   |
| **Agent Evaluation**         | Purpose-built evaluation framework that uses judge LLMs to assess agent quality across dimensions like correctness, groundedness, relevance, and safety. Tracks quality over time and identifies regressions before they reach production.                                      |
| **Vector Search**            | Managed vector database for similarity search on embeddings. Supports Delta Sync indexes (auto-sync from Delta tables) and Direct Access indexes. Powers RAG applications by enabling semantic retrieval of relevant context for LLM grounding.                                 |
| **Mosaic AI Gateway**        | A unified governance layer for LLM access. Route requests across providers (Databricks-hosted, OpenAI, Anthropic, Azure OpenAI), enforce rate limits, log usage, and apply guardrails — all through a single endpoint. Includes built-in safety filters for content moderation. |
| **External Model Endpoints** | Proxy endpoints that connect Databricks to external LLM providers using a standardized API. Centralizes credential management, enables usage tracking, and lets you swap providers without changing application code.                                                           |

### 3.4 Model Serving

| Product Feature                 | What It Is & Why It Matters                                                                                                                                                                                                                              |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Real-time Serving Endpoints** | Deploy any MLflow-logged model as an auto-scaling REST API. Supports custom PyFunc models, scikit-learn, PyTorch, transformer models, and LLMs. Includes traffic splitting for A/B testing and canary deployments with automatic rollback.               |
| **Foundation Model APIs**       | Pay-per-token access to curated open-source LLMs (DBRX, Llama, Mistral, etc.) hosted by Databricks. No provisioning or GPU management required. Ideal for prototyping, low-volume production, and applications where simplicity outweighs customization. |
| **Provisioned Throughput**      | Dedicated, performance-guaranteed serving capacity for foundation models and fine-tuned LLMs. Provides predictable latency and throughput at a fixed cost, suitable for high-volume production workloads that require SLA-level performance.             |
| **Inference Tables**            | Automatic logging of all requests and responses to model serving endpoints into Delta tables. Enables monitoring, debugging, drift detection, and compliance auditing for deployed models without any additional instrumentation.                        |

---

## 4. Data Engineering

Purpose-built tools for building, orchestrating, and operating reliable data pipelines at scale — from raw ingestion to production-ready datasets.

| Product Feature                     | What It Is & Why It Matters                                                                                                                                                                                                                                                                        |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Delta Live Tables (DLT)**         | A declarative ETL framework where you define what your data should look like, and DLT handles the how: orchestration, cluster management, error handling, retries, and data quality enforcement. Supports SQL and Python, with built-in expectations for quality gates and automatic lineage.      |
| **Lakeflow Connect**                | Managed, no-code connectors for ingesting data from SaaS applications (Salesforce, Workday, SAP, ServiceNow), databases (via CDC), and file sources directly into the Lakehouse. Handles schema mapping, incremental loading, and error recovery automatically.                                    |
| **Auto Loader**                     | Incrementally and efficiently ingests new data files as they arrive in cloud storage (ADLS Gen2, S3, GCS). Automatically infers and evolves schema, handles exactly-once guarantees, and scales to millions of files. The recommended ingestion method for all file-based workloads.               |
| **Workflows (Jobs)**                | The orchestration engine for scheduling and managing multi-task data pipelines. Supports DAGs with task dependencies, conditional logic, parameterization, and repair/rerun of failed tasks. Integrates notebooks, Python scripts, JARs, dbt, SQL, and DLT pipelines as task types.                |
| **Structured Streaming**            | Apache Spark's unified batch-and-streaming engine, optimized in Databricks with Photon acceleration, enhanced state management, and Trigger.AvailableNow for efficient micro-batch processing. Enables exactly-once stream processing from Kafka, Event Hubs, and cloud storage.                   |
| **Photon Engine**                   | A Databricks-native vectorized query engine written in C++ that replaces the Spark JVM execution layer for supported operations. Delivers up to 12x performance improvement on SQL and DataFrame workloads with zero code changes. Enabled by default on serverless and Photon-capable runtimes.   |
| **Databricks Asset Bundles (DABs)** | An infrastructure-as-code framework for defining, testing, and deploying Databricks projects (jobs, pipelines, ML models, permissions) as version-controlled YAML bundles. Supports CI/CD workflows, environment promotion (dev/staging/prod), and templating for standardized project structures. |

---

## 5. Databricks SQL & BI

A complete SQL analytics experience for data analysts and BI professionals, providing high-performance querying, natural-language exploration, and interactive dashboarding — all directly on Lakehouse data.

| Product Feature                            | What It Is & Why It Matters                                                                                                                                                                                                                                                                                         |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SQL Warehouses**                         | Dedicated, tunable compute endpoints optimized for SQL workloads. Available in Serverless (recommended), Pro, and Classic tiers. Features intelligent workload management, automatic query queuing, result caching, and auto-stop/auto-scale to balance performance with cost.                                      |
| **SQL Editor**                             | A browser-based, full-featured SQL authoring environment with intelligent autocomplete, syntax highlighting, multi-statement execution, and inline query profiling. Supports parameterized queries with interactive widgets like date pickers and dropdowns.                                                        |
| **Lakeview Dashboards (AI/BI Dashboards)** | The next-generation dashboarding experience with a drag-and-drop canvas builder, AI-assisted chart creation, cross-widget filtering, and scheduled email snapshots. Replaces legacy SQL dashboards with a more polished, collaborative authoring experience.                                                        |
| **Genie (AI/BI)**                          | A natural-language interface that lets business users ask questions about their data in plain English. Genie translates questions to SQL, executes them, and returns visual answers. Domain experts can curate Genie Spaces with trusted datasets, definitions, and example questions to ensure accurate responses. |
| **Query Profile**                          | A visual, interactive execution plan for every SQL query showing operator-level metrics, data flow, spill-to-disk, and partition pruning effectiveness. Essential for diagnosing slow queries and understanding how the optimizer processes your SQL.                                                               |
| **Databricks Assistant**                   | An AI coding assistant embedded across notebooks and the SQL editor. Generates SQL and Python from natural-language prompts, explains existing code, debugs errors, and suggests optimizations. Context-aware with access to your Unity Catalog schema for accurate completions.                                    |
| **Alerts**                                 | Automated monitors on SQL query results that trigger notifications (email, Slack, webhook) when data meets specified conditions. Useful for data quality checks, threshold monitoring, and operational alerting without building separate monitoring infrastructure.                                                |

---

## 6. Governance, Security & Compliance

Enterprise-grade security controls, data governance policies, and compliance certifications that enable organizations to operate Databricks in regulated environments with confidence.

### 6.1 Data Governance

| Product Feature                    | What It Is & Why It Matters                                                                                                                                                                                                                                                     |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Fine-Grained Access Control**    | SQL-standard GRANT/REVOKE permissions on every Unity Catalog object: catalogs, schemas, tables, views, volumes, models, and functions. Supports ownership transfer and permission inheritance for scalable governance without per-object configuration.                         |
| **Row Filters & Column Masks**     | Dynamic, policy-based data access at the row and column level. Row filters restrict which rows a user can see based on SQL expressions; column masks transform sensitive column values (e.g., masking SSNs) based on the querying user's identity or group membership.          |
| **Attribute-Based Access Control** | Tag data assets with sensitivity labels or classifications, then define policies that automatically apply access rules based on those tags. Enables governance at scale without manually managing permissions on every individual table or column.                              |
| **Data Lineage**                   | Automated, column-level lineage tracking across notebooks, jobs, DLT pipelines, and SQL queries. Visualized in the Unity Catalog UI and queryable via system tables. Shows how data flows from source to destination, who accesses it, and how transformations are applied.     |
| **System Tables**                  | A set of built-in, queryable Delta tables exposing audit logs, billing usage, compute activity, data lineage, and access patterns. Enables you to build custom dashboards, compliance reports, and cost attribution models using standard SQL on your own operational metadata. |

### 6.2 Network & Infrastructure Security

| Product Feature                 | What It Is & Why It Matters                                                                                                                                                                                                                                |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Azure Private Link**          | Connects Databricks to your Azure VNet over Microsoft's backbone network, ensuring that no data traverses the public internet. Supports both front-end (UI/API) and back-end (data plane) private connectivity for defense-in-depth network architecture.  |
| **VNet Injection**              | Deploy Databricks compute resources into your own Azure Virtual Network, giving you full control over network security groups, route tables, DNS, and peering. Required for environments that mandate all traffic flow through corporate network controls. |
| **Customer-Managed Keys (CMK)** | Encrypt Databricks-managed storage, notebook contents, and other platform data using your own Azure Key Vault keys. Provides data sovereignty and the ability to revoke access to your data independently of Databricks.                                   |
| **Secure Cluster Connectivity** | Launches clusters without public IP addresses, so all communication between the control plane and compute plane occurs over a secure, authenticated tunnel. Eliminates inbound network exposure and simplifies firewall rules.                             |
| **IP Access Lists**             | Restrict API and UI access to Databricks workspaces based on allowlisted IP ranges. Provides an additional layer of perimeter security for environments that require network-level access restrictions.                                                    |
| **Compliance Certifications**   | Databricks on Azure holds SOC 2 Type II, HIPAA BAA, FedRAMP High, ISO 27001, ISO 27017, ISO 27018, PCI DSS, and GDPR compliance certifications. Enhanced Security & Compliance (ESM) mode adds additional hardening for regulated industries.              |

---

## 7. Developer Experience

The tools, integrations, and workflows that make developing on Databricks productive — from interactive notebooks to IDE-native development and CI/CD automation.

| Product Feature           | What It Is & Why It Matters                                                                                                                                                                                                                                               |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Notebooks**             | Multi-language (Python, SQL, Scala, R) collaborative notebooks with real-time co-editing, built-in visualizations, variable explorer, and interactive widgets. Support %run chaining, %pip installs, and deep integration with MLflow for automatic experiment tracking.  |
| **Databricks Connect v2** | Run Spark DataFrames and Spark SQL from your local IDE (VS Code, PyCharm, IntelliJ) while executing on a remote Databricks cluster. Enables local debugging, unit testing, and IDE-native development workflows against full Databricks compute.                          |
| **VS Code Extension**     | First-party Visual Studio Code extension for browsing workspaces, editing notebooks in IDE format, managing clusters, running jobs, and synchronizing local projects with Databricks. Includes integrated debugging support for Databricks Connect.                       |
| **Git Folders (Repos)**   | Native Git integration that syncs workspace folders with Azure DevOps, GitHub, GitLab, or Bitbucket repositories. Enables pull request workflows, branch-based development, and version control for notebooks, Python files, and SQL files directly in the Databricks UI. |
| **Databricks CLI**        | A command-line interface for managing workspaces, clusters, jobs, secrets, Unity Catalog assets, and Databricks Asset Bundles. The primary tool for scripting deployments and integrating Databricks operations into CI/CD pipelines.                                     |
| **Databricks SDKs**       | Official client libraries for Python, Java, Go, and R that provide idiomatic access to all Databricks REST APIs. Used for building custom integrations, automation scripts, and platform tooling with full type safety and authentication handling.                       |
| **Terraform Provider**    | Official HashiCorp Terraform provider for declaratively provisioning and managing Databricks workspaces, clusters, jobs, Unity Catalog assets, permissions, and networking configurations as version-controlled infrastructure code.                                      |
| **REST APIs**             | A comprehensive set of REST APIs covering every platform capability: workspace management, clusters, jobs, SQL warehouses, Unity Catalog, model serving, secrets, and more. The foundation for all programmatic interaction with Databricks.                              |

---

## 8. Marketplace & Partner Ecosystem

An open ecosystem for discovering pre-built data products, connecting partner tools, and extending the Lakehouse with third-party integrations.

| Product Feature            | What It Is & Why It Matters                                                                                                                                                                                                                                                                                       |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Databricks Marketplace** | A discovery platform for third-party and open datasets, pre-trained ML models, solution accelerators, and notebook templates. Data products are shared via Delta Sharing and appear directly in your Unity Catalog, enabling instant access without data movement or ETL.                                         |
| **Partner Connect**        | One-click provisioning of integrations with BI tools (Power BI, Tableau, Looker), ingestion platforms (Fivetran, Airbyte), data quality tools (Monte Carlo, Great Expectations), orchestrators (Airflow, Dagster), and more. Automates service principal creation, warehouse assignment, and credential exchange. |
| **Databricks Apps**        | Deploy custom data applications (Streamlit, Dash, Gradio, Flask) directly within Databricks. Apps run on managed infrastructure with workspace-native authentication, Unity Catalog access, and automatic HTTPS. Eliminates the need for external hosting and separate access control.                            |
| **Clean Rooms**            | A secure, privacy-preserving environment for multi-party data collaboration. Multiple organizations can run approved computations on combined datasets without exposing raw data to each other. Governed by Unity Catalog and powered by Delta Sharing for zero-copy access.                                      |

---

## 9. Cost Management & Administration

Platform-level tools for managing spending, monitoring operations, and administering workspaces across the organization.

### 9.1 Cost Management

| Product Feature               | What It Is & Why It Matters                                                                                                                                                                                                                                               |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Budgets & Alerts**          | Define spending budgets at the account or workspace level and receive email alerts when consumption approaches or exceeds thresholds. Enables proactive cost control without requiring manual monitoring of billing dashboards.                                           |
| **System Tables (Billing)**   | Query granular, row-level billing data as Delta tables using standard SQL. Includes DBU consumption by workspace, cluster, job, user, and SKU. Build custom chargeback models, cost attribution dashboards, and anomaly detection queries on your own billing data.       |
| **Cluster Policies**          | Admin-defined templates that restrict what cluster configurations users can create: instance types, max workers, autoscaling ranges, spot instance ratios, and runtime versions. Prevents accidental overspending while still giving users flexibility within guardrails. |
| **Tag-based Cost Allocation** | Apply custom key-value tags to clusters, jobs, SQL warehouses, and other resources. Tags flow through to billing records, enabling department-level chargeback, project cost tracking, and spend categorization in reporting tools.                                       |

### 9.2 Administration & Monitoring

| Product Feature                  | What It Is & Why It Matters                                                                                                                                                                                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Account Console**              | The centralized administration portal for managing all Databricks workspaces, users, service principals, metastores, network configurations, and billing across an Azure subscription. Provides a single pane of glass for platform operations.                                |
| **Lakehouse Monitoring**         | Automated statistical profiling, data drift detection, and anomaly alerting for Delta tables and model serving endpoints. Creates monitor dashboards with configurable refresh schedules and alert thresholds, enabling proactive data quality management without custom code. |
| **Audit Logs / Diagnostic Logs** | Comprehensive audit trail of all workspace activity: user logins, cluster events, data access, job executions, and permission changes. Exportable to Azure Monitor, Log Analytics, or storage accounts for integration with enterprise SIEM and compliance systems.            |
| **Secret Management**            | Securely store and access credentials, API keys, and tokens using Databricks-native secret scopes or Azure Key Vault–backed scopes. Secrets are never exposed in notebook outputs, logs, or the Spark UI, and access is controlled via ACLs.                                   |
