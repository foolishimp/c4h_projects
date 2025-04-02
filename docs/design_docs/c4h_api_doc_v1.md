# C4H Services API Integration Guide (Updated)

## Introduction

This guide provides comprehensive details for integrating with the C4H Services API, a system designed for orchestrating intelligent code refactoring workflows. The API allows you to submit code refactoring intents and receive structured results, making it ideal for building GUI interfaces for code analysis and transformation.

## API Overview

The C4H Services exposes two primary API interfaces:

### Jobs API (Recommended)
- `POST /api/v1/jobs` - Submit a new job with structured configuration
- `GET /api/v1/jobs/{job_id}` - Check status of an existing job

### Workflow API (Legacy)
- `POST /api/v1/workflow` - Submit a new workflow request
- `GET /api/v1/workflow/{workflow_id}` - Check status of an existing workflow

### Utility Endpoints
- `GET /health` - Service health check

## Core Concepts

### Jobs vs. Workflows

The Jobs API is the recommended interface, providing a more structured configuration model with clear separation of concerns:
- **Workorder**: Contains project and intent information
- **Team**: Contains LLM and orchestration configuration
- **Runtime**: Contains runtime settings and environment config

The Workflow API is maintained for backward compatibility but offers less structure.

### Teams

The system uses a team-based approach where specialized agent teams handle different aspects of the workflow:
- **Discovery Team** - Analyzes project files and structure
- **Solution Team** - Designs changes based on intent
- **Coder Team** - Implements the designed changes
- **Fallback Team** - Handles failures with a simplified approach

## API Reference

### Submit Job (Recommended)

```
POST /api/v1/jobs
```

#### Request Body

```json
{
  "workorder": {
    "project": {
      "path": "/path/to/project",
      "workspace_root": "workspaces"
    },
    "intent": {
      "description": "Description of your refactoring intent",
      "target_files": ["optional/file/path.py"]
    }
  },
  "team": {
    "llm_config": {
      "default_provider": "anthropic",
      "default_model": "claude-3-5-sonnet-20241022"
    },
    "orchestration": {
      "enabled": true,
      "entry_team": "discovery"
    }
  },
  "runtime": {
    "logging": {
      "level": "INFO"
    },
    "backup": {
      "enabled": true
    }
  }
}
```

#### Response

```json
{
  "job_id": "job_20250402_123456_abcd1234",
  "status": "pending",
  "storage_path": "workspaces/lineage/wf_1234_abcd1234",
  "error": null
}
```

### Check Job Status

```
GET /api/v1/jobs/{job_id}
```

#### Response

```json
{
  "job_id": "job_20250402_123456_abcd1234",
  "status": "success",
  "storage_path": "workspaces/lineage/wf_1234_abcd1234",
  "error": null,
  "changes": [
    {
      "file": "/path/to/file.py",
      "change": {
        "type": "modify",
        "status": "success"
      }
    }
  ]
}
```

### Submit Workflow (Legacy)

```
POST /api/v1/workflow
```

#### Request Body

```json
{
  "project_path": "/path/to/project",
  "intent": {
    "description": "Description of your refactoring intent"
  },
  "app_config": {
    "key": "value"
  },
  "system_config": {
    "key": "value"
  },
  "lineage_file": "optional/path/to/lineage.json",
  "stage": "discovery",
  "keep_runid": true
}
```

#### Parameters for Workflow Continuation

| Field | Type | Description |
|-------|------|-------------|
| lineage_file | string | Path to lineage file for workflow continuation |
| stage | string | Stage to continue workflow from ("discovery", "solution_designer", "coder") |
| keep_runid | boolean | Whether to keep the original run ID from the lineage file (default: true) |

#### Response

```json
{
  "workflow_id": "wf_1234_abcd1234",
  "status": "pending",
  "storage_path": "workspaces/lineage/wf_1234_abcd1234",
  "error": null
}
```

### Health Check

```
GET /health
```

#### Response

```json
{
  "status": "healthy",
  "workflows_tracked": 5,
  "teams_available": 4
}
```

## Configuration Structure

The Jobs API uses a structured configuration with three main sections:

### Workorder Section

```yaml
workorder:
  project:
    path: "/path/to/project"
    workspace_root: "workspaces"
    source_root: "."
    output_root: "."
  intent:
    description: "Add logging to all functions with lineage tracking"
    target_files:
      - "src/main.py"
      - "src/utils.py"
```

### Team Section

```yaml
team:
  llm_config:
    default_provider: "anthropic"
    default_model: "claude-3-7-sonnet-20250219"
    providers:
      anthropic:
        api_base: "https://api.anthropic.com"
        context_length: 200000
        env_var: "ANTHROPIC_API_KEY"
        default_model: "claude-3-7-sonnet-20250219"
      openai:
        api_base: "https://api.openai.com/v1"
        env_var: "OPENAI_API_KEY"
        default_model: "o3-mini"
    agents:
      discovery:
        model: "claude-3-5-sonnet-20241022"
        temperature: 0
      solution_designer:
        model: "claude-3-7-sonnet-20250219"
        temperature: 0
      coder:
        model: "claude-3-7-sonnet-20250219"
        temperature: 0
        
  orchestration:
    enabled: true
    entry_team: "discovery"
    teams:
      discovery:
        name: "Discovery Team"
        tasks:
          - name: "discovery"
            agent_class: "c4h_agents.agents.discovery.DiscoveryAgent"
        routing:
          default: "solution"
```

### Runtime Section

```yaml
runtime:
  workflow:
    storage:
      enabled: true
      root_dir: "workspaces/workflows"
  lineage:
    enabled: true
    backend:
      type: "file"
      path: "workspaces/lineage"
  logging:
    level: "INFO"
    format: "structured"
  backup:
    enabled: true
    path: "workspaces/backups"
```

## Workflow Execution Flow

1. **Initialization**:
   - Client submits job/workflow request with project path and intent
   - Server generates a unique ID (workflow_id or job_id)
   - Server prepares configuration with default values

2. **Team Execution**:
   - Discovery team analyzes project structure using tartxt
   - Solution team designs code modifications
   - Coder team implements changes using semantic extraction and merging
   - If solution design fails, the fallback team handles using a more conservative approach

3. **Result Storage**:
   - Changes are written to the project files
   - Backup copies are created before modifications
   - Lineage information is recorded with execution paths
   - Status information is stored for retrieval

## Command Line Interface

The system provides a command-line interface through `prefect_runner.py`:

### Jobs Mode (Recommended)
```bash
python ./c4h_services/src/bootstrap/prefect_runner.py jobs -P 8000 --config jobs_config.yml
```

### Client Mode (Legacy)
```bash
python ./c4h_services/src/bootstrap/prefect_runner.py client --host localhost --port 8000 --project-path /path/to/project --intent-file intent.json --poll
```

### Service Mode (Start API Server)
```bash
python ./c4h_services/src/bootstrap/prefect_runner.py service --port 8000 --config config.yml
```

## Example Client Implementation

### Using Jobs API (Recommended)

```python
import requests
import time
import os.path

class C4HJobClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url
        
    def submit_job(self, workorder_config, team_config=None, runtime_config=None):
        """Submit a new job request"""
        url = f"{self.base_url}/api/v1/jobs"
        
        # Create job request
        payload = {
            "workorder": workorder_config
        }
        
        if team_config:
            payload["team"] = team_config
            
        if runtime_config:
            payload["runtime"] = runtime_config
            
        response = requests.post(url, json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_job_status(self, job_id):
        """Get status of a job"""
        url = f"{self.base_url}/api/v1/jobs/{job_id}"
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
    
    def wait_for_completion(self, job_id, interval=5, timeout=300, callback=None):
        """Wait for job completion with polling"""
        start_time = time.time()
        while time.time() - start_time < timeout:
            status = self.get_job_status(job_id)
            
            # Call the callback if provided
            if callback:
                callback(status)
                
            if status["status"] in ["success", "error", "complete", "failed"]:
                return status
                
            time.sleep(interval)
            
        raise TimeoutError(f"Job did not complete within {timeout} seconds")
```

### Using Workflow API (Legacy)

```python
import requests
import time
import os.path

class C4HWorkflowClient:
    def __init__(self, base_url="http://localhost:8000"):
        self.base_url = base_url
        
    def submit_workflow(self, project_path, intent_description, 
                        app_config=None, lineage_file=None, stage=None):
        """Submit a new workflow request"""
        url = f"{self.base_url}/api/v1/workflow"
        
        # Normalize project path
        project_path = os.path.abspath(project_path)
        
        payload = {
            "project_path": project_path,
            "intent": {
                "description": intent_description
            }
        }
        
        if app_config:
            payload["app_config"] = app_config
            
        if lineage_file and stage:
            payload["lineage_file"] = lineage_file
            payload["stage"] = stage
            
        response = requests.post(url, json=payload)
        response.raise_for_status()
        return response.json()
    
    def get_workflow_status(self, workflow_id):
        """Get status of a workflow"""
        url = f"{self.base_url}/api/v1/workflow/{workflow_id}"
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
```

## Common Use Cases

### Adding Logging (Jobs API)

```json
{
  "workorder": {
    "project": {
      "path": "./my_project"
    },
    "intent": {
      "description": "Add logging to all functions with proper error handling and level-appropriate log messages"
    }
  }
}
```

### Implementing Design Patterns (Jobs API)

```json
{
  "workorder": {
    "project": {
      "path": "./my_project"
    },
    "intent": {
      "description": "Refactor to use the Factory pattern for class creation in the user module"
    }
  },
  "team": {
    "llm_config": {
      "default_model": "claude-3-opus-20240229"
    }
  }
}
```

### Continuing a Workflow from Lineage (Workflow API)

```json
{
  "project_path": "./my_project",
  "intent": {
    "description": "Original intent description"
  },
  "lineage_file": "workspaces/lineage/wf_1234_abcd1234/events/123456_event_id.json",
  "stage": "solution_designer",
  "keep_runid": true
}
```

## Mapping Between APIs

The system internally maps between Jobs and Workflow APIs:

1. The Jobs API transforms structured configuration into the Workflow API format.
2. `workorder.project.path` becomes `project_path`
3. `workorder.intent` becomes `intent`
4. All configuration sections are flattened into `app_config`
5. Job IDs map to workflow IDs internally but maintain their own format

## Error Handling

Common errors and their solutions:

| Error | Possible Cause | Solution |
|-------|----------------|----------|
| "No project path specified" | Missing workorder.project.path | Provide valid project path |
| "Invalid job configuration" | Malformed request structure | Check configuration against the models |
| "Job creation failed" | Server error during processing | Check server logs for details |
| "Team not found" | Invalid entry_team | Verify orchestration.entry_team matches available teams |
| "No input paths configured" | Missing tartxt_config.input_paths | Ensure discovery agent has tartxt_config |
| "Missing agent_class" | Incorrect task configuration | Ensure all tasks have valid agent_class defined |

## Security Considerations

- The service operates on the filesystem, ensure proper isolation
- Consider running in a container or restricted environment
- Validate project paths to prevent path traversal attacks
- Implement authentication for multi-user environments
- Use dotenv files for API keys instead of embedding in configuration