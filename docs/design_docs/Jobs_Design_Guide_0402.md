# C4H Agent System: Jobs Configuration Guide

## Overview

A Jobs configuration file in the C4H Agent System consists of three core sections:

1. **Workorder** - Defines what work to perform
2. **Team** - Configures LLM and orchestration settings
3. **Runtime** - Sets execution environment parameters

These sections together form a complete job request that can be submitted to the C4H Agent API.

## 1. Workorder Section

The workorder section defines project details and the intent description.

```yaml
workorder:
  project:
    path: "/path/to/your/project"  # Required - absolute or relative path
    workspace_root: "workspaces"    # Optional - workspace directory
    source_root: "src"              # Optional - source code directory
    output_root: "output"           # Optional - output directory
  
  intent:
    description: |
      # Required - detailed description of the task
      Explain clearly what changes are needed.
      
      Be specific about:
      1. Goal of the task
      2. Required changes
      3. Implementation requirements
    
    target_files:                    # Optional - specific files to target
      - "src/file1.py"
      - "src/file2.py"
```

### Project Configuration

| Field | Description | Default |
|-------|-------------|---------|
| `path` | Path to project root | Required |
| `workspace_root` | Directory for working files | `workspaces` |
| `source_root` | Source code directory | `.` |
| `output_root` | Output directory | `.` |

### Intent Configuration

| Field | Description | Required |
|-------|-------------|----------|
| `description` | Detailed description of the work | Yes |
| `target_files` | Specific files to target | No |

### Best Practices for Intent Description

1. **Start with a clear goal statement**
2. **Use structured sections**:
   - Goal/Objective
   - Required Changes
   - Implementation Requirements
   - Files to Modify
3. **Be specific about patterns to follow**
4. **Specify what should NOT change**
5. **Provide examples when helpful**

## 2. Team Section

The team section configures LLM providers and agent settings.

```yaml
team:
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
        # Additional provider settings...
      
      openai:
        # OpenAI provider settings...
        
    default_provider: "anthropic"
    default_model: "claude-3-7-sonnet-20250219"
    
    agents:
      discovery:
        temperature: 0
        tartxt_config:
          script_path: "./c4h_agents/skills/tartxt.py"
          input_paths: ["./"]
          exclusions: ["**/__pycache__/**"]
      
      solution_designer:
        provider: "anthropic"
        model: "claude-3-7-sonnet-20250219"
        temperature: 1
        extended_thinking:
          enabled: true
          budget_tokens: 32000
        # Optional prompt overrides...
      
      # Other agent configurations...
  
  orchestration:
    enabled: true
    entry_team: "discovery"
    # Optional team definitions and routing...
```

### Provider Configuration

| Field | Description | Notes |
|-------|-------------|-------|
| `api_base` | Base URL for provider API | Provider-specific |
| `context_length` | Maximum token context length | Provider-specific |
| `env_var` | Environment variable for API key | Provider-specific |
| `default_model` | Default model for this provider | Provider-specific |
| `valid_models` | Supported models list | Provider-specific |
| `extended_thinking` | Configuration for Claude's thinking mode | Claude models only |
| `litellm_params` | LiteLLM configuration parameters | Optional |

### Agent Configuration

Key agent configurations include:

1. **Discovery** - Project analysis agent
   - `tartxt_config.script_path`: Path to tartxt.py analyzer
   - `tartxt_config.input_paths`: Directories to analyze
   - `tartxt_config.exclusions`: Patterns to exclude

2. **Solution Designer** - Creates refactoring plan
   - `provider`: LLM provider to use
   - `model`: Specific model name
   - `temperature`: Creativity level (0-1)
   - `extended_thinking`: Configuration for Claude's thinking capability

3. **Coder** - Implements the changes
   - Similar settings to Solution Designer
   - `backup_enabled`: Whether to backup files before changes

4. **Lineage** - Configuration for tracking and observability
   - `enabled`: Whether lineage tracking is enabled
   - `namespace`: Namespace for lineage events
   - `backend`: Configuration for lineage storage

## 3. Runtime Section

The runtime section configures execution environment settings.

```yaml
runtime:
  workflow:
    storage:
      enabled: true
      root_dir: "workspaces/workflows"
      format: "yymmdd_hhmm_{workflow_id}"
      # Additional storage settings...
  
  lineage:
    enabled: true
    namespace: "c4h_workflow"
    backend:
      type: "file"
      path: "workspaces/lineage"
    # Additional lineage settings...
  
  backup:
    enabled: true
    path: "./workspaces/backups"
  
  logging:
    level: "INFO"
    format: "structured"
    agent_level: "DEBUG"
    truncate:
      prefix_length: 100
      suffix_length: 100
```

### Workflow Storage

| Field | Description | Default |
|-------|-------------|---------|
| `enabled` | Whether workflow storage is enabled | `true` |
| `root_dir` | Root directory for workflow storage | `workspaces/workflows` |
| `format` | Format string for workflow directories | `yymmdd_hhmm_{workflow_id}` |
| `retention.max_runs` | Maximum number of runs to keep | `10` |
| `retention.max_days` | Maximum age for stored data in days | `30` |

### Lineage Tracking

| Field | Description | Default |
|-------|-------------|---------|
| `enabled` | Whether lineage tracking is enabled | `true` |
| `namespace` | Namespace for lineage events | `c4h_agents` |
| `backend.type` | Backend type (`file` or `marquez`) | `file` |
| `backend.path` | Path for file backend | `workspaces/lineage` |
| `context.include_metrics` | Include metrics in lineage events | `true` |
| `context.include_token_usage` | Include token usage in lineage events | `true` |

### Backup Configuration

| Field | Description | Default |
|-------|-------------|---------|
| `enabled` | Whether backups are enabled | `true` |
| `path` | Path for backup files | `workspaces/backups` |

### Logging Configuration

| Field | Description | Default |
|-------|-------------|---------|
| `level` | Global log level | `INFO` |
| `format` | Log format (`structured` or `plain`) | `structured` |
| `agent_level` | Log level for agent operations | `INFO` |
| `truncate.prefix_length` | Length of prefix when truncating log strings | `30` |
| `truncate.suffix_length` | Length of suffix when truncating log strings | `30` |

## Job Execution Flow

When a job is submitted:

1. **Discovery** analyzes the project structure
2. **Solution Designer** creates a detailed plan based on the intent
3. **Coder** implements the changes according to the plan
4. (Optional) **Assurance** validates the changes

## Common Job Examples

### Example: Adding Logging Framework

```yaml
workorder:
  project:
    path: "./my_project"
  intent:
    description: |
      Replace print statements with structured logging.
      
      Goal: 
      Implement a consistent logging system across the codebase.
      
      Required Changes:
      1. Add logging configuration
      2. Replace all print statements with appropriate log levels
      3. Include context in log messages
      
      Implementation Requirements:
      - Use Python's logging module
      - Configure log formatting for different environments
      - Add file and console handlers

team:
  llm_config:
    default_provider: "anthropic"
    default_model: "claude-3-7-sonnet-20250219"
    agents:
      solution_designer:
        extended_thinking:
          enabled: true

runtime:
  logging:
    level: "DEBUG"
  backup:
    enabled: true
```

### Example: Performance Optimization

```yaml
workorder:
  project:
    path: "./my_project"
  intent:
    description: |
      Optimize database query performance.
      
      Goal:
      Reduce query execution time by 50%.
      
      Required Changes:
      1. Add indexes to frequently queried fields
      2. Optimize SELECT queries to retrieve only needed fields
      3. Add query result caching
      
      Files to Modify:
      - src/data/repositories/*.py
      - src/models/*.py

team:
  llm_config:
    default_provider: "anthropic"
    default_model: "claude-3-opus-20240229"
    agents:
      discovery:
        tartxt_config:
          input_paths: ["./src/data", "./src/models"]

runtime:
  lineage:
    enabled: true
```

## Best Practices

1. **Provide clear intent description** with specific goals and requirements
2. **Use the right model** for your task (higher capability models for complex tasks)
3. **Enable extended thinking** for Solution Designer on complex refactoring
4. **Configure appropriate logging level** based on debugging needs
5. **Always enable backups** when making substantial changes
6. **Specify correct tartxt paths** to ensure proper project analysis
7. **Use lineage tracking** for observability and debugging