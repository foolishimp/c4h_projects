# C4H Services API Integration Guide (v1.1 - Updated)

## Introduction

This guide provides comprehensive details for integrating with the C4H Services API, a system designed for orchestrating intelligent code refactoring workflows. The API allows you to submit code refactoring intents via structured job requests and receive results, making it ideal for building GUI interfaces or programmatic clients for code analysis and transformation.

## API Overview

The C4H Services exposes the following primary API interface:

### Jobs API (Current Standard)

* `POST /api/v1/jobs` - Submit a new job with one or more configuration fragments for server-side merging.
* `GET /api/v1/jobs/{job_id}` - Check status and retrieve results of an existing job.
* `GET /api/v1/jobs` - List existing jobs with optional filtering and pagination.
* `POST /api/v1/jobs/{job_id}/cancel` - Attempt to cancel a running job (best-effort).

### Utility Endpoints

* `GET /health` - Service health check.
* `GET /api/v1/logs` - Retrieve system logs with filtering options.
* `POST /api/v1/configs/merge` - Utility endpoint to test configuration merging.
* `GET /api/v1/config-types` - Retrieve the list of supported configuration types.

### Legacy Workflow API (Deprecated)

The `/api/v1/workflow` endpoints are deprecated and maintained only for backward compatibility. **New integrations must use the Jobs API (`/api/v1/jobs`).**

## Core Concepts

### Multi-Config Job Submission

The current standard (`POST /api/v1/jobs`) uses a **multi-config** approach. Instead of sending rigidly structured `workorder`, `team`, and `runtime` objects, the client sends a list of configuration dictionaries (fragments) under a top-level `configs` key.

The server performs a **deep merge** of these fragments onto a base system configuration. Fragments are merged sequentially from the list, with later fragments overriding earlier ones (e.g., the last fragment in the list has the highest priority). This allows for flexible configuration overrides (e.g., providing job-specific intent alongside standard team/runtime configs).

### Teams

The system uses a team-based approach where specialized agent teams handle different aspects of the workflow:

* **Discovery Team** - Analyzes project files and structure
* **Solution Team** - Designs changes based on intent
* **Coder Team** - Implements the designed changes
* **Observability Team** - Tracks lineage and logs across the workflow
    * Monitors execution flow
    * Captures decision points
    * Provides traceability for debugging
* **Fallback Team** - Handles failures with a simplified approach

## API Reference

### Submit Job (Multi-Config)

```
POST /api/v1/jobs
```

Submits a new job by providing a list of configuration fragments to be merged server-side.

#### Request Body

```json
{
  "configs": [
    // Base or shared configurations (lower priority)
    {
      "runtime": {
         // Default runtime settings
         "logging": { "level": "INFO", "format": "structured"},
         "backup": { "enabled": true, "path": "workspaces/backups" }
      },
      "team": {
         // Default team/LLM settings
         "llm_config": {
             "default_provider": "anthropic",
             "default_model": "claude-3-5-sonnet-20241022",
             "providers": { /* ... provider details ... */ }
         },
         "orchestration": { /* ... orchestration details ... */ }
      }
    },
    // Job-specific configurations (higher priority)
    {
      "workorder": {
        "project": {
          "path": "/path/to/project",
          "workspace_root": "workspaces"
        },
        "intent": {
          "description": "Specific intent for this job",
          "target_files": ["src/file.py"]
        }
      },
      "team": {
          "llm_config": {
              "agents": {
                  "solution_designer": {
                      "model": "claude-3-opus-20240229" // Override model for this job
                  }
              }
          }
      }
    }
    // Add more fragments as needed
  ]
}
```

#### Response

```json
{
  "job_id": "job_20250414_110500_f3a1b2c5",
  "status": "submitted", // Or 'created' depending on immediate processing
  "storage_path": "workspaces/lineage/wf_...", // Path if lineage enabled
  "error": null
}
```

### Check Job Status

```
GET /api/v1/jobs/{job_id}
```

Retrieves the current status and results (if completed) for a specific job.

#### Response (`JobStatus` model)

```json
{
  "job_id": "job_20250414_110500_f3a1b2c5",
  "status": "success", // e.g., created, submitted, running, success, failed, cancelled
  "storage_path": "workspaces/lineage/wf_...", // Path to lineage/results if enabled
  "error": null, // Error message if status is 'failed' or 'error'
  "changes": [ // Populated when job completes successfully
    {
      "file": "/path/to/project/src/file.py",
      "change": { // Can contain diff, status, error, type etc. from Coder
        "type": "modify",
        "status": "success"
        // Potentially other details like 'diff' or 'backup_path' might be included
      }
    }
  ]
}
```

### List Jobs

```
GET /api/v1/jobs
```

This endpoint returns a list of all jobs, with optional pagination and filtering.

#### Query Parameters

| Parameter | Type    | Description                                     |
| :-------- | :------ | :---------------------------------------------- |
| `page`    | integer | Page number (default: 1)                        |
| `limit`   | integer | Items per page (default: 25)                    |
| `status`  | string  | Filter by status (e.g., "submitted", "success") |

#### Response

```json
{
  "items": [
    {
        "job_id": "job_20250414_110500_f3a1b2c5",
        "status": "success",
        "created_at": "2025-04-14T11:05:00.123Z",
        "updated_at": "2025-04-14T11:15:30.456Z",
        "configurations": { // Shows the IDs of the configs used
            "workorder": {"id": "wo-123", "version": "latest"},
            "teamconfig": {"id": "team-default", "version": "1.2"},
            "runtimeconfig": {"id": "rt-prod", "version": "latest"}
        },
        "user_id": "user@example.com"
    }
    // ... other job items
  ],
  "total": 42, // Total number of jobs matching filters
  "page": 1,
  "limit": 25
}
```

### Cancel Job

```
POST /api/v1/jobs/{job_id}/cancel
```
Attempts to cancel a job that is currently in a `submitted` or `running` state. Cancellation is best-effort.

#### Response

```json
{
  "job_id": "job_20250414_110500_f3a1b2c5",
  "status": "cancelled", // Or the status at the time of cancellation attempt
  "storage_path": "workspaces/lineage/wf_...",
  "error": null // May contain an error if cancellation failed immediately
}

```

### Retrieve Logs

```
GET /api/v1/logs?job_id={job_id}&level=INFO&start_time=2025-04-02T12:00:00Z
```

#### Query Parameters

| Parameter    | Type         | Description                                         |
| :----------- | :----------- | :-------------------------------------------------- |
| `job_id`     | string       | Filter logs by job ID                               |
| `level`      | string       | Filter by log level (DEBUG, INFO, WARNING, ERROR) |
| `start_time` | ISO datetime | Filter logs after this time                         |
| `end_time`   | ISO datetime | Filter logs before this time                        |
| `limit`      | integer      | Maximum number of log entries to return             |

#### Response

```json
{
  "logs": [
    {
        "timestamp": "2025-04-14T11:05:01Z",
        "level": "INFO",
        "message": "Job started",
        "job_id": "job_20250414_110500_f3a1b2c5",
        "workflow_id": "wf_...",
        "component": "orchestrator"
        // ... other log entries ...
    }
  ]
}
```

### Merge Configurations Utility

```
POST /api/v1/configs/merge
```

Utility endpoint to test how configuration fragments will be merged by the server. Useful for debugging configuration overrides.

#### Request Body

```json
{
    "configs": [
        // List of config dicts to merge (later items override earlier ones)
        {"runtime": {"logging": {"level": "INFO"}}},
        {"runtime": {"logging": {"level": "DEBUG", "format": "structured"}}}
    ],
    "include_system_config": true // Optional: Merge onto base system config (default: true)
}
```

#### Response (`MergeResponse` model)

```json
{
    "merged_config": {
        // ... the resulting merged dictionary ...
        "runtime": {
            "logging": {
                "level": "DEBUG", // Overridden by the last fragment
                "format": "structured" // Added by the last fragment
            }
        }
        // ... other merged sections ...
    }
}
```

### Health Check

```
GET /health
```

Provides a simple health status check for the API service.

#### Response

```json
{
  "status": "healthy",
  "version": "1.1.0", // Example version
  "workflows_tracked": 15, // Example count
  "teams_available": 4 // Example count
}
```

### Get Configuration Types

```
GET /api/v1/config-types
```

Retrieves the list of configuration types known and managed by the backend (e.g., workorder, teamconfig).

#### Response

```json
[
    {
        "type": "workorder",
        "name": "Work Order",
        "description": "Defines what needs to be done...",
        "supportsVersioning": true
    },
    {
        "type": "teamconfig",
        "name": "Team Configuration",
        "description": "Defines agent teams...",
        "supportsVersioning": true
    },
    {
        "type": "runtimeconfig",
        "name": "Runtime Configuration",
        "description": "Manages operational aspects...",
        "supportsVersioning": true
    }
]
```

## Configuration Structure

While the API accepts fragments in the `configs` list, these fragments typically correspond to sections of the overall configuration structure used internally (Workorder, Team, Runtime). Understanding this structure helps in creating effective fragments. See the `SystemConfig_Design_Guide.md` for full details on the internal configuration structure.

## Frontend Job API Implementation

The frontend implementation for the Jobs API follows these design patterns:

### 1. Job Context Management

```tsx
// JobContext.tsx
import React, { createContext, useState, useRef, useCallback } from 'react';
import { Job, JobRequest, JobListResponse, LogEntry, LogOptions, ApiError, JobStatus } from './types'; // Assuming types are defined
import { apiService } from './apiService'; // Assuming a service layer exists

interface JobContextType {
  currentJob: Job | null;
  jobHistory: Job[];
  createJob: (request: JobRequest) => Promise<Job>; // Use JobRequest here
  getJobStatus: (jobId: string) => Promise<Job>;
  pollJobStatus: (jobId: string) => void;
  listJobs: (page?: number, limit?: number, status?: string) => Promise<JobListResponse>;
  stopPolling: (jobId: string) => void;
  getLogs: (jobId: string, options?: LogOptions) => Promise<LogEntry[]>;
}

const JobContext = createContext<JobContextType>(null!);

export const JobProvider: React.FC<{children: React.ReactNode}> = ({ children }) => {
  const [currentJob, setCurrentJob] = useState<Job | null>(null);
  const [jobHistory, setJobHistory] = useState<Job[]>([]);
  const pollingRefs = useRef<{[key: string]: number}>({});
  const [jobLogs, setJobLogs] = useState<{[key: string]: LogEntry[]}>({});

  // Create a new job using the multi-config endpoint
  const createJob = async (configs: Record<string, any>[]): Promise<Job> => {
    try {
      // Use the multi-config API function
      const newJob = await apiService.createJobMultiConfig(configs);
      setCurrentJob(newJob);
      setJobHistory(prev => [newJob, ...prev]);
      // Optionally start polling or log retrieval
      // startLogPolling(newJob.job_id);
      return newJob;
    } catch (error) {
      console.error("Failed to create job:", error);
      throw error;
    }
  };

  // Get a list of jobs with pagination and filtering
  const listJobs = async (page = 1, limit = 25, status?: string): Promise<JobListResponse> => {
    try {
      const params = new URLSearchParams();
      params.append('page', page.toString());
      params.append('limit', limit.toString());
      if (status) params.append('status', status);

      return await apiService.listJobs(params);
    } catch (error) {
      console.error("Failed to list jobs:", error);
      throw error;
    }
  };

  // Get status of a specific job
  const getJobStatus = async (jobId: string): Promise<Job> => {
    try {
        const jobData = await apiService.getJobStatus(jobId); // Assuming apiService has this
        // Update local state if needed
        if (currentJob?.job_id === jobId) {
            setCurrentJob(jobData);
        }
        setJobHistory(prev => prev.map(j => j.job_id === jobId ? jobData : j));
        return jobData;
    } catch (error) {
        console.error(`Failed to get status for job ${jobId}:`, error);
        throw error;
    }
  };

  // Poll job status
  const pollJobStatus = useCallback((jobId: string) => {
    if (pollingRefs.current[jobId]) {
        console.log(`Polling already active for job ${jobId}`);
        return;
    }
    const intervalId = window.setInterval(async () => {
        try {
            const jobData = await getJobStatus(jobId);
            if (['success', 'failed', 'error', 'complete', 'cancelled'].includes(jobData.status)) {
                stopPolling(jobId);
            }
        } catch (error) {
            console.error(`Polling error for job ${jobId}:`, error);
            stopPolling(jobId); // Stop polling on error
        }
    }, 5000); // Poll every 5 seconds
    pollingRefs.current[jobId] = intervalId;
  }, [getJobStatus]); // Include getJobStatus in dependencies

  // Stop polling for a job
  const stopPolling = useCallback((jobId: string) => {
    if (pollingRefs.current[jobId]) {
        clearInterval(pollingRefs.current[jobId]);
        delete pollingRefs.current[jobId];
        console.log(`Stopped polling for job ${jobId}`);
    }
  }, []);

  // Fetch logs for a job
  const getLogs = async (jobId: string, options?: LogOptions): Promise<LogEntry[]> => {
      try {
          const logs = await apiService.getLogs(jobId, options); // Assuming apiService has this
          setJobLogs(prev => ({ ...prev, [jobId]: logs }));
          return logs;
      } catch (error) {
          console.error(`Failed to fetch logs for job ${jobId}:`, error);
          throw error;
      }
  };


  // Provide context value
  const value = {
    currentJob,
    jobHistory,
    createJob, // Updated function name if changed in context
    getJobStatus,
    pollJobStatus,
    listJobs,
    stopPolling,
    getLogs
  };

  return <JobContext.Provider value={value}>{children}</JobContext.Provider>;
};
```

### 2. Type Definitions

```typescript
// job.ts (Example types)
export interface JobConfigReference {
  id: string;
  config_type: string;
  version?: string;
}

export interface MultiConfigJobRequest {
  configs: Record<string, any>[]; // List of config fragments
  user_id?: string;
  job_configuration?: Record<string, any>;
}

export interface Job {
  job_id: string;
  status: JobStatus;
  storage_path?: string;
  error?: string | null;
  changes?: JobChange[];
  // Add other fields returned by GET /jobs/{job_id} like configurations, created_at etc.
  configurations?: Record<string, { id: string; version: string }>;
  created_at?: string;
  updated_at?: string;
}

export interface JobChange {
    file: string;
    change: {
        type?: string;
        status?: string;
        error?: string;
        // potentially diff or other details
    };
}

export interface JobListResponse {
  items: Job[];
  total: number;
  page?: number;
  limit?: number;
}

export interface LogEntry {
  timestamp: string;
  level: 'DEBUG' | 'INFO' | 'WARNING' | 'ERROR';
  message: string;
  job_id: string;
  workflow_id?: string;
  component?: string;
  // Add other potential log fields
}

export interface LogOptions {
    level?: 'DEBUG' | 'INFO' | 'WARNING' | 'ERROR';
    startTime?: Date;
    endTime?: Date;
    limit?: number;
}

export type JobStatus = 'created' | 'submitted' | 'running' | 'success' | 'failed' | 'error' | 'complete' | 'cancelled'; // Expanded statuses

// Define ApiError if not already defined
export class ApiError extends Error {
  status: number;
  constructor(status: number, message: string) {
    super(message);
    this.status = status;
    this.name = 'ApiError';
  }
}
```

### 3. API Service Implementation

```typescript
// apiService.ts
import { Job, MultiConfigJobRequest, JobListResponse, LogEntry, LogOptions, ApiError } from './types'; // Adjust import path

const API_BASE_URL = 'http://localhost:8000'; // Or from environment variable

export const apiService = {
  createJobMultiConfig: async (configs: Record<string, any>[]): Promise<Job> => {
    const payload = { configs };
    const response = await fetch(`${API_BASE_URL}/api/v1/jobs`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({ error: 'Failed to parse error response' }));
      throw new ApiError(response.status, errorData?.error || `HTTP error ${response.status}`);
    }
    return await response.json();
  },

  listJobs: async (params: URLSearchParams): Promise<JobListResponse> => {
    const response = await fetch(`${API_BASE_URL}/api/v1/jobs?${params.toString()}`);
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({ error: 'Failed to parse error response' }));
      throw new ApiError(response.status, errorData?.error || 'Failed to list jobs');
    }
    return await response.json();
  },

  getJobStatus: async (jobId: string): Promise<Job> => {
      const response = await fetch(`${API_BASE_URL}/api/v1/jobs/${jobId}`);
      if (!response.ok) {
          const errorData = await response.json().catch(() => ({ error: 'Failed to parse error response' }));
          throw new ApiError(response.status, errorData?.error || `Failed to get status for job ${jobId}`);
      }
      return await response.json();
  },

  getLogs: async (jobId: string, options?: LogOptions): Promise<LogEntry[]> => {
    const queryParams = new URLSearchParams({ job_id: jobId });
    if (options?.level) queryParams.append('level', options.level);
    if (options?.startTime) queryParams.append('start_time', options.startTime.toISOString());
    if (options?.endTime) queryParams.append('end_time', options.endTime.toISOString());
    if (options?.limit) queryParams.append('limit', options.limit.toString());

    const response = await fetch(`${API_BASE_URL}/api/v1/logs?${queryParams.toString()}`);
    if (!response.ok) {
        const errorData = await response.json().catch(() => ({ error: 'Failed to parse error response' }));
        throw new ApiError(response.status, errorData?.error || 'Failed to fetch logs');
    }
    const data = await response.json();
    return data.logs || []; // Ensure logs array exists
  }
  // Add cancelJob etc.
};
```

### 4. Job Listing Component

*(... Job Listing Component example remains largely the same, focusing on displaying data from `listJobs` ...)*

### 5. Job Details Component

*(... Job Details Component example remains largely the same, focusing on displaying data from `getJobStatus` and potentially `getLogs` ...)*

### 6. Job Request Form Component

```typescript
// JobCreator.tsx
import React, { useState } from 'react';
import { Button, TextField, CircularProgress, Alert } from '@mui/material'; // Example imports
import { useJobContext } from './JobContext'; // Adjust import path
import { MultiConfigJobRequest, ApiError } from './types'; // Adjust import path
// import { useNavigate } from 'react-router-dom'; // If using router

export const JobCreator: React.FC = () => {
  const { createJob } = useJobContext();
  // State to hold different config fragments
  const [workorderConfig, setWorkorderConfig] = useState({ project: { path: '' }, intent: { description: '' } });
  const [teamConfig, setTeamConfig] = useState({ llm_config: { /* ... */ } }); // Example
  const [runtimeConfig, setRuntimeConfig] = useState({ logging: { level: 'INFO' } }); // Example

  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  // const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);

    // Validation
    if (!workorderConfig.project?.path) {
      setError('Project path is required');
      return;
    }
    if (!workorderConfig.intent?.description) {
      setError('Intent description is required');
      return;
    }

    try {
      setSubmitting(true);
      // Construct the list of config fragments for the multi-config API
      const configFragments = [
        { runtime: runtimeConfig }, // Base runtime config
        { team: teamConfig },       // Base team config
        { workorder: workorderConfig } // Job-specific workorder
        // Add more fragments or overrides as needed
      ];

      const job = await createJob(configFragments); // Call context function
      // navigate(`/jobs/${job.job_id}`); // Navigate on success
      console.log("Job Created:", job); // Placeholder navigation
    } catch (err) {
      setError(err instanceof ApiError ? err.message : 'Failed to create job');
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <Alert severity="error">{error}</Alert>}

      <TextField
        label="Project Path"
        value={workorderConfig.project.path}
        onChange={(e) => setWorkorderConfig(prev => ({ ...prev, project: { ...prev.project, path: e.target.value } }))}
        required
        fullWidth margin="normal"
      />
       <TextField
        label="Intent Description"
        value={workorderConfig.intent.description}
        onChange={(e) => setWorkorderConfig(prev => ({ ...prev, intent: { ...prev.intent, description: e.target.value } }))}
        required
        fullWidth margin="normal" multiline rows={4}
      />

      {/* Add more form fields for other configurations if needed */}

      <Button type="submit" variant="contained" disabled={submitting}>
        {submitting ? <CircularProgress size={24} /> : 'Create Job'}
      </Button>
    </form>
  );
};
```

### 7. Log Viewer Component

```typescript
// LogViewer.tsx
export const LogViewer: React.FC<{
  logs: LogEntry[];
  onRefresh: () => void;
  onFilterChange: (filters: LogFilterOptions) => void;
}> = ({ logs, onRefresh, onFilterChange }) => {
  const [filters, setFilters] = useState<LogFilterOptions>({
    level: 'INFO',
    component: undefined,
    search: ''
  });
  
  const handleFilterChange = (key: keyof LogFilterOptions, value: any) => {
    const newFilters = { ...filters, [key]: value };
    setFilters(newFilters);
    onFilterChange(newFilters);
  };
  
  const filteredLogs = useMemo(() => {
    return logs.filter(log => {
      if (filters.level && log.level !== filters.level) return false;
      if (filters.component && log.component !== filters.component) return false;
      if (filters.search && !log.message.toLowerCase().includes(filters.search.toLowerCase())) return false;
      return true;
    });
  }, [logs, filters]);
  
  return (
    <div className="log-viewer">
      <div className="log-controls">
        <FormControl size="small">
          <InputLabel>Level</InputLabel>
          <Select
            value={filters.level || ''}
            onChange={(e) => handleFilterChange('level', e.target.value)}
          >
            <MenuItem value="">All</MenuItem>
            {['DEBUG', 'INFO', 'WARNING', 'ERROR'].map(level => 
              <MenuItem key={level} value={level}>{level}</MenuItem>
            )}
          </Select>
        </FormControl>
        
        <TextField
          label="Search"
          size="small"
          value={filters.search}
          onChange={(e) => handleFilterChange('search', e.target.value)}
        />
        
        <Button onClick={onRefresh} startIcon={<RefreshIcon />}>
          Refresh
        </Button>
      </div>
      
      <div className="log-entries">
        {filteredLogs.length === 0 ? (
          <EmptyState message="No logs matching current filters" />
        ) : (
          <table className="log-table">
            <thead>
              <tr>
                <th>Time</th>
                <th>Level</th>
                <th>Component</th>
                <th>Message</th>
              </tr>
            </thead>
            <tbody>
              {filteredLogs.map((log, index) => (
                <tr key={index} className={`log-level-${log.level.toLowerCase()}`}>
                  <td>{formatTime(log.timestamp)}</td>
                  <td>
                    <LogLevelBadge level={log.level} />
                  </td>
                  <td>{log.component || '-'}</td>
                  <td>{log.message}</td>
                </tr>
              ))}
            </tbody>
          </table>
        )}
      </div>
    </div>
  );
};

const LogLevelBadge: React.FC<{level: string}> = ({ level }) => {
  const colorMap: Record<string, string> = {
    'DEBUG': 'gray',
    'INFO': 'blue',
    'WARNING': 'orange',
    'ERROR': 'red'
  };
  
  return (
    <span className={`log-badge log-level-${level.toLowerCase()}`} style={{ backgroundColor: colorMap[level] || 'gray' }}>
      {level}
    </span>
  );
};
```

### 8. Lineage Viewer Component

```typescript
// LineageViewer.tsx
export const LineageViewer: React.FC<{lineageId: string}> = ({ lineageId }) => {
  const [lineageData, setLineageData] = useState<LineageData | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchLineageData = async () => {
      try {
        setLoading(true);
        const data = await apiService.getLineageData(lineageId);
        setLineageData(data);
        setError(null);
      } catch (err) {
        setError(err instanceof ApiError ? err.message : 'Failed to load lineage data');
      } finally {
        setLoading(false);
      }
    };
    
    fetchLineageData();
  }, [lineageId]);
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorDisplay message={error} onRetry={() => setError(null)} />;
  if (!lineageData) return <EmptyState message="No lineage data available" />;
  
  return (
    <div className="lineage-viewer">
      <h3>Execution Lineage</h3>
      
      <div className="lineage-timeline">
        {lineageData.events.map((event, index) => (
          <div key={index} className="lineage-event">
            <div className="event-time">{formatTime(event.timestamp)}</div>
            <div className="event-agent">{event.agent}</div>
            <div className="event-action">{event.action}</div>
            <div className="event-details">
              <pre>{JSON.stringify(event.details, null, 2)}</pre>
            </div>
          </div>
        ))}
      </div>
      
      <h4>Decision Points</h4>
      <div className="decision-points">
        {lineageData.decisions.map((decision, index) => (
          <div key={index} className="decision-point">
            <div className="decision-agent">{decision.agent}</div>
            <div className="decision-context">{decision.context}</div>
            <div className="decision-outcome">{decision.outcome}</div>
            <div className="decision-reasoning">
              <pre>{decision.reasoning}</pre>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### 7. Error Handling Utilities

```typescript
// errors.ts
export class ApiError extends Error {
  status: number;
  
  constructor(status: number, message: string) {
    super(message);
    this.status = status;
    this.name = 'ApiError';
  }
}

// ErrorBoundary.tsx
export class ErrorBoundary extends React.Component<{fallback: React.ReactNode}> {
  state = { hasError: false, error: null };
  
  static getDerivedStateFromError(error: any) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: any, errorInfo: any) {
    console.error
    console.error("Uncaught error:", error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    
    return this.props.children;
  }
}
```

### 8. Implementation Notes

1. **Type Safety**
   - All components use TypeScript interfaces that match the backend API
   - Strict prop validation with React PropTypes as a secondary check
   - Comprehensive logging type definitions
   - Runtime type checking for API responses

2. **State Management**
   - Job context maintains global job state
   - Component state for UI controls and loading states
   - Local storage for persistent job history


3. **Error Handling**
   - Consistent error handling patterns across components
   - Error boundaries for component-level failures
   - Detailed error messages with user-friendly suggestions
   - Clear visual indicators for error states

4. **Performance Considerations**
   - Optimized rendering with React.memo and useMemo
   - Efficient polling with dynamic intervals
   - Pagination for job history
   - Cached responses for frequently accessed data
   - Optimized log filtering and rendering

5. **Observability Features**
   - Structured logging with consistent format
   - Lineage tracking for workflow traceability
   - Correlation IDs across components
   - Filterable log viewer with search capabilities