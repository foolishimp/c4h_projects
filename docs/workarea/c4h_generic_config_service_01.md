# C4H Editor Backend API Specification

## Overview

This specification defines the backend API to support the C4H Editor, a modular configuration management system. The API provides a configuration-driven approach for managing multiple configuration types (WorkOrder, TeamConfig, RuntimeConfig, etc.) through a unified interface. The backend dynamically handles different configuration repositories based on config type.

## Base URL

```
http://localhost:8000
```

## Authentication

The backend uses API keys for third-party services stored in environment variables.

## Data Model

### Configuration Types
The system supports multiple configuration types:
- **WorkOrder**: Defines what needs to be done and against which asset
- **TeamConfig**: Defines agent teams and their capabilities
- **RuntimeConfig**: Manages operational aspects of the service

Each configuration type supports:
- Version control 
- Metadata (author, dates, description, tags)
- Archiving/unarchiving
- Cloning

### Job Model
A job represents a work unit that combines multiple configurations:
- Jobs consist of references to configurations (WorkOrder, TeamConfig, RuntimeConfig)
- Jobs progress through statuses: CREATED, SUBMITTED, RUNNING, COMPLETED, FAILED, CANCELLED
- Jobs track creation, submission, and completion times
- Jobs have configurable parameters (max runtime, notifications)
- Jobs produce results (output, artifacts, metrics, errors)

## Configuration Type Registry

The backend maintains a registry of supported configuration types. New types can be added through configuration without code changes.

```json
{
  "configTypes": [
    {
      "type": "workorder",
      "name": "Work Order",
      "description": "Defines what needs to be done and against which asset",
      "supportsVersioning": true,
      "schema": "schemas/workorder.json",
      "repository": {
        "type": "git",
        "path": "repositories/workorders"
      }
    },
    {
      "type": "teamconfig",
      "name": "Team Configuration",
      "description": "Defines the agent teams and their capabilities",
      "supportsVersioning": true,
      "schema": "schemas/teamconfig.json",
      "repository": {
        "type": "git",
        "path": "repositories/teamconfigs"
      }
    },
    {
      "type": "runtimeconfig",
      "name": "Runtime Configuration",
      "description": "Manages operational aspects of the C4H Service",
      "supportsVersioning": true,
      "schema": "schemas/runtimeconfig.json",
      "repository": {
        "type": "git",
        "path": "repositories/runtimeconfigs"
      }
    }
  ]
}
```

## API Endpoints

### Health Check

```
GET /health
```

Returns status information about the service, including registered configuration types.

```json
{
  "status": "healthy",
  "version": "0.2.0",
  "config_loaded": true,
  "services": {
    "repository": true,
    "lineage": true,
    "llm": true,
    "jobs": true,
    "c4h": true
  },
  "supported_config_types": ["workorder", "teamconfig", "runtimeconfig"]
}
```

### Configuration Types

```
GET /api/v1/config-types
```

Returns a list of all supported configuration types.

```json
[
  {
    "type": "workorder",
    "name": "Work Order",
    "description": "Defines what needs to be done and against which asset",
    "supportsVersioning": true
  },
  {
    "type": "teamconfig",
    "name": "Team Configuration",
    "description": "Defines the agent teams and their capabilities",
    "supportsVersioning": true
  },
  {
    "type": "runtimeconfig",
    "name": "Runtime Configuration",
    "description": "Manages operational aspects of the C4H Service",
    "supportsVersioning": true
  }
]
```

### Configuration Management

#### List Configurations

```
GET /api/v1/configs/{configType}
```

Parameters:
- `configType`: The type of configuration (e.g., "workorder")
- `archived` (optional): Filter by archived status (true/false)

Returns a list of all configurations of the specified type.

```json
[
  {
    "id": "config-id",
    "version": "1.0.0",
    "title": "Description of the configuration",
    "author": "author-name",
    "updated_at": "2023-01-01T00:00:00.000000",
    "last_commit": "commit-hash",
    "last_commit_message": "Commit message",
    "config_type": "workorder"
  }
]
```

#### Get Configuration

```
GET /api/v1/configs/{configType}/{configId}
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration
- `version` (optional): Specific version or commit hash to retrieve

Returns the specified configuration.

```json
{
  "id": "config-id",
  "version": "1.0.0",
  "config_type": "workorder",
  "content": {
    // Configuration content specific to the type
  },
  "metadata": {
    "author": "author-name",
    "archived": false,
    "created_at": "2023-01-01T00:00:00.000000",
    "updated_at": "2023-01-01T00:00:00.000000",
    "description": "Configuration description",
    "tags": ["tag1", "tag2"]
  },
  "commit": "commit-hash",
  "updated_at": "2023-01-01T00:00:00.000000"
}
```

#### Create Configuration

```
POST /api/v1/configs/{configType}
```

Parameters:
- `configType`: The type of configuration to create

Request Body:

```json
{
  "id": "new-config-id",
  "content": {
    // Configuration content specific to the type
  },
  "metadata": {
    "author": "author-name",
    "description": "Configuration description",
    "tags": ["tag1", "tag2"]
  },
  "commit_message": "Initial commit",
  "author": "author-name"
}
```

Response: The created configuration object.

#### Update Configuration

```
PUT /api/v1/configs/{configType}/{configId}
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Request Body:

```json
{
  "content": {
    // Updated configuration content
  },
  "metadata": {
    "author": "author-name",
    "description": "Updated description",
    "tags": ["tag1", "tag2"]
  },
  "commit_message": "Update configuration",
  "author": "author-name"
}
```

Response: The updated configuration object.

#### Delete Configuration

```
DELETE /api/v1/configs/{configType}/{configId}
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration
- `commit_message` (query): Commit message for the deletion
- `author` (query): Author of the commit

#### Get Configuration History

```
GET /api/v1/configs/{configType}/{configId}/history
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Returns the version history of a configuration.

```json
{
  "config_id": "config-id",
  "config_type": "workorder",
  "versions": [
    {
      "version": "1.0.0",
      "commit_hash": "commit-hash",
      "created_at": "2023-01-01T00:00:00.000000",
      "author": "author-name",
      "message": "Commit message"
    }
  ]
}
```

#### Archive Configuration

```
POST /api/v1/configs/{configType}/{configId}/archive
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Marks a configuration as archived.

#### Unarchive Configuration

```
POST /api/v1/configs/{configType}/{configId}/unarchive
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Marks a configuration as unarchived.

#### Clone Configuration

```
POST /api/v1/configs/{configType}/{configId}/clone
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Request Body:
```json
{
  "new_id": "cloned-config-id"
}
```

Creates a new configuration based on an existing one. Returns the cloned configuration.

### Job Management

#### List Jobs

```
GET /api/v1/jobs
```

Parameters:
- `config_id` (optional): Filter by configuration ID
- `config_type` (optional): Filter by configuration type
- `status` (optional): Filter by status
- `user_id` (optional): Filter by user ID
- `limit` (optional): Page size limit (default: 100)
- `offset` (optional): Page offset (default: 0)

Response:

```json
{
  "items": [
    {
      "id": "job-id",
      "configurations": {
        "workorder": { "id": "workorder-id", "version": "1.0.0" },
        "teamconfig": { "id": "teamconfig-id", "version": "1.0.0" },
        "runtimeconfig": { "id": "runtimeconfig-id", "version": "1.0.0" }
      },
      "status": "running",
      "service_job_id": "service-job-id",
      "created_at": "2023-01-01T00:00:00.000000",
      "updated_at": "2023-01-01T00:00:00.000000"
    }
  ],
  "total": 10,
  "limit": 100,
  "offset": 0
}
```

#### Get Job

```
GET /api/v1/jobs/{job_id}
```

Parameters:
- `job_id`: The job identifier

Returns status and details of a specific job, including all associated configurations.

```json
{
  "id": "job-id",
  "configurations": {
    "workorder": { "id": "workorder-id", "version": "1.0.0" },
    "teamconfig": { "id": "teamconfig-id", "version": "1.0.0" },
    "runtimeconfig": { "id": "runtimeconfig-id", "version": "1.0.0" }
  },
  "status": "running",
  "service_job_id": "service-job-id",
  "created_at": "2023-01-01T00:00:00.000000",
  "updated_at": "2023-01-01T00:00:00.000000",
  "submitted_at": "2023-01-01T00:00:30.000000",
  "completed_at": null,
  "user_id": "user-id",
  "job_configuration": {
    "max_runtime": 3600,
    "notify_on_completion": true
  },
  "result": null
}
```

#### Submit Job with Multiple Configs

```
POST /api/v1/jobs
```

Request Body:

```json
{
  "user_id": "user-id",
  "configurations": {
    "workorder": "workorder-id",
    "teamconfig": "teamconfig-id",
    "runtimeconfig": "runtimeconfig-id"
  },
  "job_configuration": {
    "max_runtime": 3600,
    "notify_on_completion": true
  }
}
```

Response: The created job object.

#### Cancel Job

```
POST /api/v1/jobs/{job_id}/cancel
```

Parameters:
- `job_id`: The job identifier

Cancels a running job. This is only valid if the job is in CREATED, SUBMITTED, or RUNNING state.

## Data Models

### Configuration

```typescript
interface Configuration {
  id: string;
  config_type: string;
  content: Record<string, any>;
  metadata: {
    author: string;
    archived: boolean;
    created_at: string;
    updated_at: string;
    description?: string;
    tags: string[];
    version: string;
  };
  parent_id?: string;
  lineage: string[];
}
```

### Job

```typescript
interface Job {
  id: string;
  configurations: Record<string, { id: string, version: string }>;
  status: "created" | "submitted" | "running" | "completed" | "failed" | "cancelled";
  service_job_id?: string;
  created_at: string;
  updated_at: string;
  submitted_at?: string;
  completed_at?: string;
  user_id?: string;
  job_configuration: Record<string, any>;
  result?: {
    output?: string;
    artifacts: any[];
    metrics: Record<string, any>;
    error?: string;
  };
}
```

## Repository Management

The backend dynamically manages repositories based on configuration types:

1. When a new configuration type is encountered, the service checks if a repository exists for that type.
2. If no repository exists, a new one is created at the path specified in the configuration type registry.
3. All CRUD operations for that configuration type are directed to the appropriate repository.

## Business Rules

1. Configurations should be stored with version control (Git-based storage recommended)
2. A job must include all required configuration types
3. Job execution should validate compatibility between configurations
4. Jobs should be cancelable only in certain states (CREATED, SUBMITTED, RUNNING)
5. API responses should include appropriate error details when operations fail

## Technical Requirements

1. Use RESTful endpoint design
2. Support JSON for all request/response formats
3. Implement standard HTTP response codes
4. All endpoints should include validation
5. Configuration content should support YAML format

## Error Handling

All API endpoints return standard HTTP status codes with error details:

```json
{
  "detail": "Error message",
  "config_type": "workorder",
  "validation_errors": [
    {
      "field": "field_name",
      "message": "Specific validation error"
    }
  ]
}
```

## Implementation Notes

1. The Git-based versioning system is organized per configuration type
2. Schema validation occurs against the JSON schema registered for each configuration type
3. Configuration types can be added or modified through the configuration registry without code changes
4. Cross-configuration validation occurs during job submission to ensure compatibility between configurations