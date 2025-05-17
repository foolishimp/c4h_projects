# **C4H Refactoring: Requirements and Implementation Plan (v1.0)**

Date: May 7, 2025  
Status: Draft  
Based On: C4H Unified Architecture Document (Version 3.0)

## **1\. Introduction**

### **1.1. Purpose**

This document outlines the requirements and a high-level implementation plan for refactoring the C4H (Code for Humans) system to align with the "C4H Unified Architecture Document (Version 3.0)" (hereafter referred to as "ArchDocV3"). The goal is to evolve the C4H system into a more consistent, modular, and recursively orchestrated platform.

### **1.2. Scope**

This document covers the necessary changes to the core C4H agent framework, configuration model, orchestration logic, and specific agent implementations. It will serve as the basis for creating detailed work orders for the refactoring effort.

### **1.3. Reference**

This plan is a direct derivative of the principles and structures defined in the **C4H Unified Architecture Document (Version 3.0)**. All requirements and implementation tasks herein are intended to realize that architectural vision.

## **2\. Goals of this Refactoring Effort**

The primary goals of this refactoring initiative are to:

1. **Unify Orchestration:** Implement a single, consistent execution\_plan YAML syntax and a corresponding "Execution Plan Executor" to manage orchestration at all levels (top-level workflows, intra-team agent sequences, and intra-agent skill/LLM sequences).  
2. **Clarify Component Roles:** Enforce strict adherence to the defined roles of Generic Agent Types, Personas, Skills, and Teams as outlined in ArchDocV3.  
3. **Enhance Modularity and Reusability:** Ensure that Skills, Personas, and Teams (as self-contained orchestrated units) are designed for maximum reusability.  
4. **Improve Consistency and Predictability:** Strengthen the application of statelessness, immutability, and explicit data flow to enhance system consistency.  
5. **Streamline Agent Implementations:** Refactor generic agent classes to their core responsibilities, removing special-case logic.  
6. **Align with Prefect:** Ensure all executable units (Skills, Agent Instances, Teams) are consistently wrapped and managed as Prefect tasks or flows.  
7. **Improve Configuration Management:** Centralize and clarify the definitions of core components (agent types, skills, personas, teams) within the configuration model.

## **3\. Core Architectural Changes & Requirements**

This section details the specific changes required to align the C4H system with ArchDocV3.

### **3.1. The Universal execution\_plan and "Execution Plan Executor"**

* **REQ-EXEC-001:** Define and document a comprehensive YAML schema for the universal execution\_plan. This schema must support:  
  * enabled flag.  
  * A list of steps.  
  * For each step: name, type (e.g., skill\_call, agent\_call, team\_call, llm\_call, loop, branch, set\_value), node/target\_team/skill identifiers, params/input\_params (with context templating), output\_field, conditional rules (with condition objects and route\_to\_step/action directives), and step-level condition.  
  * Support for recursive invocation (a plan step calling a component that itself runs a plan).  
* **REQ-EXEC-002:** Implement a core "Execution Plan Executor" module in Python.  
  * This executor must be capable of parsing and executing any valid execution\_plan.  
  * It must manage context immutably, creating new context states for subsequent steps based on outputs.  
  * It must correctly handle routing logic, conditional execution, loops, and parameter passing.  
  * It must integrate with Prefect for executing the underlying nodes (Skills, Agent Instances, Teams).  
* **REQ-EXEC-003:** Ensure the "Execution Plan Executor" can be invoked at any level of the hierarchy (top-level workflow, team-internal, agent-internal).

### **3.2. Configuration Model (teams\_config or equivalent)**

* **REQ-CONF-001:** Establish a clear and centralized way to define and register core components within the configuration system (conceptually teams\_config or integrated into existing structures like llm\_config and orchestration):  
  * **agent\_types:** Maintain an internal enumeration or registry of available Generic Agent Types (GenericLLMAgent, GenericSkillAgent, GenericFallbackAgent). The concept of GenericOrchestratorAgent as a distinct type is to be phased out; its functionality is now tied to any agent whose persona contains an execution\_plan.  
  * **skills:** Implement a skill registry where skills are defined with their name, reference to Python implementation, and any default skill-level configuration.  
  * **personas:** All persona definitions must explicitly declare the agent\_type they configure.  
    * Personas for GenericLLMAgent will contain LLM parameters and prompts.  
    * Personas for GenericSkillAgent will specify the skill name (to be run) and skill\_params.  
    * Personas intended to drive orchestrated behavior (previously for GenericOrchestratorAgent) will now directly embed an execution\_plan. The agent\_type for such personas might be a general "OrchestratingAgent" or implicitly determined by the presence of an execution\_plan.  
  * **teams:** Team definitions will include their list of named agents (Agent Instances) and their own internal execution\_plan for orchestrating these agent instances.  
* **REQ-CONF-002:** Refine the Effective Configuration Snapshot generation process (materialise\_config) to correctly merge all these defined components and validate them against their schemas.  
* **REQ-CONF-003:** Ensure persona overrides at the agent instance level within a team definition are correctly applied.

### **3.3. Execution Units (Prefect Wrapping)**

* **REQ-PREFECT-001:** All C4H Skills must be wrapped as Prefect @tasks. The skill's execute(\*\*kwargs) method will be the task's callable.  
* **REQ-PREFECT-002:** The process(context) method of every C4H Agent Instance (Generic Agent \+ Persona) must be executable as a Prefect @task.  
  * If the agent's persona dictates a direct LLM or Skill call, the task executes this.  
  * If the agent's persona contains an execution\_plan, this task invokes the "Execution Plan Executor" with that plan.  
* **REQ-PREFECT-003:** Each C4H "Team" (as a self-contained orchestrated unit with its own execution\_plan for its agent instances) must be executable as a Prefect @flow (sub-flow). This flow will use the "Execution Plan Executor" to run the team's internal plan.

### **3.4. Generic Agent Types Refactoring**

* **REQ-AGENT-001 (GenericLLMAgent):**  
  * Remove all special-case logic, including the direct execution of tartxt.py.  
  * Its process method must solely focus on:  
    1. Retrieving LLM parameters and prompt templates from its persona.  
    2. Formatting the prompt using the input context.  
    3. Calling the configured LLM via the standard \_get\_completion\_with\_continuation (or similar BaseLLM method).  
    4. Processing and returning the LLM's response in a standard AgentResponse.  
* **REQ-AGENT-002 (GenericSkillAgent):**  
  * Its process method must solely focus on:  
    1. Retrieving the skill name and skill\_params from its persona.  
    2. Formatting/preparing skill\_params using the input context.  
    3. Invoking the specified skill via the skill registry and \_invoke\_skill (or equivalent).  
    4. Returning the skill's output in a standard AgentResponse.  
* **REQ-AGENT-003 (GenericFallbackAgent):**  
  * Confirm it functions as a specialized GenericLLMAgent with conservative persona defaults.  
* **REQ-AGENT-004 (Evolution of GenericOrchestratorAgent):**  
  * The distinct GenericOrchestratorAgent class may be deprecated or its core logic (executing an execution\_plan) moved to the universal "Execution Plan Executor."  
  * The *behavior* of an orchestrating agent will be determined by its persona containing an execution\_plan, not by selecting GenericOrchestratorAgent as its type.

### **3.5. Discovery Agent Implementation**

* **REQ-DISCOVERY-001:** Refactor the tartxt.py script (or its core logic) into a formal C4H Skill (e.g., named "tartxt\_runner"). This skill should accept parameters like input\_paths and exclusions.  
* **REQ-DISCOVERY-002:** Create a new Persona (e.g., "discovery\_by\_skill\_persona") designed for the GenericSkillAgent. This persona will:  
  * Specify agent\_type: GenericSkillAgent.  
  * Specify skill: "tartxt\_runner".  
  * Contain skill\_params that map to the tartxt\_runner skill's expected inputs (e.g., default input paths, exclusions).  
* **REQ-DISCOVERY-003:** Update the "discovery" task definition in the default orchestration (e.g., within the discovery team in system\_config.yml) to use an agent instance configured with this new persona and GenericSkillAgent type.

### **3.6. Coder Agent Implementation**

* **REQ-CODER-001:** The Coder functionality should be fully realized by an Agent Instance whose persona contains an execution\_plan.  
  * This persona (e.g., code\_implementer\_persona) will no longer specify agent\_type: GenericOrchestratorAgent (if that type is deprecated). The presence of the execution\_plan in the persona is sufficient.  
  * The execution\_plan within this persona will orchestrate skills like semantic\_iterator and asset\_manager as currently defined (e.g., in coder\_v1.yml).  
* **REQ-CODER-002:** The physical agents/coder.py file should be reviewed. If its logic is fully superseded by the persona-based execution\_plan, it should be deprecated and removed.

### **3.7. Team and Orchestration Logic**

* **REQ-ORCH-001:** Refactor the main orchestration logic (currently in c4h\_services.src.orchestration.orchestrator.Orchestrator and Prefect flows like run\_declarative\_workflow) to use the "Execution Plan Executor" as its core engine.  
* **REQ-ORCH-002:** Ensure that when the executor encounters a step to execute a "Team" (e.g., type: "team\_call"), it correctly retrieves that Team's definition (including its internal execution\_plan) and recursively invokes the executor for that Team's plan.  
* **REQ-ORCH-003:** Update all Team definitions in system\_config.yml (e.g., discovery, solution, coder teams) to align with the ArchDocV3 model:  
  * Each team will have a list of named agents (Agent Instances).  
  * Each team will have its own execution\_plan detailing how these agent instances are sequenced and interact.  
  * The top-level orchestration (previously orchestration.teams entries) will now be a higher-level execution\_plan orchestrating these Team definitions. (Consider if a top-level "workflow" definition is needed, or if the first "Team" in teams\_config.teams with a specific entry point flag is the main workflow).

### **3.8. Context Management and Data Flow**

* **REQ-CONTEXT-001:** Strictly enforce immutability of context. The "Execution Plan Executor" must create new context objects when applying updates from step outputs.  
* **REQ-CONTEXT-002:** Standardize context templating (e.g., {{context.path.to.value}}) across all execution\_plan parameter definitions.  
* **REQ-CONTEXT-003:** Define clear conventions for how data is passed into and out of Skills, Agent Instances, and Teams when they are called as nodes in an execution\_plan (e.g., via params/input\_params and output\_field).

## **4\. High-Level Implementation Plan & Tasks**

This is a preliminary breakdown of tasks. Each will need to be expanded into detailed work orders.

1. **Phase 1: Core Engine and Configuration Model**  
   * **TASK-CORE-001:** Design and implement the universal "Execution Plan Executor" module.  
   * **TASK-CORE-002:** Define and implement the final YAML schema for execution\_plan.  
   * **TASK-CORE-003:** Refactor the configuration loading and Effective Configuration Snapshot generation (materialise\_config) to support the new centralized definitions (agent\_types, skills, personas with explicit agent\_type and embedded plans, teams with internal plans).  
   * **TASK-CORE-004:** Implement/refine the Skill Registry.  
   * **TASK-CORE-005:** Update Prefect task/flow wrappers for Skills, Agent Instances, and Teams to integrate with the "Execution Plan Executor" where appropriate.  
2. **Phase 2: Agent and Persona Refactoring**  
   * **TASK-AGENT-R-001:** Refactor GenericLLMAgent to remove tartxt logic and ensure it's a pure LLM interface.  
   * **TASK-AGENT-R-002:** Implement tartxt.py functionality as a registered Skill ("tartxt\_runner").  
   * **TASK-AGENT-R-003:** Create the new GenericSkillAgent persona for Discovery (e.g., "discovery\_by\_skill\_persona").  
   * **TASK-AGENT-R-004:** Update the default "discovery" task/team to use the new Skill-based Discovery agent instance.  
   * **TASK-AGENT-R-005:** Refactor the Coder functionality: ensure code\_implementer\_persona contains the full execution\_plan for semantic\_iterator and asset\_manager. Deprecate/remove agents/coder.py if fully superseded.  
   * **TASK-AGENT-R-006:** Review and update all existing Personas to:  
     * Explicitly declare their target agent\_type.  
     * Move any orchestration logic into an embedded execution\_plan if they are meant to drive orchestrated behavior.  
3. **Phase 3: Orchestration Logic Refactoring**  
   * **TASK-ORCH-R-001:** Refactor the top-level workflow execution (e.g., run\_declarative\_workflow and the Orchestrator class) to primarily use the "Execution Plan Executor" with a top-level execution\_plan that orchestrates Team calls.  
   * **TASK-ORCH-R-002:** Convert existing team definitions in system\_config.yml into the new structure: each team having its own agents list and internal execution\_plan.  
   * **TASK-ORCH-R-003:** Implement the recursive invocation logic where the "Execution Plan Executor" can call itself for sub-plans (Team plans, Persona plans).  
4. **Phase 4: Testing and Validation**  
   * **TASK-TEST-001:** Develop unit tests for the "Execution Plan Executor."  
   * **TASK-TEST-002:** Develop unit tests for refactored Generic Agent Types and their interaction with personas.  
   * **TASK-TEST-003:** Develop integration tests for individual Team execution\_plans.  
   * **TASK-TEST-004:** Develop end-to-end tests for complete workflows defined by top-level execution\_plans orchestrating multiple teams.  
5. **Phase 5: Documentation**  
   * **TASK-DOC-001:** Update all relevant C4H documentation (user guides, developer guides, API references) to reflect the new architecture.  
   * **TASK-DOC-002:** Formally publish the "C4H Unified Architecture Document (Version 3.0)."  
   * **TASK-DOC-003:** Create detailed documentation for the execution\_plan YAML schema and its usage.

## **5\. Impact and Considerations**

### **5.1. Impact on Existing Functionality**

* This is a significant refactoring and will touch many core parts of the system.  
* Existing workflow configurations (system\_config.yml, persona files) will need to be migrated to the new structure.  
* Thorough testing will be required to ensure no regressions in core functionality (Discovery, Solution Design, Coding).

### **5.2. Testing Strategy**

* **Unit Tests:** For individual components like the "Execution Plan Executor," refactored generic agents, and skills.  
* **Integration Tests:** For verifying the orchestration within a single execution\_plan (e.g., a Team's internal plan, an orchestrating Persona's plan).  
* **End-to-End Tests:** For complete workflows involving multiple levels of orchestration.  
* **Configuration Validation Tests:** Ensuring the system correctly parses and validates the new configuration structures.

### **5.3. Documentation**

* All developer and user documentation will need to be updated to reflect the new architecture, particularly around how workflows, teams, agents, and personas are defined and configured using execution\_plans.

### **5.4. Potential Risks**

* **Complexity of Migration:** Converting existing configurations to the new recursive execution\_plan model might be complex.  
* **Performance:** The recursive nature of the "Execution Plan Executor" needs to be implemented efficiently.  
* **Debugging:** Debugging deeply nested execution\_plans might require enhanced logging and observability tools.

This refactoring, while substantial, promises to create a more robust, flexible, and conceptually clean C4H system that is easier to maintain and extend in the long run.
