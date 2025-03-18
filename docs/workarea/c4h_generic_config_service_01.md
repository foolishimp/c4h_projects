# Updated C4H Editor Backend API Specification

## Overview

This specification extends the existing C4H Editor Backend API to support a configuration-driven approach for managing multiple configuration types (WorkOrder, TeamConfig, RuntimeConfig, etc.) while maintaining the same core architecture. The API will dynamically handle different configuration repositories based on config type.

## Base URL

```
http://localhost:8000
```

## Authentication

Authentication remains unchanged. The backend continues to use API keys for third-party services stored in environment variables.

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

## Enhanced API Endpoints

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

#### Delete Configuration

```
DELETE /api/v1/configs/{configType}/{configId}?commit_message={message}&author={author}
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration
- `commit_message`: Commit message for the deletion
- `author`: Author of the commit

#### Get Configuration History

```
GET /api/v1/configs/{configType}/{configId}/history
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Returns the version history of a configuration (if versioning is supported for the config type).

#### Archive/Unarchive Configuration

```
POST /api/v1/configs/{configType}/{configId}/archive
POST /api/v1/configs/{configType}/{configId}/unarchive
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Marks a configuration as archived or unarchived.

#### Clone Configuration

```
POST /api/v1/configs/{configType}/{configId}/clone
```

Parameters:
- `configType`: The type of configuration
- `configId`: The identifier of the configuration

Request Body (optional):
```json
{
  "new_id": "cloned-config-id"
}
```

Creates a new configuration based on an existing one.

### Enhanced Job Management

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

Response:

```json
{
  "id": "job-id",
  "configurations": {
    "workorder": {
      "id": "workorder-id",
      "version": "1.0.0"
    },
    "teamconfig": {
      "id": "teamconfig-id",
      "version": "1.0.0"
    },
    "runtimeconfig": {
      "id": "runtimeconfig-id",
      "version": "1.0.0"
    }
  },
  "status": "created",
  "service_job_id": null,
  "created_at": "2023-01-01T00:00:00.000000",
  "updated_at": "2023-01-01T00:00:00.000000",
  "submitted_at": null,
  "completed_at": null,
  "user_id": "user-id",
  "job_configuration": {
    "max_runtime": 3600,
    "notify_on_completion": true
  },
  "result": null
}
```

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

#### Get Job

```
GET /api/v1/jobs/{job_id}
```

Returns status and details of a specific job, including all associated configurations.

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

## Implementation Notes

1. The existing Git-based versioning system is reused but organized per configuration type.
2. Schema validation occurs against the JSON schema registered for each configuration type.
3. The API maintains backward compatibility with the original workorder endpoints for seamless transition.
4. Configuration types can be added or modified through the configuration registry without code changes.
5. Cross-configuration validation occurs during job submission to ensure compatibility between configurations.

## Error Handling

All API endpoints continue to return standard HTTP status codes with error details:

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

## Compatibility with Legacy Endpoints

To ensure backward compatibility, the original workorder-specific endpoints are maintained as aliases to the generic configuration endpoints:

```
GET /api/v1/workorders => GET /api/v1/configs/workorder
GET /api/v1/workorders/{id} => GET /api/v1/configs/workorder/{id}
```

This allows existing clients to continue functioning while new clients can use the generic, configuration-driven API.