# C4H Agent System: Configuration Design Guide (Updated)

This comprehensive guide details every configuration field in the C4H Agent System, organized by section. The configuration follows a hierarchical structure with specific design principles that ensure predictable and maintainable behavior.

## Table of Contents

1.  [Project Configuration](#project-configuration)
2.  [Intent Configuration](#intent-configuration)
3.  [LLM Configuration](#llm-configuration)
    * [Provider Configuration](#provider-configuration)
    * [Agent Configuration](#agent-configuration)
4.  [Orchestration Configuration](#orchestration-configuration)
    * [Team Definition](#team-definition)
    * [Task Configuration](#task-configuration)
    * [Routing Rules](#routing-rules)
5.  [Runtime Configuration](#runtime-configuration)
    * [Workflow Storage](#workflow-storage)
    * [Lineage Tracking](#lineage-tracking)
6.  [Backup Configuration](#backup-configuration)
7.  [Logging Configuration](#logging-configuration)

## Project Configuration

The `project` section defines the project scope and workspace locations.

| Field            | Type   | Description                                                  | Default                |
| :--------------- | :----- | :----------------------------------------------------------- | :--------------------- |
| `path`           | String | Absolute path to the project being analyzed                  | `.` (current directory) |
| `workspace_root` | String | Directory for working files and outputs                      | `workspaces`           |
| `source_root`    | String | Base directory for source code (relative to project path)    | `.`                    |
| `output_root`    | String | Base directory for output files (relative to project path)   | `.`                    |
| `config_root`    | String | Directory for configuration files (relative to project path) | `config`               |

Example:

```yaml
project:
  path: "/path/to/your/project"
  workspace_root: "workspaces"
  source_root: "src"
  output_root: "output"
  config_root: "config"
```

## Intent Configuration

The `intent` section describes what the system should accomplish, serving as the work order for the agents.

| Field          | Type  | Description                               | Required |
| :------------- | :---- | :---------------------------------------- | :------- |
| `description`  | String | Detailed description of the refactoring intent | Yes      |
| `target_files` | Array | Optional list of specific files to target | No       |

Example:

```yaml
intent:
  description: |
    Refactor the error handling system to use a consistent approach across all services,
    with proper logging and error propagation.
  target_files:
    - "src/services/auth.js"
    - "src/services/user.js"
```

## LLM Configuration

The `llm_config` section configures language model providers and agent-specific settings.

### Provider Configuration

The `providers` subsection defines available LLM providers and their settings.

| Field               | Type    | Description                                  | Notes             |
| :------------------ | :------ | :------------------------------------------- | :---------------- |
| `api_base`          | String  | Base URL for provider's API                  | Provider-specific |
| `context_length`    | Integer | Maximum token context length                 | Provider-specific |
| `env_var`           | String  | Environment variable for API key             | Provider-specific |
| `default_model`     | String  | Default model for this provider              | Provider-specific |
| `valid_models`      | Array   | List of supported models                     | Provider-specific |
| `extended_thinking` | Object  | Configuration for extended thinking (Claude) | Claude models only |
| `litellm_params`    | Object  | LiteLLM configuration parameters             | For all providers |
| `model_params`      | Object  | Parameters passed directly to the model API  | Provider-specific |

**Extended thinking configuration (Claude models):**

| Field               | Type    | Description                     | Default  |
| :------------------ | :------ | :------------------------------ | :------- |
| `enabled`           | Boolean | Whether extended thinking is enabled | `false`  |
| `budget_tokens`     | Integer | Token budget for thinking       | `32000`  |
| `min_budget_tokens` | Integer | Minimum token budget            | `1024`   |
| `max_budget_tokens` | Integer | Maximum token budget            | `128000` |

**LiteLLM parameters:**

| Field               | Type    | Description                     | Default |
| :------------------ | :------ | :------------------------------ | :------ |
| `retry`             | Boolean | Whether to retry failed calls   | `true`  |
| `max_retries`       | Integer | Maximum number of retries       | `5`     |
| `timeout`           | Integer | Timeout in seconds              | `30`    |
| `rate_limit_policy` | Object  | Rate limiting configuration     |         |
| `backoff`           | Object  | Retry backoff configuration     |         |

**Example Provider Configurations:**

```yaml
llm_config:
  providers:
    anthropic:
      api_base: "https://api.anthropic.com"
      context_length: 200000
      env_var: "ANTHROPIC_API_KEY"
      default_model: "claude-3-5-sonnet-20241022"
      valid_models:
        - "claude-3-7-sonnet-20250219"
        - "claude-3-5-sonnet-20241022"
        - "claude-3-5-haiku-20241022"
        - "claude-3-opus-20240229"
        - "claude-3-sonnet-20240229"
        - "claude-3-haiku-20240307"
      extended_thinking:
        enabled: false
        budget_tokens: 32000
      litellm_params:
        retry: true
        max_retries: 5
        timeout: 30
        backoff:
          initial_delay: 1
          max_delay: 30
          exponential: true

    openai:
      api_base: "https://api.openai.com/v1"
      env_var: "OPENAI_API_KEY"
      default_model: "gpt-4o"
      valid_models:
        - "gpt-4o"
        - "gpt-4o-mini"
        - "gpt-4"
        - "gpt-4-turbo"
        - "o1"
        - "o1-mini"
        - "o3-mini"
      litellm_params:
        retry: true
        max_retries: 3
        timeout: 30
        backoff:
          initial_delay: 1
          max_delay: 20
          exponential: true

    gemini:
      # Standard Google AI API endpoint base
      api_base: "https://generativelanguage.googleapis.com/v1beta"
      context_length: 32000 # Adjust based on specific model used
      env_var: "GEMINI_API_KEY" # API Key is used for this endpoint
      default_model: "gemini-1.5-pro-latest"
      valid_models:
        - "gemini-2.5-pro-preview-03-25" # The model confirmed working via curl
        - "gemini-1.5-pro-latest"
        - "gemini-1.5-flash-latest"
        # Add other models as needed
      # Model-specific parameters (passed directly in the API call)
      model_params:
        maxOutputTokens: 8192
        temperature: 0.7 # Example, adjust as needed per agent
      # LiteLLM parameters (may not apply to direct calls, but kept for consistency)
      litellm_params:
        retry: true
        max_retries: 3
        timeout: 60
        backoff:
          initial_delay: 2
          max_delay: 30
          exponential: true
      # NOTE: Gemini 2.5 models currently use a direct API call in base_llm.py
      # to bypass LiteLLM routing/authentication issues encountered previously.
      # This means LiteLLM's standard handling (e.g., model prefix 'gemini/')
      # is NOT used for these specific models in the current codebase.
      # The 'api_base' here is used by the direct call logic.

    xai:
      api_base: "https://api.x.ai/v1"
      env_var: "XAI_API_KEY"
      default_model: "grok-1.5-flash"
      valid_models:
        - "grok-1"
        - "grok-1.5"
        - "grok-1.5-flash"
      context_length: 128000
      litellm_params:
        retry: true
        max_retries: 3
        timeout: 45
        backoff:
          initial_delay: 1
          max_delay: 20
          exponential: true

  default_provider: "anthropic"
  default_model: "claude-3-opus-20240229"
```

### Agent Configuration

The `agents` subsection defines configuration for each agent type.

#### Base Agent Configuration

| Field                                  | Type    | Description                               | Default      |
| :------------------------------------- | :------ | :---------------------------------------- | :----------- |
| `storage.root_dir`                     | String  | Root directory for agent storage          | `workspaces` |
| `storage.retention.max_age_days`       | Integer | Maximum age for stored data in days     | `30`         |
| `storage.retention.max_runs`           | Integer | Maximum number of runs to keep          | `10`         |
| `storage.error_handling.ignore_failures` | Boolean | Whether to ignore storage failures        | `true`       |
| `storage.error_handling.log_level`     | String  | Log level for storage errors            | `ERROR`      |

#### Lineage Configuration

| Field                           | Type    | Description                             | Default            |
| :------------------------------ | :------ | :-------------------------------------- | :----------------- |
| `enabled`                       | Boolean | Whether lineage tracking is enabled   | `true`             |
| `namespace`                     | String  | Namespace for lineage events            | `c4h_agents`       |
| `backend.type`                  | String  | Backend type (`file` or `marquez`)    | `file`             |
| `backend.path`                  | String  | Path for file backend                   | `workspaces/lineage` |
| `backend.url`                   | String  | URL for Marquez backend                 | `null`             |
| `retention.max_age_days`        | Integer | Maximum age for lineage data            | `30`               |
| `retention.max_runs`            | Integer | Maximum number of runs to keep          | `100`              |
| `context.include_metrics`       | Boolean | Include metrics in lineage events       | `true`             |
| `context.include_token_usage`   | Boolean | Include token usage in lineage events   | `true`             |
| `context.record_timestamps`     | Boolean | Include timestamps in lineage events    | `true`             |

#### Discovery Agent Configuration

| Field                            | Type    | Description                           | Default                 |
| :------------------------------- | :------ | :------------------------------------ | :---------------------- |
| `default_provider`               | String  | Provider to use                       | Inherited from global |
| `default_model`                  | String  | Model to use                          | Inherited from global |
| `temperature`                    | Float   | Temperature setting                   | `0`                     |
| `tartxt_config.script_path`    | String  | Path to tartxt script                 | `c4h_agents/skills/tartxt.py` |
| `tartxt_config.input_paths`    | Array   | Paths to analyze                      | `["./"]`                |
| `tartxt_config.exclusions`     | Array   | Patterns to exclude                   | `["**/__pycache__/**"]` |
| `prompts.system`                 | String  | System prompt template                | Defined in system_config.yml |

#### Solution Designer Configuration

| Field                 | Type    | Description                     | Default                 |
| :-------------------- | :------ | :------------------------------ | :---------------------- |
| `provider`            | String  | Provider to use                 | Inherited from global |
| `model`               | String  | Model to use                    | Inherited from global |
| `temperature`         | Float   | Temperature setting             | `0`                     |
| `prompts.system`      | String  | System prompt template          | Defined in system_config.yml |
| `prompts.solution`    | String  | Solution prompt template        | Defined in system_config.yml |
| `extended_thinking.*` | Object  | Configuration for Claude models | See Provider Config   |

#### Semantic Extraction Configuration

| Field             | Type    | Description                   | Default                 |
| :---------------- | :------ | :---------------------------- | :---------------------- |
| `provider`        | String  | Provider to use               | Inherited from global |
| `model`           | String  | Model to use                  | Inherited from global |
| `temperature`     | Float   | Temperature setting           | `0`                     |
| `prompts.system`  | String  | System prompt template        | Defined in system_config.yml |
| `prompts.extract` | String  | Extraction prompt template    | Defined in system_config.yml |

#### Semantic Iterator Configuration

| Field                          | Type    | Description                         | Default                 |
| :----------------------------- | :------ | :---------------------------------- | :---------------------- |
| `provider`                     | String  | Provider to use                     | Inherited from global |
| `model`                        | String  | Model to use                        | Inherited from global |
| `temperature`                  | Float   | Temperature setting                 | `0`                     |
| `extractor_config.mode`        | String  | Extraction mode (`fast` or `slow`)  | `fast`                  |
| `extractor_config.allow_fallback` | Boolean | Allow fallback to slow mode         | `true`                  |
| `prompts.system`               | String  | System prompt template              | Defined in system_config.yml |

#### Semantic Fast Extractor Configuration

| Field             | Type    | Description                   | Default                 |
| :---------------- | :------ | :---------------------------- | :---------------------- |
| `provider`        | String  | Provider to use               | Inherited from global |
| `model`           | String  | Model to use                  | Inherited from global |
| `temperature`     | Float   | Temperature setting           | `0`                     |
| `prompts.system`  | String  | System prompt template        | Defined in system_config.yml |
| `prompts.extract` | String  | Extraction prompt template    | Defined in system_config.yml |

#### Semantic Slow Extractor Configuration

| Field             | Type    | Description                   | Default                 |
| :---------------- | :------ | :---------------------------- | :---------------------- |
| `provider`        | String  | Provider to use               | Inherited from global |
| `model`           | String  | Model to use                  | Inherited from global |
| `temperature`     | Float   | Temperature setting           | `0`                     |
| `prompts.system`  | String  | System prompt template        | Defined in system_config.yml |
| `prompts.extract` | String  | Extraction prompt template    | Defined in system_config.yml |

#### Semantic Merge Configuration

| Field                             | Type    | Description                   | Default                 |
| :-------------------------------- | :------ | :---------------------------- | :---------------------- |
| `provider`                        | String  | Provider to use               | Inherited from global |
| `model`                           | String  | Model to use                  | Inherited from global |
| `temperature`                     | Float   | Temperature setting           | `0`                     |
| `merge_config.preserve_formatting`| Boolean | Preserve code formatting      | `true`                  |
| `merge_config.allow_partial`      | Boolean | Allow partial merges          | `false`                 |
| `prompts.system`                  | String  | System prompt template        | Defined in system_config.yml |
| `prompts.merge`                   | String  | Merge prompt template         | Defined in system_config.yml |

#### Coder Configuration

| Field             | Type    | Description                           | Default                 |
| :---------------- | :------ | :------------------------------------ | :---------------------- |
| `provider`        | String  | Provider to use                       | Inherited from global |
| `model`           | String  | Model to use                          | Inherited from global |
| `temperature`     | Float   | Temperature setting                   | `0`                     |
| `backup_enabled`  | Boolean | Whether to backup files before changes | `true`                  |
| `prompts.system`  | String  | System prompt template                | Defined in system_config.yml |

#### Assurance Configuration

| Field             | Type    | Description             | Default                 |
| :---------------- | :------ | :---------------------- | :---------------------- |
| `provider`        | String  | Provider to use         | Inherited from global |
| `model`           | String  | Model to use            | Inherited from global |
| `temperature`     | Float   | Temperature setting     | `0`                     |
| `prompts.system`  | String  | System prompt template  | Defined in system_config.yml |

## Orchestration Configuration

The `orchestration` section defines how teams of agents work together to accomplish tasks.

| Field                          | Type    | Description                             | Default     |
| :----------------------------- | :------ | :-------------------------------------- | :---------- |
| `enabled`                      | Boolean | Whether orchestration is enabled        | `true`      |
| `entry_team`                   | String  | First team to execute                   | `discovery` |
| `error_handling.retry_teams`   | Boolean | Whether to retry failed teams           | `true`      |
| `error_handling.max_retries`   | Integer | Maximum number of team retries          | `2`         |
| `error_handling.log_level`     | String  | Log level for orchestration errors    | `ERROR`     |

### Team Definition

Each team under the `teams` section defines a group of related tasks that execute in sequence.

| Field     | Type   | Description                         | Required |
| :-------- | :----- | :---------------------------------- | :------- |
| `name`    | String | Display name for the team           | Yes      |
| `tasks`   | Array  | List of tasks to execute            | Yes      |
| `routing` | Object | Rules for determining next team     | Yes      |

### Task Configuration

Each task in a team represents an agent execution.

| Field               | Type    | Description                                | Required |
| :------------------ | :------ | :----------------------------------------- | :------- |
| `name`              | String  | Name of the task                           | Yes      |
| `agent_class`       | String  | Class name or path for the agent           | Yes      |
| `requires_approval` | Boolean | Whether approval is required before execution | No (default: `false`) |
| `max_retries`       | Integer | Maximum number of retries for this task    | No (default: `1`) |
| `config`            | Object  | Additional configuration for this task     | No (default: `{}`) |

### Routing Rules

Rules that determine which team executes next.

| Field               | Type   | Description                                                    | Required |
| :------------------ | :----- | :------------------------------------------------------------- | :------- |
| `rules`             | Array  | Ordered list of routing rules                                  | No       |
| `rules[].condition` | String | Condition for this rule (e.g., `all_success`, `any_failure`)   | Yes      |
| `rules[].next_team` | String | ID of the next team to execute, `null` to end workflow         | Yes      |
| `default`           | String | Default next team if no rules match, `null` to end workflow | Yes      |

Example:

```yaml
orchestration:
  enabled: true
  entry_team: "discovery"
  error_handling:
    retry_teams: true
    max_retries: 2
    log_level: "ERROR"
  teams:
    discovery:
      name: "Discovery Team"
      tasks:
        - name: "discovery"
          agent_class: "c4h_agents.agents.discovery.DiscoveryAgent"
          requires_approval: false
          max_retries: 2
      routing:
        default: "solution"

    solution:
      name: "Solution Design Team"
      tasks:
        - name: "solution_designer"
          agent_class: "c4h_agents.agents.solution_designer.SolutionDesigner"
          requires_approval: true
          max_retries: 1
      routing:
        rules:
          - condition: "all_success"
            next_team: "coder"
          - condition: "any_failure"
            next_team: "fallback"
        default: "coder"

    coder:
      name: "Coder Team"
      tasks:
        - name: "coder"
          agent_class: "c4h_agents.agents.coder.Coder"
          requires_approval: true
          max_retries: 1
      routing:
        rules:
          - condition: "all_success"
            next_team: null
        default: null

    fallback:
      name: "Fallback Team"
      tasks:
        - name: "fallback_coder"
          agent_class: "c4h_agents.agents.coder.Coder"
          config:
            temperature: 0
      routing:
        default: null
```

## Runtime Configuration

The `runtime` section configures workflow and lineage behavior.

### Workflow Storage

| Field                                  | Type    | Description                             | Default                     |
| :------------------------------------- | :------ | :-------------------------------------- | :-------------------------- |
| `enabled`                              | Boolean | Whether workflow storage is enabled     | `true`                      |
| `root_dir`                             | String  | Root directory for workflow storage     | `workspaces/workflows`      |
| `format`                               | String  | Format string for workflow directories  | `yymmdd_hhmm_{workflow_id}` |
| `retention.max_runs`                   | Integer | Maximum number of runs to keep          | `10`                        |
| `retention.max_days`                   | Integer | Maximum age for stored data in days     | `30`                        |
| `error_handling.ignore_storage_errors` | Boolean | Whether to ignore storage failures        | `true`                      |
| `error_handling.log_level`             | String  | Log level for storage errors            | `ERROR`                     |

### Lineage Tracking

| Field                           | Type    | Description                             | Default            |
| :------------------------------ | :------ | :-------------------------------------- | :----------------- |
| `enabled`                       | Boolean | Whether lineage tracking is enabled   | `true`             |
| `namespace`                     | String  | Namespace for lineage events            | `c4h_agents`       |
| `separate_input_output`         | Boolean | Store inputs and outputs separately     | `true`             |
| `backend.type`                  | String  | Backend type (`file` or `marquez`)    | `file`             |
| `backend.path`                  | String  | Path for file backend                   | `workspaces/lineage` |
| `error_handling.ignore_failures`| Boolean | Whether to ignore lineage failures      | `true`             |
| `error_handling.log_level`      | String  | Log level for lineage errors            | `ERROR`            |
| `context.include_metrics`       | Boolean | Include metrics in lineage events       | `true`             |
| `context.include_token_usage`   | Boolean | Include token usage in lineage events   | `true`             |
| `context.record_timestamps`     | Boolean | Include timestamps in lineage events    | `true`             |
| `retry.enabled`                 | Boolean | Whether to retry lineage operations     | `true`             |
| `retry.max_attempts`            | Integer | Maximum number of retry attempts        | `3`                |
| `retry.initial_delay`           | Integer | Initial delay between retries (seconds) | `1`                |
| `retry.max_delay`               | Integer | Maximum delay between retries (seconds) | `30`               |
| `retry.backoff_factor`          | Integer | Backoff multiplier for retries          | `2`                |
| `retry.retry_on`                | Array   | Error types to retry on                 | See example        |

Example:

```yaml
runtime:
  workflow:
    storage:
      enabled: true
      root_dir: "workspaces/workflows"
      format: "yymmdd_hhmm_{workflow_id}"
      retention:
        max_runs: 10
        max_days: 30
      error_handling:
        ignore_storage_errors: true
        log_level: "ERROR"
  lineage:
    enabled: true
    namespace: "c4h_agents"
    separate_input_output: true
    backend:
      type: "file"
      path: "workspaces/lineage"
    error_handling:
      ignore_failures: true
      log_level: "ERROR"
    context:
      include_metrics: true
      include_token_usage: true
      record_timestamps: true
    retry:
      enabled: true
      max_attempts: 3
      initial_delay: 1
      max_delay: 30
      backoff_factor: 2
      retry_on:
        - "overloaded_error"
        - "rate_limit_error"
        - "timeout_error"
```

## Backup Configuration

The `backup` section configures file backup behavior.

| Field     | Type    | Description                 | Default            |
| :-------- | :------ | :-------------------------- | :----------------- |
| `enabled` | Boolean | Whether backups are enabled | `true`             |
| `path`    | String  | Path for backup files       | `workspaces/backups` |

Example:

```yaml
backup:
  enabled: true
  path: "workspaces/backups"
```

## Logging Configuration

The `logging` section controls logging behavior.

| Field                       | Type    | Description                               | Default        |
| :-------------------------- | :------ | :---------------------------------------- | :------------- |
| `level`                     | String  | Global log level                          | `INFO`         |
| `format`                    | String  | Log format (`structured` or `plain`)      | `structured`   |
| `agent_level`               | String  | Log level for agent operations            | `INFO`         |
| `providers.anthropic.level` | String  | Log level for Anthropic provider          | `debug`        |
| `providers.openai.level`    | String  | Log level for OpenAI provider             | `debug`        |
| `truncate.prefix_length`    | Integer | Length of prefix when truncating log strings | `200`          |
| `truncate.suffix_length`    | Integer | Length of suffix when truncating log strings | `100`          |

Example:

```yaml
logging:
  level: "INFO"
  format: "structured"
  agent_level: "DEBUG"
  providers:
    anthropic:
      level: "debug"
    openai:
      level: "debug"
  truncate:
    prefix_length: 200
    suffix_length: 100
```

Following these configuration guidelines and adhering to the design principles will ensure your C4H Agent System operates efficiently and predictably.
