# Agent platform architecture

## Table of Contents

1. **Introduction**
2. **Motivation (WHY)**
   - 2.1. Business Pain Points
   - 2.2. Required Technology Capabilities (WHY do we need AI Agents?)
   - 2.3. Approach to build AI Agents (WHY do we need a Platform?)
3. **New Capabilities enabled by Agentic AI Platform (WHAT)**
   - 3.1. About AI agents and Agentic AI Platform
   - 3.2. New capabilities enabled by Agentic AI Platform
   - 3.3. Benefits
4. **Proposed Solution (TO-BE) (HOW)**
   - 4.1. Current Technology Capabilities
   - 4.2. Targeted Technology Capabilities
     - 4.2.1. Guiding Principles
     - 4.2.2. Targeted Technology Capabilities
     - 4.2.3. To-be Maturity level of Autonomy in AI Agents
   - 4.3. IT Architecture Components
   - 4.4. Solution options analysis
   - 4.5. Tech Stack and License
   - 4.6. Target Architecture
   - 4.7. Security Considerations
     - 4.7.1 Data and AI Security
     - 4.7.2. System security
     - 4.7.2. Compliance
5. **Appendix**
   - 5.1. Use cases and Functional Requirements
   - 5.2. Non-Functional Requirements
   - 5.3. Popular Agentic AI Patterns
   - 5.4. Example of Human-AI Collaboration flow
   - 5.5. Example of data flow in Agentic AI
   - 5.6. Agentforce vs LangChain comparison

---

## 1. Introduction

In this document, we are seeking approval for the adoption of an Agentic AI Platform to transform banking operations by enhancing customer engagement, streamlining workflows, and driving growth, with this scope of endorsement.

### In-scope

- Address the need for an Agentic AI Platform that fulfill the approached use cases, while having the capabilities of no-code/low-code agent, UI, Workflow building to democratize Agentic AI access even for non-technicals.
- Adopt exploration approach using open-source technologies for the first iteration of Agentic AI Platform in the bank to save cost, remain flexible to extend, and have option to upgrade to Enterprise version when needed, while later architecture improvements can be refined and updated in subsequent documents.
- Adopt Agentic AI for 3 high potential use cases (Refer to *5.1. Use cases and Functional Requirements* for more details)
- Resolve road-blockers to enable the bank's AI/ Agentic AI Adoption: Implement Model selection, Data Protection, Data Poisoning Prevention, Data Loss Prevention, Model Guardrails, prompt validation, prompt logging and audit. (Which is scope of the AI Gateway project)

### Out of the scope

- Data Processing, Model development & Model training for specific use cases
- Alignment with OPS for specific use cases
- AI Agents with a high-level of autonomy (Refer to *4.2.3. To-be Maturity level of Autonomy in AI Agents* for more details)
- Agentic AI Operating model
- Approval from the steering committee

---

## 2. Motivation (WHY)

### 2.1. Business Pain Points

| # | Business Pain Points | Examples |
| --- | --- | --- |
| 1 | **Inefficient Human Dependency, reducing productivity and scalability** | Repetitive manual tasks with Inefficient Human Dependency prevent staff from focusing on high-value activities (e.g., customer engagement) and reduce scalability:<br>• RMs manually generate call summaries and update CRM with customer interactions, spending hours daily.<br>• BAs manually generate BRDs and user stories, consuming 50-60% of their time |
| 2 | **Poor Customer Understanding** | • Frequent staff turnover (4-5 changes per year) disrupts customer engagement continuity<br>• Generic product offers miss tailored opportunities based on customer preferences or behavior. |
| 3 | **Inconsistent Quality** | • Frequent staff turnover (4-5 changes per year) disrupts customer engagement continuity<br>• Output quality depends on the skill levels and knowledge of RMs and BAs. |
| 4 | **Delayed Follow-Ups and Communication** | Customer data or emails are manually entered, leading to delays. Slow follow-ups frustrate customers, especially priority clients. |
| 5 | **High Operational Costs** | Manual processes, such as CRM updates, combined with frequent staff retraining due to turnover, consume excessive staff time, increasing operational costs |

### 2.2. Required Technology Capabilities (WHY do we need AI Agents?)

| # | Business Pain Points | Required Technology Capabilities | Potential Solution (Illustrative) — Option 1 (AI Agents) | Potential Solution (Illustrative) — Option 2 (Others) |
| --- | --- | --- | --- | --- |
| 1 | **Inefficient Human Dependency, reducing productivity and scalability** | • Handle processes with more automation and less human intervention<br>• Context-dependent process: able to adjust workflows dynamically (define when and what to create/update in CRM)<br>• Integrate with enterprise applications (i.e. to create/update leads) | • AI Agents with multi-step reasoning, leveraging Retrieval-Augmented Generation (RAG) from the bank knowledge base, and executing tasks using LLMs for context-dependent workflows<br>• Integrate with enterprise applications (e.g., CRM systems like Salesforce for creating/updating leads) | • BPM Platforms: Configure predefined workflows, but limited for dynamic, context-dependent processes.<br>• RPA: Automates CRM updates and lead creation in the bank's systems via APIs (e.g., Salesforce), reducing manual effort. |
| 2 | **Poor Customer Understanding** | • Able to retrieve standardized and reusable knowledge bases (i.e. Customer knowledge base)<br>• Context-aware responses result in personalized engagement, increasing conversion rates and satisfaction | • AI Agents to access data across corporate RAG knowledge base and execute tasks using LLMs, handling context-dependent workflow<br>• Support popular AI Agent Patterns (i.e. Context-aware Adjusting) | • Data Lake with Analytics: Aggregates customer data for real-time insights.<br>• ML Models: Deliver tailored product offers |
| 3 | **Inconsistent Quality** | • Able to retrieve standardized and reusable knowledge bases (i.e. Customer's interactions, Products knowledge base).<br>• Provide recommendations (i.e. Engagement scripts, BRD) with consistent quality | • AI Agents to access data across corporate RAG knowledge base<br>• Other standardized AI Agents for specific tasks (i.e. Provide personalized recommendations) | • Data Lake with Analytics: Aggregates customer interactions for insights via tools like Power BI.<br>• Knowledge Hub/ Knowledge Management system: Provides standardized product knowledge.<br>• ML Models/Gen AI: Delivers consistent engagement scripts and BRDs |
| 4 | **Delayed Follow-Ups and Communication** | • Support faster decision-making and follow-up actions with less human intervention | • AI Agents for Autonomous Decision-Making (i.e. Action identification)<br>• Integrate with enterprise applications (i.e. to create/update leads) | • ML: Identifies actions<br>• RPA: Automates CRM updates and lead creation in the bank's systems via APIs (e.g., Salesforce), reducing manual effort. |
| 5 | **High Operational Costs** | • The root causes of High Operational Costs include the business pain points listed above, and thus the Required Technology Capabilities have already been covered above. | • The solutions have already been covered above. | • The solutions have already been covered above. |

**High-level Assessment**

| Option | Pros | Cons |
| --- | --- | --- |
| **AI Agents** | Work well with dynamic, context-dependent processes requiring less human intervention (autonomous decision-making) or multi-step reasoning. | Might trade latency and cost for better task performance. Should consider when this tradeoff makes sense. |
| **Others (BPM, RPA, ML)** | Works well with deterministic, rule-based tasks with predefined steps. They excel in structured environments. | Struggles with dynamic, context-dependent processes, or multi-step reasoning |

> **Recommendation:** Adopt AI Agents to enable the opportunities of the bank in optimize client engagement and operations. AI Agents with multi-step reasoning, leveraging Retrieval-Augmented Generation (RAG) from the bank knowledge base, and executing tasks using LLMs for context-dependent workflows.

### 2.3. Approach to build AI Agents (WHY do we need a Platform?)

| Approach | Pros | Cons |
| --- | --- | --- |
| **With Agentic AI Platform** (e.g., using LangChain framework) | • Fast prototyping with pre-built tools, memory, workflows<br>• Built-in error handling, scalability<br>• Security controls (e.g., DLP, Guardrails, prompt filtering, encryption)<br>• Standardized implementation<br>• Governance for compliance/ethics | • Dependency on framework updates<br>• Less control over logic<br>• Potential overhead |
| **Without Agentic AI Platform** (From Scratch e.g., ReAct Loop, Function Calling) | • Fully customizable, lightweight<br>• Better understanding of core mechanics (e.g., LLM prompts, tool parsing), aiding debugging | • Time-consuming to code/maintain<br>• Risk of errors in complex workflows<br>• Scalability challenges<br>• Lacks security controls (e.g., DLP, Guardrails, prompt filtering, encryption)<br>• No standards, causing inconsistency<br>• Missing governance for compliance/ethics |

> **Recommendation:** Building AI agents with Agentic AI Platform is recommended for their rapid prototyping, built-in security, and scalability, streamlining development. They offer standardized workflows, security and governance, ensuring security, consistency and compliance. This approach saves time and reduces risks compared to custom-built solutions.

---

## 3. New Capabilities enabled by Agentic AI Platform (WHAT)

### 3.1. About AI agents and Agentic AI Platform

Agentic AI/AI Agents, a rapidly emerging trend in 2025, transforms work by managing end-to-end processes with minimal human intervention, evolving from narrow tools to team members, boosting enterprise productivity and growth:

- Tech giants (Google, Microsoft, AWS, OpenAI) and startups are racing to deploy AI agent platforms .
- Adoption is strong: **14%** of organizations have implemented AI agents (12% partial, 2% full), **23%** are piloting, and **61%** are exploring deployment *(Ref: Capgemini research)*.

**What are AI Agents?**

- AI agents are programs/software that are connected to the business environment within a defined boundary, make autonomous decisions, and act to achieve specific goals with or without human intervention.
- With the latest advances in reasoning AI models, AI agents are able to break down tasks, "reason" through potential pathways to find solutions to the given problem, try those solutions, and present successful outcomes.

**What is Agentic AI Platform?**

- Agentic AI is a broader term and includes systems, tools, and technologies that enable agents to function.
- Agentic systems have been around since well before the current AI boom and can be built using both AI and non-AI technologies. Recent advances in AI, language models, and reasoning capabilities have driven the rise of agentic AI systems.

### 3.2. New capabilities enabled by Agentic AI Platform

| ID | Capability | Description |
| --- | --- | --- |
| **C01** | Autonomous Decision-Making *(starting with low level of autonomy)* | Agents reason (via scalable orchestrators), plan, and execute tasks using LLMs, handling context-dependent workflow and ensuring flexibility and performance across use cases. |
| **C02** | Standardized Agent-to-Agent, Agent-to-Service Collaboration | Enable each agent to collaborate with other agent and systems/tools using standardized protocols (i.e. A2A for Agent-to-Agent interactions, Model Context protocol - MCP for Agent-to-Service integrations) |
| **C03** | Support popular AI Agent Patterns | (i.e. Reflection, Specific task handling, Hierarchical task delegation, Coordinating, Human-in-the-loop Collaboration, Context – Aware Adjusting, Self-Learning, RAG-Based Agent) |
| **C04** | Context and Memory Management | Retain interaction history and context (e.g., LangChain's memory systems) for consistent, personalized responses. |
| **C05** | No-Code/Low-Code Agent Builder with model selections | Access to both self-built models and 3rd party models |
| **C06** | Provide flexible and standardized Security Controls | To support flexibility in future development of Agentic AI Applications: Data Protection, Data Poisoning Prevention, Data Loss Prevention, model Guardrails, Prompt validation, logging and audit |

### 3.3. Benefits

**Realize the benefits from AI Agents:**

- **Increase Productivity & scalability**, handle Context-Dependent processes with more automation and less human intervention: i.e. Automate repetitive post-call tasks to allow RMs to focus on client relationships.
- **Provide recommendations** (i.e. Engagement scripts, BRD) with consistent quality
- **Reduce Follow-Up Delays**: Support faster decision-making and follow-up actions with less human intervention
- **Increase customer satisfaction**: provide Context-Aware, Personalized Engagement
- **Allow staff to focus on high-value activities** (e.g., customer engagement) by reducing Repetitive Manual Tasks

**Support to build AI Agents with rapid prototyping, built-in security, and scalability**

- Offer flexible and standardized security controls, ensuring security and compliance.
- Agentic AI/AI Agents adoption can also enable the bank to establish itself as a market leader and drive market dominance with better reputation and competitiveness.

---

## 4. Proposed Solution (TO-BE) (HOW)

### 4.1. Current Technology Capabilities

> No Agentic AI capability exists currently.

### 4.2. Targeted Technology Capabilities

#### 4.2.1. Guiding Principles

- **Principle 01:** AI agents designed to become coworkers in Human-AI Collaboration
  - Humans play the role of domain experts, providing new thinking and new initiatives. AI Agent handles repetitive tasks according to standards. 
- **Principle 02:** Designing with security in mind from the start, focus on safety to build trust and credibility 
  - Built-in controls and monitoring help mitigate risk and preserve operational stability, ensuring systems operate reliably and securely under all conditions.
  - Reinforced by ethics and responsible AI, where transparency, fairness, and accountability are foundational.
- **Principle 03:** Extendable 
  - The system must be extendable since we are in the exploration phase and currently evaluating multiple use cases for the potential application of Agentic AI.
- **Principle 04:** Interoperability
  - Allowing seamless Agent-to-Agent inter-communication and integration with other tools, platforms, and environments
- **Principle 05:** Evaluating use cases and finding the simplest solution possible, and only use Agentic AI when needed.
  - **Pros:** Agentic AI is the better option (than traditional workflow management systems) when flexibility and model-driven decision-making are needed at scale. They work well with dynamic, context-dependent processes requiring faster decision-making, natural language understanding, or multi-step reasoning.
  - **Cons:** Agentic AI might trade latency and cost for better task performance, and you should consider when this tradeoff makes sense.

#### 4.2.2. Targeted Technology Capabilities

In the enterprise-wide architecture, Agentic AI Platform is positioned as a capability in the **System of Innovation Layer**. It is a global revolution in which Agentic AI helps to change the way we work. Agentic AI can manage and execute end-to-end processes with less human intervention, evolving from tools (which are focusing on narrow, predefined tasks) to team members and marking a new era in enterprise productivity, efficiency, and growth. It can enable agile, experimental applications like personalized customer engagement and automation for internal operations. Agentic AI Platform can be integrated into Interaction Layer, Systems of Differentiation, System of Record, Data & Analytics to get input and deliver outputs.

| Layer | Targeted Capabilities |
| --- | --- |
| **Input Layer** | • Multiple data sources feed into the system.<br>&nbsp;&nbsp;◦ Multiple data sources can be ingested into Data lake zones for Agentic AI system, which is a hyperscale data engine providing all of the data and metadata Agentic AI system needs. These Data lake zones enable Agentic AI to access and interpret enterprise knowledge from every relevant source—including files, emails, images, audio, videos, websites, service tickets, Customer 360....<br>&nbsp;&nbsp;◦ In addition, AI agents can get input data from enterprise applications (e.g., CRM, ERP) (using MCP)<br>• AI agents can receive requirements/triggers, inputs, goals and feedback from users' prompt (on Agent Builder and UI for end-users) |
| **Agent Orchestration Layer** | An "orchestrator" agent can break down the larger request/problem into smaller pieces to be tackled using agents, significantly reducing the cost and time needed to arrive at an outcome. By employing reasoning models, they can break down the goals into specific actions or steps and prioritize them. |
| **AI Agent Layer** | • Support popular AI Agent Patterns *(Refer to Appendix for descriptions)*. Highlighted patterns:<br>&nbsp;&nbsp;◦ **RAG-Based Agent:** For task execution, AI agents may access internal data and enterprise tools systems (such as knowledge bases and enterprise systems, external tools (such as web search, third-party databases)<br>&nbsp;&nbsp;◦ **Coordinating Agent:** AI agent interact with other AI agents, or request clarification from users<br>&nbsp;&nbsp;◦ **Human-in-the-Loop Collaboration:** Although AI agents could be set up to operate autonomously, they still function within the scope of execution. Critical situations will require human oversight or decision-making<br>&nbsp;&nbsp;◦ **Planning Agent:** The above iterations can be repeated in different combinations and until the stated goals have been achieved.<br>&nbsp;&nbsp;◦ **Context-Aware Agent:** The agent utilizes its memory to maintain context<br>&nbsp;&nbsp;◦ **Self-Learning and Adaptive Agents:** Learn from previous iterations or past experiences, and improve its performance over time. |
| **Data Storage / Retrieval Layer** | • **Diverse Data Repositories (Unified storage):** It unifies structured and unstructured data, providing Agentic AI with a trusted, contextual understanding.<br>• To mitigate LLM hallucinations: leverage external knowledge sources like vector databases and knowledge graphs, integrated via RAG for factual accuracy.<br>&nbsp;&nbsp;◦ **Vector stores:** A vector database indexes embeddings for fast retrieval of relevant knowledge, enabling agents to access contextually accurate data.<br>&nbsp;&nbsp;◦ **Knowledge graphs:** map complex relationships between entities, concepts, and events. This interconnected structure enhances reasoning and supports deeper insights across domains. |
| **Output Layer** | The following combination ensures that outputs are not only relevant and actionable but also contribute to the agent's ongoing learning and system-wide intelligence:<br>• Delivering custom output formats tailored to specific user needs or systems.<br>• Beyond presentation, agents actively perform knowledge updates, refining their internal models based on new information and interactions.<br>• Additionally, they generate enriched data, enhancing raw inputs with context, insight, or structure. |
| **Service Layer** | Together, the following capabilities enable a highly responsive, scalable, and intelligent service layer that supports diverse operational demands.<br>• With **multi-channel delivery systems**, AI agents can seamlessly operate across platforms i.e. chat, enterprise applications, ensuring consistent and contextual user engagement.<br>• **Provide recommendations:** Agents deliver meaningful recommendations without manual intervention. Complementing this is the use of adaptive response mechanisms, which tailor outputs and interactions based on user's inputs, user's behavior, and environmental changes. |
| **Foundation Layer** | • **Security Control** to enforce ethical, operational, and safety standards: Data Protection, Data Poisoning Prevention, Data Loss Prevention, Model Guardrails, Prompt template/validation, Logging and audit, Executing constraints, Authentication and access control<br>• **Workflow and Agent Builder:** No-Code/Low-Code Agent Builder with model selections (Access to both self-built models and 3rd party models)<br>• **Standardized Interaction protocol:** Agent-to-Agent protocols (i.e. A2A) help collaboration between agents, while Agent to services protocol (.i.e. Model Context Protocol – MCP aids in accessing external tools) |

#### 4.2.3. To-be Maturity level of Autonomy in AI Agents

- **Current Maturity level:** Level 0
- **Proposed To-be Maturity level:** Level 1-2

**Rationale:**

- Principle 01 — AI agents designed to become coworkers in Human-AI Collaboration
- Businesses need confidence in AI systems before granting them higher level of autonomy. To build trust, we should start with low level of autonomy (Level 1-2), where AI provides recommendations (and provides value) with predefined workflows, but humans make all final decisions.

### 4.3. IT Architecture Components

### 4.4. Solution options analysis

**Overview:** The Agentic AI Framework high-level comparisons:

| Framework | Multi-agent workflow orchestration | App Development And Deployment w low-code support | Key Technical Features | Assessments |
| --- | --- | --- | --- | --- |
| **LangChain / LangGraph / LangFlow / LangFuse** | Stateful multi-agent workflow orchestration with cyclical operations | Visual low-code development and deployment of AI applications | Modular components (Chains, Agents, Retrieval, Models, Prompt Templates, Output Parsers); Async processing; Vector DB integrations; Scalable infrastructure support; Concurrency management. Graph-based architecture (Nodes, Edges); Stateful memory; Cyclical graphs; Human-in-the-Loop (HITL) support; Security (tool access limits). Drag-and-drop UI; Pre-built components; Custom Python components; Model Context Protocol (MCP); API deployment. LLM agnostic. Security built in security capability and combine with GuardRail (Prompt guarding, data privacy, DLP, tool access limits, output sanitization, exe constraints, auditability …). Open source and highly customization. | Manages complex state and iterative logic (Orchestration Layer), built-in HITL (Core Principle), strong for multi-agent collaboration. Accelerates development (Orchestration Layer), enables non-technical users (Interaction Layer), seamless integration with existing systems (Integration Layer) via MCP. High flexibility for complex workflows (Orchestration Layer), extensive integrations with data sources (Data & Knowledge Layer), scalable (Infrastructure Layer). LangChain offer non-code agent, UI, workflow builder for low/non-technical people. Also offer development/code-focus in-dept agent building with LangGraph with open source and enterprise offering. |
| **AWS Bedrock Agents** | Fully managed service for building and scaling generative AI agents. Support hierarchy and routing modes. AWS tool for tracing and debugging. Primarily support real-time chat assistant. Support memory retention. Not supporting building individual apps. | Simplified LLM integration. Managed Infrastructure. | Leverages AWS FMs, APIs, data sources; Multi-agent collaboration; RAG capabilities; Memory retention; Code interpretation; Built-in Guardrails (Automated Reasoning, topic blocking); GuardRail (Prompt guarding, data privacy, DLP, auditability …) | Tie-in with AWS ecosystem. No-to-low code deployment of Agents leading to less re-usability and porting to other platforms. Not accessible for low-tech staff to build agent since it's required access to AWS development console and advanced AWS configuration |
| **AgentForce (Salesforce)** | Native multi-agent collaboration, complete multi-step processes with minimal human intervention. Command Center for real time monitoring, tracing, health check of Agent Health. Modularised Agent design with unified data and processes protocols in Salesforce sales cloud, service cloud, marketing cloud, etc. Support MCP enabler especially with Salesforce services. | Low-code, no-code tools to build app: agent builder, prompt builder, model and flow builders. | Query databases, unstructured data, external tools/APIs; Real-time learning; Prompt chaining; Salesforce CRM integration; MCP support; Low-code Agent Builder; AI reasoning for sentiment. Salesforce Ecosystem Dependency. | Deep integration with Salesforce CRM (Core Banking & Enterprise Systems Layer), comprehensive guardrail framework (Data & Knowledge Layer, Core Principle), strong focus on customer-centricity. Powerful but heavily focus and depend on Salesforce ecosystem, products and data. |
| **CrewAI** | Collaborative multi-agent orchestration. Flexible workflow processes. Memory management, tool integration. | Code-heavy approach using python, development-focus provide CLI scaffolding tool to build application focus on backend capabilities. LLM agnostic. | Role-based agent architecture; Supports sequential and hierarchical processes; Intelligent collaboration; Dual workflow management; Task guardrails via output validation. | Facilitates complex task decomposition and collaboration (Orchestration Layer), flexible workflow control, built-in guardrails for output validation (Core Principle) with RoleBased Agent, agent-to-agent collaboration. Strong open-source framework but lacking in term of observability (not included, must build observability), and UI builder support. Very development-focus, but could be integrate into LangChain/LangFlow later on if the need arise. |
| **Microsoft AutoGen** | Flexible multi-agent conversation framework. Memory management, tool integration. observability and debugging OOTB. | Autogen studio is no/low-code for UI builders. LLM agnostic. | Asynchronous, event-driven architecture; LLM-as-agent abstraction; Customizable memory/context; Scalability & extensibility; Built-in observability/debugging; Cross-language support; Azure ecosystem integration; Data loss prevention features. | Highly flexible and extensible (Orchestration Layer), strong support for multi-agent conversations, robust debugging and observability (Core Principle), deep integration with Microsoft enterprise tools for DLP. Various tools that support rapid prototyping, does not focus on implementing authentication and other security measures but not ready for production yet. Not ready for production yet, focus on rapid prototyping. |

From the assessment above, we've short-listed 1 option:

- **Build:** LangChain, LangGraph, LangFlow & LangFuse

> Refer to *4.3. Agentforce vs LangChain comparison* for detailed comparison between LangChain and AgentForce
>
> **Recommendation:** Build the Agentic AI Framework (using LangChain, LangGraph, LangFlow & LangFuse) to ensure full customization for banking-specific requirements (e.g., compliance, personalized workflows). Agentforce is viable for faster Time to market but may require significant customization or incur higher costs.

### 4.5. Tech Stack and License

| No. | Product name | Brief description | Paid/Free | License |
| --- | --- | --- | --- | --- |
| **1** | **LangChain** | LangChain is a framework for building LLM-powered applications, chaining together interoperable components and third-party integrations to simplify AI application development, future-proofing decisions as the underlying technology evolves. LangChain helps developers build applications powered by LLMs through a standard interface for models, embeddings, vector stores, and more.<br><br>**Use LangChain for:**<br>• Real-time data augmentation.<br>• Model interoperability. | Free | MIT |
| **3** | **LangFuse** *(self-hosted, open-source)* | LangFuse is an open-source LLM Engineering Platform for Traces, evals, prompt management and metrics to debug and improve your LLM application, that works with any LLM app and model. SDKs provided for Python & JS/TS, native integrations for popular libraries and support for OpenTelemetry.<br><br>**Features:**<br>• Model and framework agnostic<br>• Built for production & scale<br>• Incrementally adoptable, start with one feature and expand to the full platform over time<br>• API-first, all features are available via API for custom integrations<br>• Optionally, Langfuse can be easily self-hosted | Free | MIT |
| **6** | **LangGraph** *(open-source)* | LangGraph is an open-source framework, built on top of LangChain, designed to build complex, stateful, and cyclical workflows for AI agents, particularly those involving large language models (LLMs).<br><br>Balance agent control with agency by gaining control with LangGraph to design agents that reliably handle complex tasks.<br><br>Guide, moderate, and control your agent with human-in-the-loop: prevent agents from veering off course with easy-to-add moderation and quality controls.<br><br>Add human-in-the-loop checks to steer and approve agent actions.<br><br>Build expressive, customizable agent workflows: Design diverse control flows — single, multi-agent, hierarchical — all using one framework. | Free | MIT |
| **8** | **LangFlow** | Langflow provides developers with both a visual authoring experience and built-in API and MCP servers that turn every workflow into a tool that can be integrated into applications built on any framework or stack. Langflow supports all major LLMs, vector databases and a growing library of AI tools.<br>• Visual builder interface<br>• Source code access lets you customize any component using Python.<br>• Interactive playground to immediately test and refine your flows with step-by-step control.<br>• Multi-agent orchestration with conversation management and retrieval.<br>• Deploy as an API or export as JSON for Python apps.<br>• Deploy as an MCP server and turn your flows into tools for MCP clients.<br>• Observability with LangSmith, LangFuse and other integrations.<br>• Enterprise-ready security and scalability. | Free | MIT |
| **9** | **RAG Knowledge Base** | Project will provide available integration to the following knowledge base RAG depending on individual use cases asssessment:<br>• AWS Bedrock Knowledge Base on Open Search<br>• AWS Bedrock Knowledge Base on S3 Vector (new)<br>• Mongo DB Vector DB | Available for enterprise and community edition | AWS Bedrock (existing enterprise license)<br>Mongo DB Community SSPL License (available for enterprise internal use)<br>https://www.mongodb.com/legal/licensing/server-side-public-license |

### 4.6. Target Architecture

> Reference Deployment View **the Detailed Design Document, Deployment View section**

### 4.7. Security Considerations

#### 4.7.1 Data and AI Security

**Data Flow:**

```
Internal User (Prompt + File)
        ↓
Chat WebApp (Authenticated)
        ↓
Trellix DLP Scan (Checkpoint 1)
        ↓
AI Application Backend (Process text / image / document)
        ↓
GuardRail API (Checkpoint 2)
        ↓
Routing + Audit + API
        ↓
LLM (OpenAI, Gemini, LLaMa, Bedrock)
        ↓
Response to User
```

**Security at each checkpoint:**

- **At the endpoint and Network level (McAfee/Trellix)**
  - Use Trellix on workstations to block users from sending sensitive information outside of the bank
  - Protects against data loss from:
    - Clipboard software
    - Cloud applications
    - Email
    - Network shares
    - Printers
    - Screenshots
    - Application file access
    - Web posts
    - Removable storage
    - Local file system files

- **In the system (Guardrail)**
  - Integrate with Guardrail for robust Data Loss Prevention (DLP) capabilities
  - Amazon Bedrock Guardrails provides the following safeguards (also known as policies) to detect and filter harmful content:
    - Content filters
    - Denied topics
    - Word filters
    - Sensitive Information filters / marking
    - Contextual grounding check
    - Prompt validation
  - Block or masking all prompt, file contain sensitive data
  - Detail feature assessment: *internal Guardrail / DLP test report*

- **User portal**
  - Authentication and grant access for only authorized user
  - Prompt/data and file input validation: accept only document and image file type/format. Block all other file type/format
  - Implement separate LLMs to support extracting sensitive information from file and detect data obfuscate, data transform ..
  - Logging and auditing
  - Data protection (encryption, purge prompt history, masking sensitive data)
  - OWASP top 10 security

- **Lang\* security**
  - Tool restriction, restrict flow path, memory management, logging, prompt template, executing constraints, access control
  - Input and output sanitize by Code

#### 4.7.2. System security

- The system is deployed in the corporate landing zone and complies with the bank's security standards.
- Centrally authenticate users with Azure AD using SSO and authorize using IAM SailPoint.
- For Knowledge Base (KB), build an authorization matrix based on functional roles in the bank for data that needs restricted access. For example, EA cannot access ITS's KB and vice versa.
- Authentication for communication between service to service
- Separate IAM Role for each LLM model used on AWS Bedrock.
- The system only allows access within the internal network.
- Require encryption of all data in transit and data at rest.
- Logging and monitoring user action, LLM models processing file.
- Deploy AWS Bedrock on EC2 vs Serverless/Managed Service

#### 4.7.2. Compliance

Compliance with corporate security and data privacy.

| Aspect | Deploy LLM on Amazon EC2 (in the corporate landing zone) | Deploying LLM with Amazon Bedrock (Serverless/Managed Service) |
| --- | --- | --- |
| **Control & Customization** | • **Highest control:** Full control over the operating system, libraries, and deployment environment.<br>• **High customization:** Can install security controls according to corporate standards | • **Limited control:** AWS manages the infrastructure. You only interact via APIs.<br>• **Limited customization:** Depends on models and configurations provided by Bedrock. |
| **Data Privacy** | • **Highest priority on privacy:** Data is processed within the corporate landing zone environment, minimizing the risk of data leaving control.<br>• **High control:** the bank can apply customized security, encryption, and access control policies at the network and system level according to corporate standards | • **High level of privacy:** Bedrock guarantees that your data will not be used to train the underlying models. Data is encrypted in transit and at rest.<br>• **Still via public APIs, though:** Despite AWS PrivateLink, data still travels through AWS services, which can be a concern for extremely sensitive data. *Refer:* https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html |
| **Cost** | Fixed or hourly cost: Pay for EC2 instances that run continuously, even when there are no requests. | Pay as you go (tokens/requests): Pay only for what you use. |
| **Management & Operations** | Comprehensive Management: the bank is responsible for operating systems, patching, maintenance, monitoring, and security. | Minimal management: AWS is responsible for infrastructure, patching, and maintenance. |
| **Implementation time** | Longer: Requires EC2 environment setup, configuration, and optimization. | Faster: Simply integrate via API. Can start using immediately. |

> **Propose for endorsement:**
> - Using Amazon BedRock Serverless endpoint for available models (already in use in an existing internal chatbot and several internal use cases)
> - Using Amazon BedRock Marketplace deployment to Sage Maker EC2 for non-available models, which fulfill required niche.

---

## 5. Appendix

### 5.1. Use cases and Functional Requirements

**Overview:**

- **Use Case 1: Automate Post-Call Workflows**
  - Automate repetitive post-call tasks (e.g., CRM updates, follow-up scheduling) to allow RMs to focus on client relationships.
  - Automate action item identification and scheduling for faster client engagement.
- **Use Case 2: Customer 360 Assistant**
  - Automate consolidate all 360 customer information
  - Provide sentiment analysis, insights, lead recommendation, personalized engagement scripts and product positioning.
- **Use Case 3: BA GenAI Assistant**
  - Automated Data Extraction from existing documents (BRD document, Confluence page, existing user stories, etc)
  - Summarizing large, complex business documents
  - Generate BA documentation

**Problems & Needs which can be solved by Agentic AI (Consolidated from the use cases above):**

- **Inefficient Human Dependency**, reducing productivity and scalability: Manual tasks (e.g., CRM updates, report generation) persist
- **Limited Adaptability:** Rule-based systems cannot adjust workflows dynamically, leading to missed opportunities in customer engagement and sales.
- **Lack of personalization:** Lack of real-time, context-aware responses results in generic engagement, lowering conversion rates and satisfaction.
- The need to have **specialized tools to handle NPL and unstructured data** (e.g., call transcripts, customer sentiment).
- The need to use **internal Data and Insights** as an input for next steps in the workflow: Traditional systems need predefined, rule-based integrations, delaying insights
- **High Development cost:** Frequent updates to rules and integrations are needed to accommodate new use cases, increasing costs and complexity

#### Use Case 1: Automate Post-Call Workflows

By integrating Agentic AI Platform into banking operations, the bank aims to improve and boost the customer journey and elevate sales capabilities through hyper-personalized engagement, streamlined onboarding, and intelligent automation.

| No. | Category of Business Pain Points or Needs | Business Pain Point | Functional Requirement |
| --- | --- | --- | --- |
| 1 | Inefficient Human Dependency, reducing productivity and scalability; The need to have specialized tools to handle NPL and unstructured data | Manual extraction of customer information from call transcripts is slow and error-prone. | Automated voice to text post-call meeting |
| 2 | | Excessive time spent summarizing key discussion points. | Rapid generation of concise summaries. |
| 3 | Inefficient Human Dependency, reducing productivity and scalability; Limited Adaptability | • Manual adjusting based on context.<br>• Manual drafting and sending of summary emails delays communication. | • Context-dependent process: able to adjust workflows dynamically (define when to send email and draft email content)<br>• Integrate with email systems to send summaries |
| 4 | | • Manual adjusting based on context.<br>• Manual updates to the CRM compromise data quality + reduce productivity. | • Context-dependent process: able to adjust workflows dynamically (define when and what to create/update in CRM)<br>• Create/Update in CRM with structured data. |
| 5 | | • Manual adjusting based on context.<br>• Lack of reminders and follow-ups reduce engagement opportunities | • Context-dependent process: able to adjust workflows dynamically (define follow-up actions when needed)<br>• Provide reminders on follow-up actions |

**Expected Business Benefits:** The technology enables real-time understanding of customer needs, enhances cross-selling and upselling opportunities, and boosts conversion rates by guiding sales teams with data-driven insights. Additionally, Agentic AI supports customer retention through proactive interventions and empowers staff productivity by automating routine tasks and providing contextual recommendations—ultimately driving growth, efficiency, and customer satisfaction as following benefits:

- **Increase RM Productivity:** Automate repetitive post-call tasks (e.g., CRM updates, follow-up scheduling) to allow RMs to focus on client relationships.
- **Reduce Follow-Up Delays:** Automate action item identification and scheduling for faster client engagement.
- **Increase customer satisfaction:** Concise and real time customized & personalized recommendations

#### Use Case 2: Customer 360 Assistant

> Full use case: Use case 2 - Customer 360 Assistant

In the following table, we only list down Functional Requirements which have potential to be solved by Agentic AI (Filter by Requirement Group = "Công nghệ thông minh"). In this use case, AI agents are expected to Automate consolidate all 360 customer information, provide sentiment analysis, insights, lead recommendation, personalized engagement scripts and product positioning.

| No. | Category of Business Pain Points or Needs | Business Pain Point | Functional Requirement |
| --- | --- | --- | --- |
| 1 | The need to have specialized tools to handle unstructured data | Manual processing of unstructured documents is slow and error-prone. | Extract data from unstructured documents. |
| 2 | The need to use internal Data and Insights as an input for next steps in the workflow | Siloed data delays insights from transactional, behavioral, and demographic sources. Traditional systems need predefined, rule-based integrations, delaying insights | Provide insights from transactional, behavioral, and demographic sources. |
| 3 | | | Perform sentiment analysis |
| 4 | Inefficient Human Dependency, reducing productivity and scalability | • Manual adjusting based on context.<br>• Manual lead identification misses opportunities. | • Context-dependent process: able to adjust workflows dynamically (define leads when needed)<br>• Automated and accurate lead recommendation based on need-fit criteria (right needs, right timing, right audience, financial readiness) |
| 5 | | • Manual adjusting based on context.<br>• Manual updates to the CRM compromise data quality + reduce productivity | • Context-dependent process: able to adjust workflows dynamically (when and what to create/update in CRM)<br>• Offer a user interface with a "Create Lead" button. |
| 6 | Lack of personalization | • Generic engagement scripts reduce conversion rates. | Provide personalized recommendation for engagement scripts and product positioning (need-based selling). |

**Expected Business Benefits:**

- **Increase RM Productivity:** Automate consolidate all 360 customer information instead of manually collection
- **Increase customer satisfaction:** Concise and real time customized & personalised recommendations regarding financial plannings, investment...

#### Use Case 3: BA GenAI Assistant

AI agents revolutionize the role of business analysts/product owners by automating mundane tasks, providing rapid insights, and enhancing decision-making processes.

| No. | Category of Business Pain Points | Business Pain Point | Functional Requirement |
| --- | --- | --- | --- |
| 1 | Inefficient Human Dependency, reducing productivity and scalability; Limited Adaptability | Manual Data Extraction | **Automated Data Extraction:** Extract data from existing documents (BRD document, Confluence page, existing user stories, etc) using intelligent document recognition (IDR) and advanced knowledge analysis. |
| 2 | | Manual Report and Use case Generation, Delayed Insights and Follow-Ups | **Efficient Report and Use case Generation:** Summarizing large, complex business documents like loan agreements, distilling key terms, interest rates, and repayment conditions. Generate Report and Use cases. |
| 3 | | Manual Routine Tasks | **Automation of Routine Tasks:** Generate BA documentation based on existing summary and knowledge |

**Expected Business Benefits:**

### 5.2. Non-Functional Requirements

#### Non-Functional Requirement 1: Secured LLM model exploration and integration

Using model hosted inside the corporate landing zone, and third party model endpoints *(Which is scope of the AI Gateway project)*.

the bank has multiple AI/Agentic AI POC/ Internal tools for selected use-cases, but not able to scale-up due to the lack of comprehensive solution with foundational components (including Security control).

**Current limitations:**

- Limited support for selected use cases.
- Lack of full control over inputs/outputs from the bank.

Although the bank is awared of the importance and opportunities to adopt AI/Agentic AI, the lack of foundational components (i.e. Data Protection, Data Poisoning Prevention, Data Loss Prevention, Model Guardrails, prompt validation, prompt logging and audit) becomes road-blockers, requiring the bank to act decisively to mitigate them.

| No. | Targeted Capability | Non-Functional Requirement |
| --- | --- | --- |
| 1 | **Data Loss Prevention (DLP):** Security controls to detect and prevent unauthorized transmission, exfiltration, or exposure of sensitive information. | Reuse and enhance user endpoint device DLP using existing solution Trellix DLP |
| 2 | **AI Guardrails:** Predefined rules and filters to protect LLM applications from data leakage, algorithmic bias, hallucinations, and malicious inputs. These ensure safe and responsible deployment. This includes PII masking, granular access controls, real-time monitoring. | Configure and deploy AI GuardRails solution to perform content filter, PII masking/filtering, preventing banned topics, context assessment and truth grounding of various models and use cases. |
| 3 | **Prompt Auditing, Logging, and Monitoring:** Comprehensive tracking and analysis of all prompts and responses to ensure compliance, detect anomalies, and provide an auditable trail of AI interactions. | Flexible UI and LLM gateway to provide an comprehensive and unified Human Interface, System Interface to integrate with different AI model endpoints |
| 4 | **Rapidly build and deploy agents**, reducing time-to-market and technical barriers. | No-Code/Low-Code Agent Builder |
| 5 | **Model selections** (Access to both self-built models and 3rd party models) | |

#### Non-Functional Requirement 2: Number of users and transactions

**Use Case 1: Automate Post-Call Workflows and Use Case 2: Customer 360 Assistant**

The users of this use case include PRMs across the bank. The following data is provided by the business team (Workstream 1):

- The number of users is **800**. The business team expects that the number of users will not grow over the years
- The number of concurrent users (peak time) is **800**
- The number of transactions (maximum) is:
  - Use case 1: **6 calls/user/day**
  - Use case 2: **4 meetings/user/day**

| User | 2025 | 2026F | 2027F | 2028F |
| --- | --- | --- | --- | --- |
| System users | 800 | 800 | 800 | 800 |
| Concurrent user | 800 | 800 | 800 | 800 |
| Calls | 1,752,000 | 1,752,000 | 1,752,000 | 1,752,000 |
| Meetings | 1,168,000 | 1,168,000 | 1,168,000 | 1,168,000 |

**Use Case 3: BA GenAI Assistant**

The party providing the business requirements is COC, ROC, CBT, RBT.... The user of this use case is business analyst/product owners across the bank. The following data is provided by DEV team (TBC with COC, ROC, CBT, RBT....).

| User | 2025 | 2026F | 2027F | 2028F |
| --- | --- | --- | --- | --- |
| System users | 50 | 60 | 70 | 80 |
| Concurrent user | 25 | 30 | 35 | 40 |
| Transaction | 500 transactions/day | 600 transactions/month | 700 transactions/month | 800 transactions/mont |

### 5.3. Popular Agentic AI Patterns

**Agentic AI Patterns:**

- **Planning Agent:** Agents autonomously devise and execute multi-step plans to achieve complex objectives. *Example:* Task management systems organizing and prioritizing tasks based on user goals.
- **Reflective Agent:** Agents that iteratively evaluate and critique their own outputs to enhance performance.
- **Task-Oriented Agent:** Agents designed to handle specific tasks with clear objectives. *Example:* appointment scheduling.
- **Hierarchical Agent:** Agents are organized in a hierarchy, managing multi-step workflows or distributed control systems, where higher-level agents oversee task delegation.
- **Coordinating Agent:** Agents facilitate collaboration and coordination and tracking, ensuring efficient execution. *Example:* a coordinating agent assigns subtasks to specialized agents.
- **Context-Aware Agent:** Agents dynamically adjust their behavior and decision-making based on the context in which they operate.
- **Self-Learning and Adaptive Agents:** Agents adapt through continuous learning from interactions and feedback: adapt to user interactions over time, learning from feedback and adjusting responses to better align with user preferences and evolving needs.
- **Human-in-the-Loop Collaboration:** Agents operate semi-autonomously with human oversight. *Example:* Sales assisted tools that provide recommendations but allow RM to make final decisions.
- **RAG-Based Agent:** This pattern involves the use of Retrieval Augmented Generation (RAG), where AI agents utilize knowledge sources (corporate systems or external data sources i.e. web browsing) dynamically to enhance their decision-making and response.

**Sources:**

- Ken Huang's CSA blog an agentic patterns at https://cloudsecurityalliance.org/blog/2024/12/09/fromai-agents-to-multiagent-systems-a-capability-framework
- Andrew Ng's articles on the Batch on Agentic Design patterns https://www.deeplearning.ai/thebatch/how-agents-can-improve-llm-performance
- Building effective agents by Anthropic team http://anthropic.com/research/building-effectiveagents
- Agents by Chip Huyen https://huyenchip.com/2025/01/07/agents.html

### 5.4. Example of Human-AI Collaboration flow

**Use Case 1: Automate Post-Call Workflows**

> *Notes: The detailed design (including Agent-Orchestration, Agent-Service, Agent-Agent, Orchestration-UI interactions) will be finalized in the Detailed Design Document.*

### 5.5. Example of data flow in Agentic AI

> *Notes: The detailed design will be finalized in the Detailed Design Document.*

### 5.6. Agentforce vs LangChain comparison

> **Appendix:** Agentforce vs LangChain
