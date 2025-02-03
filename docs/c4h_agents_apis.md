# C4H Agents Library API Documentation

## Overview

The C4H Agents library provides a framework for LLM-powered code refactoring with a focus on minimal agent logic and LLM-first processing. This documentation is designed to help LLMs understand and extend the library's capabilities.

## Core Components

### 1. BaseAgent

The foundational class for all agents providing LLM interaction capabilities.

```python
from c4h_agents.agents.base import BaseAgent, AgentResponse

class CustomAgent(BaseAgent):
    def __init__(self, config: Dict[str, Any] = None):
        super().__init__(config=config)
    
    def _get_agent_name(self) -> str:
        return "custom_agent"
    
    def process(self, context: Dict[str, Any]) -> AgentResponse:
        # Implement agent logic here
        pass
```

Key Features:
- Automatic LLM provider configuration
- Continuation handling for long responses
- Consistent logging and metrics
- Project context awareness
- Standardized response format

Configuration Requirements:
```yaml
llm_config:
  agents:
    agent_name:  # Replace with actual agent name
      provider: "anthropic"  # or "openai", "gemini"
      model: "claude-3-opus-20240229"  # Model identifier
      temperature: 0
      prompts:
        system: "System prompt for agent context"
        # Additional prompt templates as needed
```

### 2. Discovery Agent

Analyzes project structure and files.

```python
from c4h_agents.agents.discovery import DiscoveryAgent

discovery = DiscoveryAgent(config={...})
result = discovery.process({
    "project_path": "/path/to/project"
})
```

Configuration Requirements:
```yaml
llm_config:
  agents:
    discovery:
      tartxt_config:
        script_path: "path/to/tartxt.py"
        input_paths: ["src", "tests"]
        exclusions: ["**/__pycache__/**"]
        output_type: "stdout"
```

Response Format:
```python
{
    "success": bool,
    "data": {
        "files": Dict[str, bool],  # Paths and existence
        "raw_output": str,        # Complete analysis
        "project_path": str,
        "timestamp": str
    },
    "error": Optional[str]
}
```

### 3. Solution Designer

Creates refactoring solutions based on intent and discovery analysis.

```python
from c4h_agents.agents.solution_designer import SolutionDesigner

designer = SolutionDesigner(config={...})
result = designer.process({
    "input_data": {
        "discovery_data": discovery_result.data,
        "intent": intent_description
    }
})
```

Configuration Requirements:
```yaml
llm_config:
  agents:
    solution_designer:
      intent:
        description: "Intent description"
      prompts:
        system: "System prompt"
        solution: "Solution template"
```

Response Format:
```python
{
    "success": bool,
    "data": {
        "changes": [
            {
                "file_path": str,
                "type": "create|modify|delete",
                "description": str,
                "content": str
            }
        ]
    },
    "error": Optional[str]
}
```

### 4. Coder Agent

Implements code changes safely with backup support.

```python
from c4h_agents.agents.coder import Coder

coder = Coder(config={...})
result = coder.process({
    "input_data": solution_result.data
})
```

Configuration Requirements:
```yaml
llm_config:
  agents:
    coder:
      backup_enabled: true
      backup:
        path: "workspaces/backups"
```

Response Format:
```python
{
    "success": bool,
    "data": {
        "changes": [
            {
                "file": str,
                "success": bool,
                "error": Optional[str],
                "backup": Optional[str]
            }
        ],
        "metrics": {
            "total_changes": int,
            "successful_changes": int,
            "failed_changes": int,
            "error_count": int,
            "processing_time": float
        }
    },
    "error": Optional[str]
}
```

### 5. Semantic Skills

#### SemanticIterator
Extracts structured information using fast or slow modes.

```python
from c4h_agents.skills.semantic_iterator import SemanticIterator
from c4h_agents.skills.shared.types import ExtractConfig

iterator = SemanticIterator(config={...})
iterator.configure(
    content=content,
    config=ExtractConfig(
        instruction="Extraction instruction",
        format="json"
    )
)

for item in iterator:
    # Process extracted items
    pass
```

Configuration:
```yaml
llm_config:
  agents:
    semantic_iterator:
      extractor_config:
        mode: "fast|slow"
        allow_fallback: true
```

#### SemanticMerge
Handles code merging with formatting preservation.

```python
from c4h_agents.skills.semantic_merge import SemanticMerge

merger = SemanticMerge(config={...})
result = merger.process({
    "file_path": str,
    "original": str,
    "diff": str
})
```

Configuration:
```yaml
llm_config:
  agents:
    semantic_merge:
      merge_config:
        preserve_formatting: true
        allow_partial: false
```

### 6. Project Management

```python
from c4h_agents.core.project import Project, ProjectPaths

project = Project.from_config({
    "project": {
        "path": "/path/to/project",
        "workspace_root": "workspaces",
        "name": "project_name",
        "version": "1.0.0"
    }
})
```

Project paths provided:
- root: Project root directory
- workspace: Working files directory
- source: Source code directory
- output: Output directory
- config: Configuration directory

## Best Practices

1. Configuration Management
- Always provide complete configuration
- Use system_config.yml for defaults
- Override specific settings in application config

2. Error Handling
- Check AgentResponse.success
- Log errors appropriately
- Handle backup failures gracefully

3. Project Context
- Use Project instance when available
- Resolve paths through project
- Maintain workspace structure

4. LLM Integration
- Follow provider-specific settings
- Handle continuations automatically
- Use appropriate temperature settings

5. Agent Design
- Keep agents focused and minimal
- Let LLM handle complex logic
- Use standardized response formats

## Common Patterns

1. Basic Agent Flow:
```python
agent = AgentClass(config=config)
result = agent.process(context)
if result.success:
    process_result(result.data)
else:
    handle_error(result.error)
```

2. Iterator Pattern:
```python
iterator = SemanticIterator(config=config)
iterator.configure(content=content, config=extract_config)
results = [item for item in iterator]
```

3. Project-Aware Pattern:
```python
project = Project.from_config(config)
agent = AgentClass(config=config)
result = agent.process({
    "project": project,
    "input_data": data
})
```

4. Safe Code Changes:
```python
coder = Coder(config=config)
result = coder.process({
    "input_data": {
        "changes": changes,
        "project": project
    }
})
if result.success:
    verify_changes(result.data["changes"])
```

## Extension Points

1. Custom Agents:
- Inherit from BaseAgent
- Implement _get_agent_name()
- Override process() method

2. New Skills:
- Follow semantic skill patterns
- Maintain stateless operation
- Use standard configurations

3. Custom Extractors:
- Extend SemanticIterator
- Implement extraction logic
- Follow iterator protocol

4. Project Extensions:
- Extend Project class
- Add custom path resolution
- Maintain path abstractions