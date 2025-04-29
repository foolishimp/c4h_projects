## **C4H Agent System Refactoring: Factory Model & Enhanced Orchestration**

Version: 1.1  
Date: 2025-04-29  
**1. Introduction**

This document outlines the requirements and design for refactoring the C4H Agent System. The primary goal is to move from a model where agent behavior is tightly coupled to specific agent classes towards a more flexible factory pattern. In this new model, agent instances are created based on configuration that specifies a generic agent capability (agent_type), a specific set of behaviors and settings (persona_key), and a unique instance name (unique_name).

A secondary, related goal is to enhance the workflow orchestration capabilities, currently implemented using Prefect, to support more complex control flow, including conditional branching based on task outcomes and effective recursion (re-running teams with updated context). This will be achieved by evolving the configuration DSL (Domain-Specific Language) defined in YAML and adapting the Prefect flows to interpret this declarative DSL.

This refactoring aims to enhance reusability, configuration flexibility, and maintainability, enabling the creation of more complex workflows involving specialized teams like Requirements Generation, Implementation Review, and Test Automation. It builds upon the existing Agent_Design_Principles_v2.md, leveraging the LLM-First, Minimal Agent Logic, and Stateless Operation principles, while clarifying the roles of Agents and Skills.

**2. Requirements**

* **R1: Generic Agent Capabilities:** Introduce generic agent base classes representing core capabilities (e.g., single-shot execution, internal orchestration). (See Section 3.1)  
* **R2: Persona-Driven Configuration:** Define agent behavior (prompts, LLM settings, specific logic flags) in reusable "persona" configurations, separate from the generic agent classes. (See Section 3.2)  
* **R3: Factory-Based Instantiation:** Implement an Agent Factory responsible for creating agent instances based on configuration specifying agent type, persona, and unique instance name. (See Section 3.3)  
* **R4: Updated Orchestration Configuration:** Modify the system_config.yml structure for defining team tasks to support the new factory model (agent_type, persona_key, unique_name). (See Section 3.2)  
* **R5: Declarative Orchestration DSL:** Enhance the YAML DSL for orchestration.teams.<team_id>.routing.rules to support expressive conditional branching based on task outcomes and context variables. (See Section 3.4.1)  
* **R6: Context-Aware Recursion:** Implement a mechanism within the orchestration logic to allow teams to be re-executed with modified context, enabling effective recursion for scenarios like iterative refinement. (See Section 3.4.2)  
* **R7: Prefect-Based DSL Execution:** Refactor the Prefect implementation (c4h_services) to act as an execution engine for the declarative YAML DSL, dynamically running configured teams and tasks. (See Section 3.4.3)  
* **R8: Backward Compatibility:** The refactoring must not break existing workflows that use the current model (specific agent_class definitions in orchestration config). The Prefect execution engine must support both models. (See Section 3.5)  
* **R9: New Team Support:** The refactored system must support the definition and execution of the new proposed teams (Requirements, Implementation Review, Test Automation) using the generic agent/persona model and enhanced orchestration.  
* **R10: Clarified Agent/Skill Roles:** Maintain a clear conceptual distinction between Agents (orchestrated workflow participants) and Skills (reusable operational capabilities, often with side effects), even if Skills leverage BaseAgent infrastructure. (See Section 3.6)  
* **R11: Recursion Safety Controls:** Implement mechanisms within the Prefect orchestration flow to prevent infinite recursion or unproductive loops, specifically including **Maximum Total Team Executions** per workflow and **Maximum Team Recursion Depth** per team instance within a workflow. (See Section 3.4.4)

**3. Design**

**3.1. Generic Agent Classes (c4h_agents/agents/generic.py - New)**

* **GenericSingleShotAgent(BaseAgent):**  
  * **Purpose:** Represents agents performing a single primary LLM interaction based on input context (e.g., analysis, generation, evaluation). Replaces the need for many specific classes like the current SolutionDesigner.  
  * **__init__(self, persona_config: Dict, unique_name: str, full_config: Dict):** Accepts the merged persona configuration, a unique instance name, and the full system config (passed to BaseAgent.__init__). Stores persona_config and unique_name.  
  * **Configuration Access:** Overrides methods like _get_system_message, _get_prompt, _resolve_model, _get_agent_config to read directly from its self.persona_config dictionary instead of relying on BaseConfig's standard path lookups based on _get_agent_name().  
  * **_get_agent_name(self) -> str:** Returns the unique_name provided during instantiation for logging and identification.  
* **GenericOrchestratingAgent(BaseAgent):**  
  * **Purpose:** Represents agents that perform multiple internal steps, potentially calling skills or other LLM interactions based on a declarative execution_plan defined within their persona configuration. Replaces the need for hardcoded logic like in the current Coder.  
  * **__init__:** Similar to GenericSingleShotAgent. Stores the execution_plan from the persona_config.  
  * **Configuration Access:** Similar overrides as GenericSingleShotAgent.  
  * **process(self, context: Dict) -> AgentResponse:** Overrides the base process method. Instead of a direct LLM call, it interprets the self.execution_plan from its persona config. It manages an internal execution state, calls registered Skills (using self.call_skill for lineage), handles control flow (loops, conditionals defined in the plan), and aggregates results.  
  * **_get_agent_name(self) -> str:** Returns the unique_name.

**3.2. Configuration Enhancements (config/system_config.yml)**

* **llm_config.personas Section (New):**  
  * A new top-level key under llm_config to store reusable persona definitions.  
  * Each key under personas is a unique persona identifier (persona_key).  
  * The value is a dictionary containing all settings previously found under llm_config.agents.<agent_name> (provider, model, temperature, prompts, etc.) and potentially an execution_plan for orchestrating agents.  
  * Example:  
    llm_config:  
      personas:  
        solution_designer_v1:  
          provider: "anthropic"  
          model: "claude-3-7-sonnet-20250219"  
          temperature: 1  
          prompts:  
            system: "You are a solution designer..."  
            solution: "..."  
        standard_coder_persona:  
           provider: "anthropic" # Provider for merger skill, if needed by plan  
           model: "claude-3-opus-20240229"  
           temperature: 0  
           execution_plan: # Internal workflow definition  
             - name: "initialize_iterator"  
               skill: "SemanticIterator.initialize"  
               input: "{{context.input_data}}"  
               output_variable: "change_iterator"  
             - name: "process_changes_loop"  
               type: "loop"  
               iterate_on: "{{change_iterator}}"  
               loop_variable: "current_change_item"  
               body:  
                 - name: "apply_change"  
                   skill: "AssetManager.process_action"  
                   input: "{{current_change_item}}"  
                   output_variable: "apply_result"  
             - name: "aggregate_results" # ...  
      # Existing agents section for backward compatibility  
      agents:  
        discovery: # ...

* **orchestration.teams.tasks Structure Update:**  
  * Existing tasks using specific agent_class paths will continue to work.  
  * New tasks using the factory model will use:  
    * name: (Required) The unique instance name for this task step (e.g., design_solution, generate_wo). Used for logging, identification, and potentially routing conditions.  
    * agent_type: (Required) String key mapping to a generic agent class (e.g., "generic_single_shot", "generic_orchestrating"). The factory uses this to select the class.  
    * persona_key: (Required) String key referencing a definition under llm_config.personas (e.g., "solution_designer_v1").  
    * config: (Optional) Dictionary for task-specific overrides that will be deep-merged onto the loaded persona configuration *before* agent instantiation.  
* **Orchestration Safety Limits (New):**  
  * Add global safety limits under the orchestration: key:  
    orchestration:  
      enabled: true  
      entry_team: "discovery"  
      max_total_teams: 30 # Limit total team steps in a workflow  
      max_recursion_depth: 5 # Limit how many times one team can run  
      # ... teams: ...

  * Optionally, allow per-team overrides for max_recursion_depth:  
    orchestration:  
      # ...  
      teams:  
        engineering_team:  
          name: "Engineering Team"  
          max_recursion_depth: 3 # Override global limit for this team  
          tasks: # ...  
          routing: # ...

**3.3. Agent Factory (c4h_services/src/orchestration/factory.py - New)**

* **AgentFactory Class:**  
  * **__init__(self, full_system_config: Dict):** Stores the complete system configuration.  
  * **_get_persona_config(self, persona_key: str) -> Dict::** Loads the specified persona dictionary from self.full_system_config['llm_config']['personas']. Handles errors if the key doesn't exist.  
  * **_get_agent_class(self, agent_type: str) -> Type[BaseAgent]::** Maps the agent_type string to the actual generic agent class (e.g., "generic_single_shot" -> GenericSingleShotAgent). Uses dynamic import (importlib) or a simple mapping dictionary.  
  * **create_agent(self, task_config: Dict) -> BaseAgent::**  
    1. Extract name, agent_type, persona_key, and optional config (overrides) from task_config.  
    2. Call _get_agent_class(agent_type) to get the class reference.  
    3. Call _get_persona_config(persona_key) to get the base persona settings.  
    4. deep_merge the persona_config with the task-specific overrides.  
    5. Instantiate the retrieved agent class: agent_class(persona_config=merged_config, unique_name=name, full_config=self.full_system_config).  
    6. Return the instantiated agent.

**3.4. Orchestration Enhancements (Prefect-Based)**

The core idea is to use Prefect as the execution engine for the workflow defined declaratively in the YAML DSL.

**3.4.1. Declarative Routing DSL (system_config.yml)**

* Enhance the orchestration.teams.<team_id>.routing.rules.condition syntax. A structured dictionary format is preferred for clarity and easier parsing:  
  routing:  
    rules:  
      # Example 1: Branch based on specific task output field  
      - condition:  
          task: "impl_evaluator"       # Target task by name within the current team  
          output_field: "status"     # Path within the task's AgentResponse.data  
          operator: "equals"         # Comparison operator  
          value: "FAIL"              # Value to compare against  
        next_team: "correction_wo_generator_team"

      # Example 2: Combine task status and context variable (AND logic implied)  
      - condition:  
          - task: "impl_evaluator"  
            status: "success"         # Check overall task success (AgentResponse.success)  
          - context_field: "review_outcome.needs_rework" # Path within workflow context  
            operator: "equals"  
            value: false  
        next_team: null # Success, end workflow

      # Example 3: Simple task success check  
      - condition:  
          task: "impl_evaluator"  
          status: "success"  
        next_team: "manual_approval" # Default success path if above didn't match

    default: "handle_evaluation_error" # Fallback if 'impl_evaluator' task fails entirely

* Supported operator values should include equals, not_equals, contains, greater_than, less_than, exists, is_empty, etc.  
* Accessing task output requires the AgentResponse.data to be structured predictably.

**3.4.2. Context-Aware Recursion Mechanism**

* Effective recursion (re-running a team) is achieved by routing rules targeting a previously executed team ID. This functions like functional recursion because each execution receives a potentially *new*, immutable context, rather than modifying a shared stateful loop.  
* **Context Updates:** To prevent infinite loops with identical inputs and to pass necessary information (like a new WorkOrder ID for the next iteration), the team triggering the recursion must signal context modifications.  
  * The Prefect task responsible for routing (evaluate_routing_task) will return not only next_team_id but also a context_updates dictionary.  
  * This dictionary contains key-value pairs that should be merged into the main workflow context *before* the next team (the one being recursed to) is executed.  
  * Example: The correction_wo_generator_team's routing task, when deciding to loop back to engineering_team, would return {'next_team_id': 'engineering_team', 'context_updates': {'current_work_order_id': <new_id>}}.  
* **Orchestrator (Main Flow) Responsibility:** The main Prefect flow (run_declarative_workflow) is responsible for applying these context_updates (using an immutable merge approach) to create the *new* context passed to the subsequent team execution.  
* **Loop Prevention:** See Section 3.4.4.

**3.4.3. Prefect Implementation Refactoring (c4h_services)**

* **run_declarative_workflow (@flow):**  
  * Replaces the core logic of the old Orchestrator class.  
  * Takes initial context/config.  
  * Initializes safety counters (total teams executed, per-team execution counts).  
  * Contains the main loop that iterates based on current_team_id.  
  * **Checks safety limits** before executing a team.  
  * Calls execute_team_subflow for the current team.  
  * Calls evaluate_routing_task with the results.  
  * Applies context_updates to generate the context for the next iteration.  
  * Updates current_team_id.  
  * Handles termination (normal or due to limits).  
* **execute_team_subflow (@flow):**  
  * Takes team_id, config, current_context.  
  * Looks up the team's task list in config.  
  * Iterates through tasks, calling run_agent_task for each.  
  * Collects results (AgentResponse list).  
  * Returns the collected results.  
* **run_agent_task (@task):**  
  * Takes task_config (from YAML), config, current_context.  
  * **Integrates Agent Factory:** Checks task_config for agent_type/persona_key. If found, uses the factory (instantiated with config) to create the agent instance. If agent_class (legacy) is found, uses dynamic import.  
  * Calls the agent's process method with the current_context.  
  * Returns the AgentResponse.  
* **evaluate_routing_task (@task):**  
  * Takes team_results (list of AgentResponse), current_context, routing_config (from YAML).  
  * Implements the logic to parse and evaluate the enhanced condition syntax against the inputs.  
  * Returns {'next_team_id': '...', 'context_updates': {...}}.

**3.4.4. Recursion Safety Controls (New Section)**

To prevent infinite loops or runaway executions, the run_declarative_workflow Prefect flow **must** implement safety controls:

* **Maximum Total Team Executions:**  
  * Read orchestration.max_total_teams from the config.  
  * Maintain a counter for the total number of teams executed in the current run.  
  * Before executing any team, check if the counter exceeds the limit. If so, terminate the flow with an error state ("Max total teams exceeded").  
* **Maximum Team Recursion Depth:**  
  * Read the global orchestration.max_recursion_depth and any per-team overrides from the config.  
  * Maintain a dictionary tracking execution counts per team_id for the current run.  
  * Before executing a team, check if its count exceeds the applicable limit (per-team override or global default). If so, terminate the flow with an error state ("Max recursion depth for team X exceeded").  
  * Increment the team's count after successful execution.  
* **(Optional) Context Change Detection:**  
  * For more advanced loop detection, consider implementing state hashing as described previously. Maintain a history of relevant context hashes for each team invocation. If an identical relevant context hash is encountered for the same team, terminate with a "Non-progressing loop detected" error. This is more complex and should be added if the simpler count/depth limits prove insufficient.

**3.5. Backward Compatibility Strategy**

* The run_agent_task will check if a task definition in the YAML uses the legacy agent_class key or the new agent_type/persona_key keys.  
* It will branch its logic accordingly, either dynamically importing the specific class or using the Agent Factory.  
* Existing llm_config.agents.<agent_name> sections remain valid for legacy task definitions.  
* New development should use llm_config.personas and the factory model (agent_type, persona_key).

**3.6. Agent vs. Skill Roles (Clarified)**

* **Agents:**  
  * Primary, orchestrated participants in the workflow (Teams defined in YAML).  
  * Defined by Personas (configured goals, settings, prompts, potentially execution_plan). Instantiated via the Factory.  
  * Focus: Achieving a workflow stage's goal, primarily interacting with the LLM for reasoning/decision-making, managing their step in the flow, potentially adhering to external standards (like Tool Use if integrated into BaseLLM).  
  * Delegate operations with side effects or complex, reusable logic to Skills.  
* **Skills:**  
  * Reusable capabilities invoked *by* Agents (often directed by an Agent's execution_plan or LLM Tool Use).  
  * Encapsulate specific operations, often involving **side effects** (file I/O via AssetManager, API calls via SubmitWorkOrderSkill, running external tools like tartxt).  
  * May leverage BaseAgent infrastructure (logging, config, LLM) but are operationally focused.  
  * Interface defined by how Agents call them (Python methods or Tool Use definitions). Not directly managed by the top-level orchestrator/YAML.

This refined design provides a robust, configuration-driven architecture executed by Prefect, supporting complex workflows with branching and recursion while maintaining clarity, safety, and leveraging the power of LLMs effectively.