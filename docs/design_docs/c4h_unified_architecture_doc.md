**C4H Unified Architecture Document (Version 3.0)**

Date: May 7, 2025  
Status: Adopted  
Purpose: This document outlines the definitive architecture for the C4H (Coder for Hire) system. It consolidates and refines previous architectural concepts, design principles, and operational models into a single, authoritative reference. This document supersedes all prior C4H architecture documents.

## **1\. Introduction and Vision**

### **1.1. Purpose of C4H**

The C4H system is designed to automate complex software development and code management tasks by leveraging Large Language Models (LLMs) and other programmatic tools within a robust, predictable, and auditable framework. Its primary goal is to enable the creation of reliable, intent-driven AI systems capable of understanding requirements, designing solutions, implementing changes, and performing associated engineering tasks with a high degree of consistency and traceability.

### **1.2. Core Philosophy: The Declarative, Recursive Orchestration Methodology**

C4H is built upon a **declarative methodology** where workflows are defined primarily through configuration rather than imperative code. This separates the *process logic* (the "what") from the *execution mechanics* (the "how").

The core tenets of this methodology include:

* **Configuration-Driven Behavior:** All system actions, from high-level workflow sequencing to individual agent behavior and skill execution, are driven by declarative configurations.  
* **Hierarchical & Recursive Orchestration:** Complex processes are decomposed into nested, orchestrated units. The same fundamental orchestration pattern, defined by an **execution\_plan**, is applied recursively at various levels:  
  * Orchestrating sequences of "Teams" (macro-workflows).  
  * Orchestrating sequences of "Agent Instances" within a Team.  
  * Orchestrating sequences of "Skills" or LLM calls within an Agent Instance (if that agent's persona dictates an orchestrated behavior).  
* **Statelessness and Immutability:** Agents are stateless, and they operate on immutable configuration snapshots and contexts to ensure predictability and reproducibility.  
* **Clear Separation of Concerns:** The system distinguishes between:  
  * Generic **Agent Types** that provide fundamental execution capabilities.  
  * **Personas** that configure and specialize these generic types for specific roles.  
  * Reusable **Skills** that encapsulate deterministic, often non-LLM, operations.  
* **Underlying Execution Engine:** Prefect is utilized as the underlying engine for executing the defined workflows, managing tasks, dependencies, and state.

This approach aims to achieve process-level determinism, even when incorporating non-deterministic components like LLMs, by providing a highly structured, observable, and controllable execution environment.

## **2\. Core Architectural Principles**

The C4H architecture adheres to the following fundamental principles, largely derived and refined from "Agent Design Principles v2.md" (Source 2511-2570) and "Config Design Principles.md" (Source 3074-3080):

* **2.1. Declarative Definition (YAML First for Orchestration):**  
  * All orchestration logic (sequences, parallelism, conditions, data flow) is defined declaratively using a consistent YAML structure, primarily the execution\_plan.  
* **2.2. Stateless Agents & Immutable Context (Principles 1.3, 2.6 from Agent Design):**  
  * Agent instances do not maintain state between invocations.  
  * Each execution is driven by the provided context and an immutable Effective Configuration Snapshot.  
  * Context is treated as immutable by agents; they return updates, and the "Execution Plan Executor" (see section 3.4) constructs the next context.  
* **2.3. Immutable Configuration Snapshots (Principle 2.7 from Agent Design):**  
  * Before any run, all relevant configuration fragments are merged into a single, immutable Effective Configuration Snapshot. All components use this snapshot for the duration of the run.  
* **2.4. Explicit Data Flow & Clear Interfaces (Principle 2.3 from Agent Design):**  
  * Data flows explicitly between orchestrated units (Teams, Agent Instances, Skills) via the managed context and structured return objects (e.g., AgentResponse, SkillResult).  
  * Inputs and outputs adhere to defined contracts.  
* **2.5. Configuration-Driven Behavior (Principle 1.1 from Agent Design):**  
  * The behavior of any component is dictated by its configuration, as loaded from the Effective Configuration Snapshot. Agent logic is lean and focused on interpreting this configuration.  
* **2.6. Separation of Concerns:**  
  * **Generic Agent Types:** Provide basic execution patterns (LLM interaction, Skill execution, plan execution).  
  * **Personas:** Specialize generic agents for specific roles and tasks, providing prompts, parameters, or sub-execution plans.  
  * **Skills:** Encapsulate discrete, reusable functionalities.  
* **2.7. Reproducibility and Auditability:**  
  * The combination of immutable snapshots, stateless execution, explicit data flow, and comprehensive logging (including lineage events) aims for high reproducibility (at the process level) and full auditability.  
* **2.8. Total Functions (Principle 1.4 from Agent Design):**  
  * Agents and Skills should return predictable, structured responses for both success and known failure conditions, avoiding exceptions for flow control.

## **3\. Key Architectural Components**

### **3.1. The execution\_plan: A Universal DAG Language**

The execution\_plan is the cornerstone of C4H orchestration. It is a YAML-defined structure that declaratively specifies a Directed Acyclic Graph (DAG) of operations.

* **Structure:**  
  * enabled: (boolean) Activates or deactivates the plan.  
  * steps: (list) An ordered list of step objects, defining the nodes and flow of the DAG.  
* **Step Definition:** Each step includes:  
  * name: (string) Unique identifier for the step.  
  * type: (string) Type of operation (e.g., "skill\_call", "agent\_call", "team\_call", "llm\_call", "loop", "branch", "set\_value"). This determines the nature of the node or action.  
  * node / target\_team / skill: (string) Identifier for the component to be executed (Skill name, Agent Instance name, Team name).  
  * params / input\_params: (object) Input parameters for the node, supporting context variable templating (e.g., {{context.path.to.value}}).  
  * output\_field: (string, optional) Dot-notation path in the context where the step's result is stored.  
  * rules: (list, optional) Routing rules to determine the next step(s) based on conditions.  
    * condition: A condition object evaluated against the current context.  
    * route\_to\_step / action: Specifies the next step name or a control action (e.g., exit\_plan\_with\_success, break\_loop).  
  * condition: (object, optional) A precondition for executing the step.  
  * Other type-specific fields (e.g., iterate\_on, loop\_variable, body for loops; branches for conditional branching).  
* **Recursive Application:** This execution\_plan syntax is used consistently at all levels of orchestration (see section 4).  
* **BNF Reference:** (A detailed BNF for the execution\_plan should be maintained, similar to the one discussed previously, and referenced here).

### **3.2. Configuration Model & Management**

The system relies on a hierarchical and mergeable configuration system, culminating in an immutable Effective Configuration Snapshot for each run.

* **Effective Configuration Snapshot:** All configuration sources (system defaults, environment-specific, job-specific, persona definitions, team definitions) are merged into a single, read-only YAML file at the start of a workflow. This snapshot is the sole source of truth for configuration during that run.  
* **Conceptual teams\_config (Centralized Definitions):** While physically spread across files (e.g., system\_config.yml, persona files, skill registration), the following components are conceptually managed:  
  * **agent\_types:** An enumeration of the available generic agent classes:  
    * GenericLLMAgent: For pure LLM interactions. Its behavior is defined by its persona's prompts and LLM parameters. *It should not contain special handling for specific tools or scripts like tartxt.py.*  
    * GenericSkillAgent: For direct, non-LLM execution of a single registered skill. Its persona specifies the skill and its parameters.  
    * GenericFallbackAgent: A specialized GenericLLMAgent with conservative settings for error handling.  
    * *(The concept of GenericOrchestratorAgent as a distinct type is superseded by agents whose personas contain an execution\_plan)*.  
  * **skills:** A registry or definitions of reusable, typically deterministic, functional units (e.g., tartxt\_runner, semantic\_iterator, asset\_manager, mcp\_endpoint). Each skill has a clear interface and purpose.  
  * **personas:** Named configurations that specialize a generic agent type for a particular role or task.  
    * A persona explicitly states the agent\_type it configures.  
    * If for GenericLLMAgent: Contains provider, model, temperature, prompts, etc.  
    * If for GenericSkillAgent: Contains skill (name of skill to run) and skill\_params.  
    * **If a persona is intended for an agent that orchestrates multiple sub-steps (previously the role of GenericOrchestratorAgent): It directly contains an execution\_plan section.** The agent\_type might be implicit or a general "OrchestratingAgent" if needed, but the presence of the execution\_plan is key.

### **3.3. Execution Units (Wrapped as Prefect Nodes)**

All active components in C4H are designed to be wrapped as Prefect tasks or flows, enabling robust execution and observability.

* **Skills:** Each C4H Skill (e.g., tartxt\_runner, semantic\_iterator) is executed as a Prefect @task.  
* **Agent Instances (Generic Agent \+ Persona):**  
  * The process(context) method of any C4H Agent Instance is executed as a Prefect @task.  
  * If the agent's persona leads to a direct LLM call (GenericLLMAgent) or a direct skill call (GenericSkillAgent), the Prefect task executes that singular action.  
  * If the agent's persona contains an execution\_plan, the Prefect task for that agent instance involves invoking the "Execution Plan Executor" (see below) with that plan.  
* **Teams (Self-Contained Orchestrated Workflows):**  
  * A C4H "Team" (as defined in teams\_config.teams, with its own internal execution\_plan orchestrating its agent instances) is executed as a Prefect @flow (a sub-flow).

### **3.4. The "Execution Plan Executor" (Unified Orchestration Engine)**

This is the core underlying mechanism responsible for interpreting and running any execution\_plan definition.

* **Functionality:** It takes an execution\_plan and the current context as input. It iterates through the plan's steps, evaluates conditions, executes the specified nodes (Skills, Agent Instances, or other Teams/sub-plans by recursive invocation), manages data flow via context updates (based on output\_field and input params), and handles routing.  
* **Implementation:** This logic is implemented in Python and leverages Prefect's capabilities for executing the underlying nodes (which are Prefect tasks/flows). The code previously in GenericOrchestratorAgent.process() and parts of c4h\_services.src.orchestration.orchestrator.Orchestrator would be consolidated or refactored into this unified executor pattern.  
* **Universality:** The same executor logic is used regardless of whether the execution\_plan is for a top-level workflow, a team's internal agent sequence, or an agent's internal skill sequence.

### **3.5. Context Management**

* Context is a dictionary passed through the workflow.  
* It is treated as immutable by all execution units (Skills, Agent Instances, Teams).  
* The "Execution Plan Executor" is responsible for creating new context states for subsequent steps by taking the current context and applying updates returned from completed steps.  
* Standardized keys within the context (e.g., input\_data, results, config\_snapshot\_path, workflow\_run\_id) facilitate interoperability.

### **3.6. Lineage and Logging**

* Comprehensive structured logging is implemented throughout the system.  
* An EventLogger captures key orchestration events (workflow start/end, step start/end, routing decisions, errors) for auditability and traceability (as detailed in c4h\_agents/lineage/event\_logger.py).

## **4\. Orchestration Model: Hierarchical & Recursive**

The C4H system employs a hierarchical and recursive orchestration model built upon the consistent use of execution\_plans.

* **Top-Level Workflow (e.g., development\_shop):**  
  * Defined by an execution\_plan where steps primarily involve type: "team\_call" and target\_team pointing to Team definitions.  
  * Manages the gross sequence of major business functions (e.g., Design \-\> Develop \-\> Review \-\> Assure).  
  * *Example:* development\_shop team orchestrates designers\_team, developers\_team, reviewers\_team, and assurance\_team.  
* **Team-Level Orchestration (e.g., developers\_team):**  
  * Each Team (like developers\_team) is defined with its own list of constituent agents (Agent Instances, each with a name and persona) and an internal execution\_plan.  
  * This plan's steps use type: "agent\_call" (or similar) with node pointing to one of its named Agent Instances.  
  * It manages the sequence of agent activities required for that team to fulfill its function (e.g., developers\_team orchestrates discover\_instance \-\> design\_solution\_instance \-\> implement\_solution\_instance).  
* **Agent-Level Orchestration (via Orchestrator Personas):**  
  * An Agent Instance (e.g., implement\_solution\_instance used by developers\_team) whose assigned persona (e.g., code\_implementer\_persona) contains an execution\_plan.  
  * This plan's steps primarily use type: "skill\_call" (with skill and params) or type: "llm\_call".  
  * It manages the sequence of atomic skills or LLM interactions needed for that specific agent to perform its complex task (e.g., the Coder agent iterating through changes and applying them using semantic\_iterator and asset\_manager skills).

**Data Flow in Recursion:**

* Context is passed down to invoked sub-plans (for Teams or orchestrating Agents).  
* Outputs from sub-plans are mapped back to the calling plan's context via output\_field specifications, enabling data to flow up and influence subsequent steps in the parent plan.

## **5\. Benefits of the Unified Architecture**

* **Conceptual Simplicity & Consistency:** A single, powerful execution\_plan paradigm for defining all levels of orchestration, making the system easier to understand, learn, and maintain, despite its capacity for complexity.  
* **Modularity & Reusability:**  
  * Skills are atomic and reusable.  
  * Personas define reusable agent specializations. Orchestrator personas define reusable sub-workflows of skills.  
  * Teams become reusable, complex sub-workflows that can be composed into larger applications.  
* **Scalability & Flexibility:** New capabilities can be added by defining new skills, personas, or teams, or by composing existing ones in new execution\_plans without extensive code changes to the core engine.  
* **Testability:** Individual Skills, Agent-Persona combinations, and Team-level execution\_plans can be tested in isolation.  
* **Clarity of Agent Roles:**  
  * GenericLLMAgent \+ Persona: Purely for LLM interaction based on persona prompts.  
  * GenericSkillAgent \+ Persona: Purely for single, deterministic skill execution based on persona parameters.  
  * Any Agent \+ Persona with an execution\_plan: This agent *is* an orchestrator for that plan.  
* **Enhanced Traceability:** The recursive execution of named plans, coupled with lineage logging, provides clear insight into how results are achieved.

## **6\. Conclusion**

The C4H Unified Architecture (Version 3.0) establishes a powerful, declarative, and recursive framework for building complex, AI-driven systems. By standardizing on the execution\_plan as the universal language for orchestration at all levels—from global workflows down to intra-agent skill sequences—and by clearly defining the roles of generic agent types, personas, and skills, the system aims to provide a robust, consistent, and understandable platform. This architecture, executed by an underlying "Execution Plan Executor" (leveraging Prefect), is designed to meet the demands of auditable and reproducible AI system development.

---

This document forms the new baseline for C4H architectural understanding and future development.
