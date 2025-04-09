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
- `GET /api/v1/logs` - Retrieve system logs with filtering options

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
- **Observability Team** - Tracks lineage and logs across the workflow
  - Monitors execution flow
  - Captures decision points
  - Provides traceability for debugging
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
      "entry_team": "discovery",
      "lineage": {
        "enabled": true,
        "tracking_level": "detailed",
        "correlation_id": "optional-custom-id",
        "persist": true
      }
    }
  },
  "runtime": {
    "logging": {
      "level": "INFO"
    },
    "backup": {
      "enabled": true
    },
    "observability": {
      "logging": {
        "level": "INFO",
        "format": "json"
      }
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
  "error": null,
  "lineage_id": "lin_20250402_123456_abcd1234",
  "log_path": "workspaces/logs/job_20250402_123456_abcd1234.log"
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
  "lineage_id": "lin_20250402_123456_abcd1234",
  "log_path": "workspaces/logs/job_20250402_123456_abcd1234.log",
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

### Retrieve Logs

```
GET /api/v1/logs?job_id={job_id}&level=INFO&start_time=2025-04-02T12:00:00Z
```

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| job_id | string | Filter logs by job ID |
| level | string | Filter by log level (DEBUG, INFO, WARNING, ERROR) |
| start_time | ISO datetime | Filter logs after this time |
| end_time | ISO datetime | Filter logs before this time |
| limit | integer | Maximum number of log entries to return |

#### Response

```json
{
  "logs": [{"timestamp": "2025-04-02T12:34:56Z", "level": "INFO", "message": "Job started", "job_id": "job_20250402_123456_abcd1234", "lineage_id": "lin_20250402_123456_abcd1234", "component": "discovery_agent"}]
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
  "enable_logging": true
}
```

#### Parameters for Workflow Continuation

| Field | Type | Description |
|-------|------|-------------|
| lineage_file | string | Path to lineage file for workflow continuation |
| stage | string | Stage to continue workflow from ("discovery", "solution_designer", "coder") |
| keep_runid | boolean | Whether to keep the original run ID from the lineage file (default: true) |
| enable_logging | boolean | Whether to enable detailed logging (default: true) |

#### Response

```json
{
  "workflow_id": "wf_1234_abcd1234",
  "status": "pending",
  "storage_path": "workspaces/lineage/wf_1234_abcd1234",
  "error": null,
  "log_path": "workspaces/logs/wf_1234_abcd1234.log"
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
  "version": "1.5.2",
  "workflows_tracked": 5,
  "log_level": "INFO",
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
      observability:
        name: "Observability Team"
        tasks:
          - name: "lineage_tracking"
            agent_class: "c4h_agents.agents.observability.LineageTrackingAgent"
          - name: "log_aggregation"
            agent_class: "c4h_agents.agents.observability.LogAggregationAgent"
        routing: {}
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
      type: "structured"
      path: "workspaces/lineage" 
      format: "json"
      retention: "30d"
  logging:
    level: "INFO"
    format: "json"
    handlers:
      - type: "file"
        path: "workspaces/logs"
        rotation: "10MB"
  backup:
    enabled: true
    path: "workspaces/backups"
```

## Frontend Job API Implementation

The frontend implementation for the Jobs API follows these design patterns:

### 1. Job Context Management

```typescript
// JobContext.tsx
interface JobContextType {
  currentJob: Job | null;
  jobHistory: Job[];
  createJob: (request: JobRequest) => Promise<Job>;
  getJobStatus: (jobId: string) => Promise<Job>;
  pollJobStatus: (jobId: string) => void;
  stopPolling: (jobId: string) => void;
  getLogs: (jobId: string, options?: LogOptions) => Promise<LogEntry[]>;
}

const JobContext = createContext<JobContextType>(null!);

export const JobProvider: React.FC<{children: React.ReactNode}> = ({ children }) => {
  const [currentJob, setCurrentJob] = useState<Job | null>(null);
  const [jobHistory, setJobHistory] = useState<Job[]>([]);
  const pollingRefs = useRef<{[key: string]: number}>({});
  const [jobLogs, setJobLogs] = useState<{[key: string]: LogEntry[]}>({});
  
  // Create a new job with proper validation
  const createJob = async (request: JobRequest): Promise<Job> => {
    try {
      const newJob = await apiService.createJob(request);
      setCurrentJob(newJob);
      setJobHistory(prev => [newJob, ...prev]);
      startLogPolling(newJob.job_id);
      return newJob;
    } catch (error) {
      console.error("Failed to create job:", error);
      throw error;
    }
  };

  // Other job management methods...
};
```

### 2. Type Definitions

```typescript
// job.ts
export interface JobRequest {
  workorder: {
    project: {
      path: string;
      workspace_root?: string;
    };
    intent: {
      description: string;
      target_files?: string[];
    };
  };
  team?: TeamConfig;
  runtime?: RuntimeConfig;
  logging_config?: LoggingConfig;
}

export interface Job {
  job_id: string;
  status: JobStatus;
  storage_path: string;
  error: string | null;
  changes?: JobChange[];
  lineage_id?: string;
  log_path?: string;
  created_at?: string;
  updated_at?: string;
}

export interface LogEntry {
  timestamp: string;
  level: 'DEBUG' | 'INFO' | 'WARNING' | 'ERROR';
  message: string;
  job_id: string;
  lineage_id?: string;
  component?: string;
  context?: Record<string, any>;
  trace_id?: string;
}

export type JobStatus = 'pending' | 'running' | 'success' | 'error' | 'complete' | 'failed';
```

### 3. API Service Implementation

```typescript
// apiService.ts
export const createJob = async (jobRequest: JobRequest): Promise<Job> => {
  const response = await fetch(`${API_BASE_URL}/api/v1/jobs`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(jobRequest)
  });
  
  if (!response.ok) {
    const errorData = await response.json().catch(() => null);
    throw new ApiError(response.status, errorData?.error || 'Failed to create job');
  }
  
  return await response.json();
};

export const getLogs = async (jobId: string, options?: LogOptions): Promise<LogEntry[]> => {
  const queryParams = new URLSearchParams();
  queryParams.append('job_id', jobId);
  if (options?.level) queryParams.append('level', options.level);
  if (options?.startTime) queryParams.append('start_time', options.startTime.toISOString());
  if (options?.endTime) queryParams.append('end_time', options.endTime.toISOString());
  if (options?.limit) queryParams.append('limit', options.limit.toString());
  
  const response = await fetch(`${API_BASE_URL}/api/v1/logs?${queryParams.toString()}`);
  if (!response.ok) throw new ApiError(response.status, 'Failed to fetch logs');
  return (await response.json()).logs;
};
```

### 4. Job Listing Component

```typescript
// JobsList.tsx
export const JobsList: React.FC = () => {
  const { jobHistory } = useJobContext();
  const [filter, setFilter] = useState<JobStatus | 'all'>('all');
  
  const filteredJobs = useMemo(() => {
    if (filter === 'all') return jobHistory;
    return jobHistory.filter(job => job.status === filter);
  }, [jobHistory, filter]);
  
  return (
    <div className="jobs-list">
      <div className="filter-controls">
        {/* Filter controls */}
      </div>
      
      <table className="jobs-table">
        <thead>
          <tr>
            <th>Job ID</th>
            <th>Status</th>
            <th>Created</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {filteredJobs.map(job => (
            <tr key={job.job_id} className={`status-${job.status}`}>
              <td>{job.job_id}</td>
              <td>
                <StatusBadge status={job.status} />
              </td>
              <td>{formatDate(job.created_at)}</td>
              <td>
                <button onClick={() => viewJobDetails(job.job_id)}>Details</button>
                {job.status === 'error' && 
                <button onClick={() => viewJobLogs(job.job_id)}>Logs</button>
                  <button onClick={() => retryJob(job.job_id)}>Retry</button>}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};
```

### 5. Job Details Component

```typescript
// JobDetails.tsx
export const JobDetails: React.FC<{jobId: string}> = ({ jobId }) => {
  const { getJobStatus } = useJobContext();
  const [job, set null>(null);
  const [loading, setLoading] = useState(true);
  const [logs, setLogs] = useState<LogEntry[]>([]);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchJobDetails = async () => {
      try {
        setLoading(true);
        const jobData = await getJobStatus(jobId);
        setJob(jobData);
        if (jobData.log_path) fetchJobLogs(jobData.job_id);
        setError(null);
      } catch (err) {
        setError(err instanceof ApiError ? err.message : 'Failed to load job details');
      } finally {
        setLoading(false);
      }
    };
    
    fetchJobDetails();
    
    // Start polling if job is in active state
    if (job && ['pending', 'running'].includes(job.status)) {
      const pollingInterval = setInterval(fetchJobDetails, 5000);
      return () => clearInterval(pollingInterval);
    }
  }, [jobId, job?.status]);
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorDisplay message={error} onRetry={() => setError(null)} />;
  if (!job) return <EmptyState message="Job not found" />;
  
  return (
    <div className="job-details">
      <h2>Job {job.job_id}</h2>
      <StatusBadge status={job.status} />
      
      {job.lineage_id && (
        <div className="lineage-info">
          <h3>Lineage ID: {job.lineage_id}</h3>
        </div>
      )}
      
      <h3>Job Configuration</h3>
      <pre>{JSON.stringify(job, null, 2)}</pre>
      
      <h3>Changes</h3>
      {job.changes?.map(change => (
        <FileChangeCard key={change.file} change={change} />
      ))}
      
      <h3>Logs</h3>
      <LogViewer 
        logs={logs} 
        onRefresh={() => fetchJobLogs(job.job_id)}
        onFilterChange={handleLogFilterChange}
      />
    </div>
  );
};
```

### 6. Job Request Form Component

```typescript
// JobCreator.tsx
export const JobCreator: React.FC = () => {
  const { createJob } = useJobContext();
  const [formData, setFormData] = useState<Partial<JobRequest>>({
    workorder: {
      project: { path: '' },
      intent: { description: '' }
    },
    runtime: {
      observability: {
        logging: {
          level: 'INFO',
          format: 'json'
        }
    }
  });
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const navigate = useNavigate();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    
    // Validation
    if (!formData.workorder?.project?.path) {
      setError('Project path is required');
      return;
    }
    
    if (!formData.workorder?.intent?.description) {
      setError('Intent description is required');
      return;
    }
    
    try {
      setSubmitting(true);
      const job = await createJob(formData as JobRequest);
      navigate(`/jobs/${job.job_id}`);
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
        value={formData.workorder?.project?.path || ''}
        onChange={(e) => setFormData(prev => ({ ...prev, workorder: { ...prev.workorder, project: { ...prev.workorder?.project, path: e.target.value } } }))}
        required
      />

      <FormControl>
        <InputLabel>Logging Level</InputLabel>
        <Select
          value={formData.runtime?.observability?.logging?.level || 'INFO'}
          onChange={(e) => setFormData(prev => ({ ...prev, runtime: { ...prev.runtime, observability: { ...prev.runtime?.observability, logging: { ...prev.runtime?.observability?.logging, level: e.target.value } } } }))}
        >
          {['DEBUG', 'INFO', 'WARNING', 'ERROR'].map(level => <MenuItem key={level} value={level}>{level}</MenuItem>)}
        </Select>
      </FormControl>
      
      {/* Other form fields */}
      
      <Button type="submit" disabled={submitting}>
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
}
]