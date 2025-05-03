# C4H Agent System: Agent Design Principles (v2.1 - Updated)

**Date:** 2025-05-04 (Sydney Time)
**Status:** Adopted (Reflects Factory Model & Snapshot Architecture)

## 1. Overarching Principles

These principles guide the fundamental design and interaction patterns within the C4H Agent System.

### 1.1. Configuration-Driven Behavior (LLM-First Processing)
* Agents act as executors of behaviour defined entirely within configuration (personas, execution plans).
* Complex logic, decision-making, validation, and data transformation should be offloaded to the LLM or specialized Skills whenever feasible.
* **Template Configuration Over Code:** All prompts and critical behavioural parameters *must* reside in configuration files (personas, system config), not hardcoded in agent logic[cite: 467].
* Agent code focuses on:
    * Interpreting its configuration from the effective snapshot.
    * Preparing inputs for and processing outputs from LLMs or Skills based on config.
    * Managing execution context (immutably).
    * Handling infrastructure concerns (logging, errors, basic I/O).

### 1.2. Minimal Agent Logic
* Keep agent class implementations lean and focused on their core capability type (e.g., LLM interaction, orchestration, skill execution)[cite: 468].
* Avoid embedding complex business logic, detailed parsing, or extensive data transformations directly within agent classes. Delegate to LLMs or specialized Skills.
* Agents should primarily orchestrate calls to LLMs or Skills based on their configuration.

### 1.3. Immutability
* **Configuration:** Agents operate *exclusively* on an immutable Effective Configuration Snapshot generated at the start of a run. They do not modify this configuration.
* **State/Context:** Agents and Skills should treat the input `context` dictionary as immutable. They must not modify the input context directly. Instead, they should return necessary updates or the complete new state within their result object (e.g., `AgentResponse.data`). The orchestrator is responsible for constructing the *next* context based on the previous context and the returned updates.

### 1.4. Total Functions (Predictable Returns)
* Agent `process` methods and Skill execution functions should strive to be total functions.
* They must always return a predictable, structured response object (e.g., `AgentResponse`, `SkillResult`) that explicitly represents either success (with data) or a known failure condition (with an error description).
* Foreseeable errors (LLM API errors, file not found, validation issues, configuration problems) must be caught internally and returned as part of the standard response structure (`success=False`, `error=...`), not by raising exceptions intended for flow control.
* Unhandled exceptions should be reserved *only* for unexpected programming errors or critical, unrecoverable infrastructure failures.

## 2. Implementation Principles

### 2.1. Single Responsibility & Type-Based Roles
* Each agent instance performs a single, clearly defined task based on its configured `agent_type` (e.g., `GenericLLMAgent`, `GenericOrchestratorAgent`, `GenericSkillAgent`, `GenericFallbackAgent`) and unique `name`.
* Agents do not duplicate the core responsibilities of other types.
* Agents pass data through the workflow, relying on subsequent agents/skills for processing as defined by the orchestration configuration.

### 2.2. Minimal Processing (Delegation)
* Default to passing data directly to the next step (LLM or Skill) as defined by configuration.
* Only perform data transformation if it's essential for the agent's core function (e.g., formatting an LLM request based on a prompt template).
* Delegate complex parsing, validation, or business logic to specialized Skills or the LLM.

### 2.3. Clear Boundaries & Interfaces
* Agents interact based on the orchestration defined in the configuration (`orchestration.teams`).
* Agents expect inputs and produce outputs according to predefined structures (e.g., `AgentResponse`, context conventions).
* An agent's configuration (derived from the effective snapshot based on its unique name) is its primary contract.

### 2.4. Logging Over Internal Validation
* Prioritize comprehensive structured logging for observability and debugging[cite: 470]. Logs should allow reconstruction of the agent's actions and decisions based on its input context and configuration.
* Agents perform minimal internal validation, primarily checking for the presence of required configuration or context keys needed for their *own* operation.
* Business logic validation should be handled by the LLM or dedicated validation Skills.

### 2.5. Graceful Error Handling (via Total Functions)
* Agents handle *expected* errors related to their specific function (e.g., LLM API call failures, skill execution failures) by returning `AgentResponse(success=False, error=...)`.
* Error messages should be informative.
* Do not swallow errors silently; report them in the structured response.
* Let unexpected runtime exceptions propagate to be caught by the orchestrator's top-level error handling.

### 2.6. Stateless Operation & Immutable Context
* Agents themselves do not maintain state between separate `process` calls. Each execution is driven by the provided `context` and the immutable effective configuration snapshot.
* State progression is managed by the orchestrator, which constructs the *next* context immutably based on the results of the previous step [see Principle 1.3].

### 2.7. Snapshot-Based Execution
* All agent behaviour (prompts, models, parameters, execution plans) is derived *solely* from the Effective Configuration Snapshot created for the specific run.
* Agents are instantiated by a Factory which provides the full snapshot.
* This ensures reproducibility and auditability.

### 2.8. Observable Behavior
* Leverage structured logging heavily (see Principle 2.4).
* Ensure clear, consistent input/output structures (`AgentResponse`, Skill results).
* Lineage tracking (handled by `BaseLineage` and configured globally) provides workflow traceability.

### 2.9. Focused Testing
* Unit tests should focus on the agent's core responsibility based on its type:
    * `GenericLLMAgent`: Verify correct prompt formatting, LLM parameter construction, and response handling based on configuration. Mock LLM calls.
    * `GenericOrchestratorAgent`: Verify correct parsing and execution of `execution_plan` steps, state/context management (immutability), and result aggregation. Mock Skill/LLM calls within the plan.
    * `GenericSkillAgent`: Verify the specific skill's logic (parsing, file I/O, etc.) and its handling of inputs/outputs according to its defined contract. Mock external dependencies if any.
* Do *not* test the LLM's generation quality or the internal logic of other agents/skills within a single agent's unit test.
* Integration tests verify the interaction between agents/teams based on orchestration configuration.

### 2.10. Forward-Only Flow (Orchestration Driven)
* Data and control flow primarily move forward as dictated by the team orchestration and routing rules defined in the configuration.
* Agents receive context, perform their task based on their type and configuration, and return results/updates.
* The orchestrator manages the sequence.

## 3. Benefits
* **Maintainability:** Clearer roles, configuration-driven behavior reduces code complexity.
* **Flexibility:** Easily define new agent behaviors or workflows by changing configuration and personas.
* **Testability:** Focused testing on core agent responsibilities and infrastructure.
* **Reproducibility:** Snapshot-based execution ensures runs are deterministic based on their config.
* **Robustness:** Explicit error handling via total functions and immutable state reduce unexpected side effects.
* **Leverages LLM:** Maximizes LLM capabilities for complex tasks.

## 4. Application Guidelines Summary
1.  **Define via Config:** Agent behavior = Agent Type + Configuration (from Snapshot).
2.  **Keep Agents Simple:** Focus agent code on interpreting config and managing its core execution type (LLM call, plan execution, skill execution).
3.  **Embrace Immutability:** Treat context as immutable; return updates, don't modify in place.
4.  **Use Total Functions:** Return structured results (`AgentResponse` etc.) for success and expected failures. Don't use exceptions for flow control.
5.  **Log Everything:** Use structured logging for observability.
6.  **Delegate Logic:** Push complex logic, validation, and transformation to LLMs or specialized Skills.