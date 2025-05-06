**C4H Agent System: Requirements and Design Document**

Version: 3.1 (Proposed)  
Date: Sunday, May 4, 2025 (Sydney Time)  
Status: Proposed  
**Table of Contents:**

1. [Introduction](https://www.google.com/search?q=%231-introduction)  
   * 1.1. Purpose  
   * 1.2. Scope  
   * 1.3. Goals  
   * 1.4. Core Concepts Overview  
   * 1.5. Reference Documents  
2. [Requirements](https://www.google.com/search?q=%232-requirements)  
   * 2.1. Functional Requirements  
   * 2.2. Architectural Requirements  
   * 2.3. Non-Functional Requirements  
3. [Architecture & Design](https://www.google.com/search?q=%233-architecture--design)  
   * 3.1. Overview  
   * 3.2. Core Components  
   * 3.3. Key Concepts  
   * 3.4. Data Flow (Event Model & Context)  
   * 3.5. Control Flow (Orchestration)  
4. [Agent Implementation (TaskAgent)](https://www.google.com/search?q=%234-agent-implementation-taskagent)  
   * 4.1. Responsibilities  
   * 4.2. Configuration & Hydration  
   * 4.3. Dynamic Configuration Selection  
5. [Skill Implementation](https://www.google.com/search?q=%235-skill-implementation)  
   * 5.1. Skill Definition & Contract  
   * 5.2. Skill Interface & Registry (Requirement Detail)  
   * 5.3. Example Core Skills  
6. [Orchestration Implementation](https://www.google.com/search?q=%236-orchestration-implementation)  
   * 6.1. Orchestrator Engine Responsibilities  
   * 6.2. Declarative DSL Configuration (Detailed)  
   * 6.3. Context Management Conventions (Requirement Detail)  
7. [Configuration Management](https://www.google.com/search?q=%237-configuration-management)  
   * 7.1. Effective Configuration Snapshots  
   * 7.2. Merging & Validation  
   * 7.3. Persona Injection  
8. [Event Sourcing / Lineage](https://www.google.com/search?q=%238-event-sourcing--lineage)  
   * 8.1. Event Logger Service  
   * 8.2. Event Trigger Points & Structure (Detailed)  
9. [Example Workflow: Coder Team](https://www.google.com/search?q=%239-example-workflow-coder-team)  
10. [Migration Path (Conceptual)](https://www.google.com/search?q=%2310-migration-path-conceptual)

## ---

**1\. Introduction**

### **1.1. Purpose**

This document defines the requirements and design for the refactored C4H Agent System. It mandates a transition to a unified, flexible, event-driven architecture based on stateless agents, immutable configuration snapshots, and declarative orchestration.

### **1.2. Scope**

This document covers the design of the core agent execution framework, the orchestration engine, configuration management, event sourcing/lineage, agent/skill implementation patterns, and the required orchestration DSL capabilities. It replaces previous agent class-specific designs.

### **1.3. Goals**

* **Eliminate Legacy Model:** Mandate the factory/snapshot pattern, removing agent\_class support.  
* **Simplify Agent Implementation:** Define minimal, reusable agent types whose behavior is configuration-driven \[User Requirement\].  
* **Declarative Workflows:** Define complex workflow logic, including control flow, declaratively in configuration.  
* **Improve Robustness & Reproducibility:** Enforce immutability for configuration and context updates; implement total functions for predictable error handling \[User Requirement\]. Ensure runs are reproducible via effective configuration snapshots.  
* **Enhance Orchestration:** Support complex patterns like conditional branching, loops, fan-out, and concentration/fan-in.  
* **Comprehensive Observability:** Implement granular event sourcing/lineage triggered by the orchestrator \[User Requirement\].

### **1.4. Core Concepts Overview**

The system operates on an **event-driven** model where **stateless agents** (primarily TaskAgent) process input events (composed of data\_context and execution\_metadata) based on **immutable configuration snapshots**. The workflow is defined declaratively and executed by a central **Orchestrator**, which manages state transitions immutably and applies **controls** (routing, loops, concentration) between agent steps. Complex logic is encapsulated in reusable **Skills**. **Total functions** and structured responses (AgentResponse) ensure predictable error handling. Granular **event sourcing**, managed by the orchestrator, provides observability.

### **1.5. Reference Documents**

* docs/c4h\_orchestration\_enhancements\_0430.md (Basis for factory/snapshot/DSL requirements)  
* docs/proposed\_agent\_architecture.md (Introduced type-based agents)  
* docs/Agent\_Design\_Principles\_v2.1.md (Incorporates immutability/totality)  
* docs/Config\_Design\_Principles.md  
* Other C4H design guides (Work Order, API, etc.)

## **2\. Requirements**

### **2.1. Functional Requirements**

* **FR1:** The system must execute workflows defined declaratively in YAML configuration.  
* **FR2:** The system must support sequential execution of agent tasks within a team/stage.  
* **FR3:** The system must support conditional routing between teams/stages based on the results (AgentResponse) of the preceding step. The DSL must support operators like equals, contains, exists, etc., evaluating task status and data/context fields.  
* **FR4:** The system must support looping constructs (type: "loop") within the orchestration DSL for iterating over lists in the context \[User Requirement\].  
* **FR5:** The system must support fan-out orchestration (specifying a list of next\_team IDs in routing or entry\_team for parallel execution) \[User Requirement\].  
* **FR6:** The system must support fan-in/concentration (type: "concentrator") based on configurable completion conditions (count, timeout, signal) and aggregation strategies (list, merge, custom) \[User Requirement\].  
* **FR7:** The system must provide a mechanism to execute specific "Skills" (defined Python functions/classes) as distinct workflow steps via TaskAgent configuration.  
* **FR8:** The system must support invoking LLMs as a specific workflow step via TaskAgent configuration.  
* **FR9:** The system must support invoking external scripts (e.g., tartxt.py) as a specific workflow step via TaskAgent configuration (likely using a dedicated Skill).  
* **FR10:** The system must implement granular event sourcing via a central EventLogger, logging key orchestration and execution events \[User Requirement\].  
* **FR11:** The system must provide mechanisms to handle failures, including routing to fallback agents/teams based on AgentResponse status.  
* **FR12:** The system must support dynamic, context-aware configuration selection for agent tasks at runtime (e.g., selecting a persona based on execution\_metadata or data\_context). *(New)*  
* **FR13:** The system must define and adhere to clear conventions for managing workflow data context structure, propagation, and merging, especially in parallel and looping scenarios. *(New)*  
* **FR14:** A formal interface or contract for reusable Skills, along with a registration/discovery mechanism, must be defined. *(New)*

### **2.2. Architectural Requirements**

* **AR1:** Agent instantiation *must* use a Factory pattern based on agent\_type and unique\_name specified in configuration. Legacy agent\_class instantiation must be removed.  
* **AR2:** Agent execution *must* be based solely on an immutable Effective Configuration Snapshot generated per run.  
* **AR3:** Agents *must* be stateless regarding workflow execution state between invocations.  
* **AR4:** Workflow context/state updates *must* be handled immutably by the orchestrator \[Principle 1.3\].  
* **AR5:** Agents and Skills *must* implement total functions, returning structured success/failure responses (AgentResponse, SkillResult) instead of raising exceptions for expected errors \[Principle 1.4\].  
* **AR6:** Core agent logic should be minimal; complex operations must be delegated to reusable Skills or LLMs \[Principle 1.2, 2.2\].  
* **AR7:** Event sourcing/lineage recording must be driven by the orchestrator, not individual agents \[User Requirement\].  
* **AR8:** The system must primarily use a minimal set of generic agent types (primarily TaskAgent, optionally OrchestratorAgent).  
* **AR9:** Configuration fragments must be validated against schemas before merging.

### **2.3. Non-Functional Requirements**

* **NFR1:** **Reproducibility:** Workflow runs must be reproducible given the same input context and effective configuration snapshot.  
* **NFR2:** **Observability:** Provide detailed structured logging and granular event sourcing for debugging and tracing.  
* **NFR3:** **Maintainability:** Minimize agent code complexity; workflow logic should primarily reside in declarative configuration.  
* **NFR4:** **Extensibility:** Easily add new Skills or define new workflows via configuration.  
* **NFR5:** **Recursion Safety:** Implement configurable limits (global max\_total\_teams, per-team max\_recursion\_depth) to prevent infinite loops/recursion.

## **3\. Architecture & Design**

### **3.1. Overview**

The architecture is event-driven and declarative. A central Orchestrator Engine executes workflows defined in YAML. Workflows consist of steps, each typically executed by a stateless TaskAgent. Agents perform actions (LLM calls, Skill execution, Script execution) based on their configuration loaded from an immutable snapshot specific to the run. The Orchestrator manages the flow, routing events (composed of data\_context and execution\_metadata) between agents based on declarative rules, and handling control structures like loops, fan-out, and concentration. Event sourcing is integrated at the orchestration level.

\!\[Conceptual Diagram Placeholder: Orchestrator \-\> Reads Config Snapshot \-\> Dispatches Event(DataContext+ExecMeta) \-\> TaskAgent \-\> Executes Configured Action(Skill/LLM/Script) \-\> Returns AgentResponse \-\> Orchestrator \-\> Evaluates Routing/Controls \-\> Dispatches Next Event\]

### **3.2. Core Components**

* **Orchestrator Engine:** (e.g., Prefect flow run\_declarative\_workflow) Reads YAML config snapshot, manages workflow state (data\_context, execution\_metadata) immutably, executes steps via Agent Factory, handles declarative controls (routing, loops, concentration, fan-out), triggers event logging.  
* **Agent Factory:** Instantiates agent objects (TaskAgent), providing them with the effective configuration snapshot and their unique name.  
* **TaskAgent:** The primary generic agent type. Stateless executor that calls a configured function (LLM, Skill, Script) based on input event and config snapshot. Inherits from BaseAgent.  
* **OrchestratorAgent (Optional):** Secondary generic agent type for complex *internal* sub-workflows within a single logical step. Interprets an execution\_plan from its config. Inherits from BaseAgent.  
* **Skills:** Encapsulated, reusable Python functions or classes implementing specific logic (e.g., ChangeExtractorSkill, AssetManagerSkill, TartxtRunnerSkill). Adhere to a defined Skill contract.  
* **Configuration:** YAML files (system\_config.yml, personas) defining agents, skills, orchestration, prompts, etc. Merged into an immutable Effective Configuration Snapshot per run.  
* **Event Logger / Lineage Service:** Central service called by the Orchestrator to record workflow events to persistent storage (e.g., files per run).

### **3.3. Key Concepts**

* **Stateless Agents:** Agents hold no run-specific state between process calls.  
* **Immutability:** Config snapshots are immutable per run. Context updates create new context objects.  
* **Totality:** Functions return predictable AgentResponse or SkillResult objects, handling expected errors internally.  
* **Declarative Control:** Workflow logic (sequence, routing, loops, concentration, fan-out) is defined in YAML.  
* **Event-Driven:** Agent execution is triggered by the arrival of an event dispatched by the orchestrator. The event comprises the data\_context and execution\_metadata.  
* **Snapshot-Based Execution:** All runtime behavior is dictated by the immutable config snapshot.

### **3.4. Data Flow (Event Model & Context)**

1. Orchestrator starts with an initial data\_context and creates initial execution\_metadata.  
2. For Step N, Orchestrator calls Agent N's process method, passing the *current* data\_context, the *current* execution\_metadata (containing sequence N, prior event ID, etc.), and providing access to the immutable config\_snapshot (typically via self.config within the agent).  
3. Agent N processes data\_context based on config\_snapshot and potentially uses execution\_metadata for logging or context-aware config lookup.  
4. Agent N returns agent\_response\_N (containing success, data, error, metrics).  
5. Orchestrator uses agent\_response\_N.data and routing context\_updates to immutably construct the data\_context for Step N+1.  
6. Orchestrator constructs the execution\_metadata for Step N+1.  
7. This cycle repeats. AgentResponse.data is the primary carrier of evolving workflow data, while execution\_metadata tracks the process itself.

### **3.5. Control Flow (Orchestration)**

Control flow is determined by the Orchestrator interpreting the orchestration section of the config snapshot. It uses routing rules, loop definitions (type: "loop"), fan-out directives (next\_team as list), and concentrator definitions (type: "concentrator") associated with teams/steps to decide which agent(s) to call next and how to manage the data\_context and execution\_metadata.

## **4\. Agent Implementation (TaskAgent)**

### **4.1. Responsibilities**

* Act as a stateless executor based on configuration provided at instantiation (via snapshot) and execution metadata provided at invocation.  
* Interpret its configuration to determine the specific action (LLM call, Skill call, Script execution).  
* Prepare inputs for and process outputs from the executed action.  
* Adhere to Immutability and Totality principles.  
* Return results encapsulated in AgentResponse.

### **4.2. Configuration & Hydration**

* Receives the full effective config snapshot during instantiation by the AgentFactory.  
* Uses its unique\_name (provided during instantiation) to locate its specific configuration block within the snapshot (e.g., under llm\_config.agents.\<unique\_name\> or within a task definition).  
* Reads prompts, model parameters, skill names, script paths, etc., from its config block within the snapshot.

### **4.3. Dynamic Configuration Selection**

* The TaskAgent.process method *must* support selecting specific configuration elements (like persona\_key, llm\_prompt\_key, or skill parameters) based on values present in the execution\_metadata or data\_context passed to it.  
* A common pattern is to look for an override key (e.g., context.target\_persona\_override\[self.unique\_name\]) and use its value if present, otherwise falling back to the default specified in the snapshot.

## **5\. Skill Implementation**

### **5.1. Skill Definition & Contract**

* Skills represent reusable units of specific logic or side effects.  
* Implemented as Python classes or functions.  
* **Contract Requirement (FR14):** Skills intended for orchestrator invocation *must* adhere to a defined contract:  
  * **Input:** Accept necessary arguments derived from data\_context, execution\_metadata, and configuration (passed by the calling TaskAgent).  
  * **Output:** Return a predictable, structured result object encapsulating success or failure (e.g., a dataclass like SkillResult { success: bool; value: Any | None; error: str | None }).  
  * **Totality:** Handle their own expected errors (e.g., I/O, parsing, API) and report them via the return object, not by raising exceptions meant for flow control.

### **5.2. Skill Interface & Registry (Requirement Detail)**

* **Interface:** Define a standard base class or protocol for Skills (optional, but recommended for consistency).  
* **Registry/Discovery:** Implement a mechanism for the TaskAgent to find and invoke the correct Skill function/method based on a string name provided in its configuration (e.g., config: { skill: "AssetManagerSkill.process\_action" }). This could be:  
  * A simple dictionary mapping names to callables.  
  * A dynamic import mechanism based on convention.  
  * A more formal registry pattern.

### **5.3. Example Core Skills**

* **ChangeExtractorSkill:** Wraps SemanticIterator logic. Input: SolutionDesigner output string. Output: SkillResult(value=List\[change\_dict\]).  
* **AssetManagerSkill:** Wraps AssetManager logic. Method process\_action. Input: single change\_dict. Output: SkillResult(value=AssetResult).  
* **SemanticMergeSkill:** Wraps SemanticMerge logic. Input: original content, diff string. Output: SkillResult(value=merged\_content).  
* **TartxtRunnerSkill:** (New) Runs tartxt.py subprocess. Input: config (paths, exclusions). Output: SkillResult(value=tartxt\_output\_string). Handles subprocess management.  
* **(Others):** FileReaderSkill, WorkOrderArchitectSkill, ConfigWriterSkill, ImplementationReviewSkill, TestCaseGeneratorSkill, TestExecutorSkill, ReportGeneratorSkill.

## **6\. Orchestration Implementation**

### **6.1. Orchestrator Engine Responsibilities**

*(Implemented within run\_declarative\_workflow Prefect flow)*

* Load/receive initial context.  
* Invoke materialise\_config task to generate/load the effective configuration snapshot.  
* Initialize workflow state (context, metadata, counters) and EventLogger service.  
* Log WORKFLOW\_START.  
* Execute the main loop based on routing and control structures defined in the snapshot:  
  * Manage step sequence, context (immutably), and metadata.  
  * Check safety limits.  
  * Log STEP\_START.  
  * Invoke agent execution (run\_agent\_task) or handle internal control steps (loop, concentrator).  
  * Log STEP\_END with results.  
  * Handle step failures based on configuration.  
  * Evaluate routing rules (evaluate\_routing\_task) and determine next step(s). Log ROUTING\_EVALUATION.  
  * Handle fan-out dispatch. Log FAN\_OUT\_DISPATCH.  
  * Manage concentrator state and aggregation. Log CONCENTRATOR\_\* events.  
  * Manage loop state and iteration.  
  * Update context immutably based on results and routing context\_updates.  
* Log WORKFLOW\_END.  
* Return final result.

### **6.2. Declarative DSL Configuration (Detailed)**

*(Defined within system\_config.yml under orchestration.teams.\<team\_id\>)*

* **tasks (List):** Sequence of steps. Each item is a dictionary defining *one* step.  
  * **Standard Task Step (TaskAgent / OrchestratorAgent):**  
    * name: str (Unique ID for this step instance)  
    * agent\_type: str ("TaskAgent" or "OrchestratorAgent")  
    * config: Dict (Agent-specific config: skill name, prompt key, input/output mappings, etc.)  
    * persona\_key: Optional\[str\]  
  * **Loop Step:**  
    * name: str  
    * type: "loop"  
    * iterate\_on: str (Context field with list, e.g., "{{ data\_context.extracted\_changes }}")  
    * loop\_variable: str (Name for item in loop context, e.g., "current\_change")  
    * body: List\[Dict\] (Standard task definitions executed per iteration)  
  * **Concentrator Step:**  
    * name: str  
    * type: "concentrator"  
    * source\_step: str (Name of the step generating inputs to concentrate, especially within loops)  
    * source\_teams: Optional\[List\[str\]\] (List of team/step names generating inputs for fan-in)  
    * completion\_condition: Dict ({type: "count", value: N | value\_from\_context: "..."} or {type: "timeout", value: "Xs"} or {type: "signal", value: "..."})  
    * aggregation\_strategy: Dict ({type: "list" | "merge\_dict" | "custom\_skill", output\_field: "...", skill?: "..."})  
* **routing (Dict):** Defines flow after the *last* task in the tasks list completes (or after a concentrator).  
  * rules: List\[Dict\]  
    * condition: Union\[Dict, List\[Dict\]\] (Structured conditions: {task: "n", status: "success"}, {context\_field: "f.x", operator: "eq", value: V}, etc.)  
    * next\_team: Optional\[Union\[str, List\[str\]\]\] (Single ID, null, or List for fan-out)  
    * context\_updates: Optional\[Dict\] (Data to merge into next context)  
  * default: Optional\[Union\[str, List\[str\]\]\]

### **6.3. Context Management Conventions (Requirement Detail)**

* Clear conventions must be documented and enforced for:  
  * Naming context keys.  
  * Defining which parts of an AgentResponse.data are merged into the next step's data\_context. Default should be selective merging via context\_updates, not wholesale merging.  
  * Handling context merging after fan-out/parallel branches.  
  * Passing large data (prefer passing references/paths if feasible).  
  * Structure of the execution\_metadata object.

## **7\. Configuration Management**

### **7.1. Effective Configuration Snapshots**

* Generated via materialise\_config task. Immutable per run. Sole source of config during run. Persisted. Path/hash recorded.

### **7.2. Merging & Validation**

* Handled via render\_config utility. Fragments validated via JSON Schema before deep\_merge. Environment variables expanded.

### **7.3. Persona Injection**

* materialise\_config loads persona YAML based on persona\_key found in task fragments and injects it into the merge list with appropriate priority.

## **8\. Event Sourcing / Lineage**

### **8.1. Event Logger Service**

* Central service used by Orchestrator. Reads config from runtime.lineage. Persists structured events.

### **8.2. Event Trigger Points & Structure (Detailed)**

* Orchestrator emits events at: WORKFLOW\_START, STEP\_START, STEP\_END (incl. AgentResponse), ROUTING\_EVALUATION (incl. outcome, next team), LOOP\_ITERATION\_START, LOOP\_ITERATION\_END, CONCENTRATOR\_START, CONCENTRATOR\_INPUT\_RECEIVED, CONCENTRATOR\_END (incl. aggregated result), FAN\_OUT\_DISPATCH (incl. target teams), WORKFLOW\_END (incl. final status/error).  
* **Standard Event Structure:** *(See previous response for detailed JSON structure)* \- Includes event\_id, workflow\_run\_id, timestamp, sequence, event\_type, step\_name, payload (context/response data), execution\_metadata, config\_snapshot\_ref.

## **9\. Example Workflow: Coder Team**

*(Utilizes TaskAgent configured with ChangeExtractorSkill \-\> loop orchestration construct \-\> TaskAgent configured with AssetManagerSkill.process\_action (per item) \-\> concentrator orchestration construct \-\> final routing. See previous response for conceptual YAML.)*

## **10\. Migration Path (Conceptual)**

1. **Implement Core:** TaskAgent, AgentFactory, EventLogger, render\_config, update Orchestrator Engine (run\_declarative\_workflow).  
2. **Encapsulate Skills:** Wrap existing logic (SemanticIterator, AssetManager, etc.) into Skills adhering to the defined contract (return SkillResult). Define Skill interface/registry.  
3. **Update Config:** Modify system\_config.yml, create personas. Define initial teams using TaskAgent and skills. Add schema validation.  
4. **Implement Orchestration DSL:** Add loop, concentrator, fan-out (next\_team: \[...\]), context-aware routing, dynamic persona selection support to the Orchestrator Engine. Define context conventions.  
5. **Refactor Workflows:** Update team YAML to use new DSL features.  
6. **Testing:** Unit test Skills, TaskAgent dispatch, Orchestrator controls. Integration test full workflows.

---

This version (3.1) integrates the requirements for dynamic configuration, enhanced DSL capabilities for parallelism and control flow, formalized skill interfaces, context conventions, and detailed event sourcing into the existing framework, providing a comprehensive design specification.
