# C4H Editor Comprehensive Implementation Guide

## Overview

The C4H Editor is a client frontend application for the C4H (Code for Humans) service, an intelligent code refactoring system that analyzes and modifies codebases according to specified intents. This document combines detailed design specifications with clear MVP requirements to guide implementation.

## System Architecture

The C4H Editor follows a client-server architecture:

1. **C4H Service**: The core service that performs code refactoring using LLM-based agent teams
2. **C4H Editor Backend**: A server application that communicates with the C4H Service and manages state
3. **C4H Editor Frontend**: A web application for creating and managing work orders

```
┌────────────────┐      ┌────────────────┐      ┌────────────────┐
│                │      │                │      │                │
│  C4H Editor   │<─────>│  C4H Editor   │<─────>│  C4H Service  │
│   Frontend    │      │    Backend     │      │                │
│                │      │                │      │                │
└────────────────┘      └────────────────┘      └────────────────┘
```

## Core Principles

The C4H Editor adheres to the following design principles:

### Agent Design Principles

1. **LLM-First Processing**: Offload logic and decision-making to the LLM
2. **Minimal Agent Logic**: Keep agent code focused on infrastructure concerns
3. **Single Responsibility**: Each agent has one focused task
4. **Clear Boundaries**: Each component has well-defined responsibilities
5. **Logging Over Validation**: Focus on detailed logging for debugging
6. **Forward-Only Flow**: Data flows forward through the agent chain

### Configuration Design Principles

1. **Hierarchical Configuration**: All configuration follows a strict hierarchy
2. **Smart Merge Behavior**: Base config provides foundation, overrides update leaf values
3. **Separation of Responsibilities**: Each component owns its configuration section

## C4H Service Integration

The C4H Editor integrates with the C4H Service API, which provides the following endpoints:

- `POST /api/v1/workflow` - Submit a new workflow request
- `GET /api/v1/workflow/{workflow_id}` - Check status of an existing workflow
- `GET /health` - Service health check

### Workflow Process

When a work order is submitted, it follows this process:

1. **Discovery Phase**: The C4H Service analyzes project structure and files
2. **Solution Design Phase**: Plans modifications based on intent
3. **Coding Phase**: Implements the designed changes
4. **Verification Phase**: Validates changes (if configured)

### Integration Flow

1. The C4H Editor Backend sends a request to the C4H Service with the project path and intent
2. The backend polls the C4H Service for status updates
3. The frontend displays the status and results to the user

## MVP Requirements

### 1. WorkOrder Management

#### 1.1 Create WorkOrders
- Allow users to create new work orders according to the WorkOrder design
- The editor should align with the major sections of the top-level root nodes of the work order configuration
- Support free-form text editing while maintaining the required configuration structure
- Provide YAML-aware editing as a bonus feature

#### 1.2 List WorkOrders
- Display a grid view of all available work orders
- Include key metadata: ID, description, status, created date, last modified date
- Support filtering and sorting
- Enable quick actions from the grid view

#### 1.3 Edit WorkOrders
- Allow editing of existing work orders
- Maintain version history
- Preserve configuration hierarchy during edits

#### 1.4 Copy WorkOrders
- Enable duplication of existing work orders as templates for new ones
- Automatically assign new unique IDs to copied work orders

#### 1.5 Archive WorkOrders
- Allow users to archive work orders without deleting them
- Provide a view for archived work orders

### 2. Job Management

#### 2.1 Submit WorkOrders
- Enable submission of work orders to the C4H service
- Validate work order configuration before submission

#### 2.2 Track Jobs
- Monitor job status from submission until completion
- Display progress indicators and status updates
- Show detailed information for each job

#### 2.3 Job Results
- Display results of completed jobs
- Provide access to logs and output artifacts

## Data Models

### WorkOrder Model

```typescript
interface WorkOrder {
  id: string;
  template: WorkOrderTemplate;
  metadata: WorkOrderMetadata;
  parent_id?: string;
  lineage?: string[];
}

interface WorkOrderTemplate {
  text: string;
  parameters: WorkOrderParameter[];
  config: WorkOrderConfig;
}

interface WorkOrderMetadata {
  author: string;
  archived: boolean;
  created_at: string;
  updated_at: string;
  description?: string;
  tags?: string[];
  version: string;
  goal?: string;
  priority?: string;
  due_date?: string;
  assignee?: string;
  asset?: string;
}
```

### Job Model

```typescript
interface Job {
  id: string;
  workOrderId: string;
  workOrderVersion: string;
  status: JobStatus;
  createdAt: string;
  updatedAt: string;
  submittedAt?: string;
  completedAt?: string;
  userId?: string;
  configuration: Record<string, any>;
  results?: JobResult;
}

enum JobStatus {
  DRAFT = 'draft',
  SUBMITTED = 'submitted',
  RUNNING = 'running',
  COMPLETED = 'completed',
  FAILED = 'failed',
  CANCELED = 'canceled'
}
```

## Backend Architecture

### API Routes

The backend provides these key endpoints:

1. **WorkOrder Routes**:
   - `GET /api/v1/workorders` - List all WorkOrders
   - `POST /api/v1/workorders` - Create a new WorkOrder
   - `GET /api/v1/workorders/{id}` - Get a specific WorkOrder
   - `PUT /api/v1/workorders/{id}` - Update a WorkOrder
   - `DELETE /api/v1/workorders/{id}` - Delete a WorkOrder
   - `GET /api/v1/workorders/{id}/history` - Get version history
   - `POST /api/v1/workorders/{id}/archive` - Archive a WorkOrder
   - `POST /api/v1/workorders/{id}/unarchive` - Unarchive a WorkOrder
   - `POST /api/v1/workorders/{id}/clone` - Clone a WorkOrder

2. **Job Routes**:
   - `GET /api/v1/jobs` - List all jobs
   - `POST /api/v1/jobs` - Create a new job
   - `GET /api/v1/jobs/{id}` - Get a specific job
   - `POST /api/v1/jobs/{id}/cancel` - Cancel a job
   - `GET /api/v1/jobs/{id}/logs` - Get job logs

### Storage

The C4H Editor Backend uses two primary storage mechanisms:

1. **Git Repository**: Stores WorkOrders as files with version history
2. **Database**: Stores metadata, job records, and user information

## Frontend Architecture

### React Components

The frontend is built with React and TypeScript, with a component hierarchy organized by feature:

1. **WorkOrderEditor**: The main editor component
   - IntentConfigTab
   - ProjectConfigTab
   - LLMConfigTab
   - OrchestrationConfigTab
   - RuntimeConfigTab
   - YAMLEditorTab
   - WorkOrderParameterPanel
   - WorkOrderMetadataPanel
   - WorkOrderVersionControl

2. **WorkOrderList**: For browsing and managing WorkOrders
   - WorkOrderListItem
   - WorkOrderFilters
   - WorkOrderSortControls
   - WorkOrderPagination

3. **JobsList**: For viewing and managing jobs
   - JobListItem
   - JobFilters
   - JobStatusIndicator

4. **JobDetails**: For viewing job details and results
   - JobStatus
   - JobLogs
   - JobResults

### State Management

State is managed through React hooks and context:

1. **useWorkOrderApi**: Custom hook for WorkOrder CRUD operations
2. **useJobApi**: Custom hook for Job submission and status tracking
3. **AppContext**: Global state for user settings and session info

## Implementation Phases

The C4H Editor will be implemented in phases:

### Phase 1: WorkOrder Editor & Management

1. **WorkOrder Editor Enhancements**:
   - Tabbed interface for configuration sections
   - Each editor is for a YAML configuration
  - For MVP only two tabs
    1.1 System Configuratin - YAML 
    1.2 Intent Configuration -YMAL
  
  - Workorder is the full system + intent config
  - Refer to the SystemConfig Design Guide, WorkOrder Design Guide, Config Design Principles

2. **WorkOrder Management Improvements**:
   - Archive/unarchive functionality
   - Clone functionality
   - Filtering, sorting, and pagination
   - Version comparison and rollback UI

### Phase 2: Job Management & Integration

1. **Job Management Enhancements**:
   - Improved job status tracking and polling
   - Detailed job results visualization
   - Job logs viewer
   - Job failure handling and retry options

2. **C4H Service Integration**:
   - Robust error handling
   - Better status monitoring
   - Progress indicators

### Phase 3: User Experience & Advanced Features

1. **User Experience Improvements**:
   - Better error handling and feedback
   - Responsive design
   - Performance optimizations

2. **Advanced Features**:
   - Team collaboration
   - Templates for common tasks
   - Advanced search

## Technical Dependencies

### Backend

- **Framework**: FastAPI for API endpoints
- **Version Control**: Git for tracking WorkOrder changes
- **Data Validation**: Pydantic for model validation
- **Storage**: File-based storage with Git and optional database
- **Authentication**: Basic authentication for MVP
- **Logging**: Structured logging with correlation IDs

### Frontend

- **Framework**: React with TypeScript
- **UI Components**: Material-UI for consistent design
- **State Management**: React hooks with context for global state
- **Code Editor**: Monaco Editor (same as VS Code) with YAML support
- **Routing**: React Router for navigation
- **API Client**: Axios for HTTP requests

## Data Flow

1. User creates/edits a work order in the frontend
2. Frontend sends work order data to backend
3. Backend validates and stores the work order with version control
4. User submits work order for processing
5. Backend sends work order to C4H service
6. Backend polls C4H service for job status
7. Frontend displays job status and results to user

## Version Management

- Work orders are stored as single files in a Git repository
- Each edit creates a new commit with the changes
- Backend provides access to version history
- Frontend allows viewing and restoring previous versions

## Deployment Considerations

1. **Environment Variables**:
   - C4H Service API URL and credentials
   - Storage paths and database connections
   - Authentication settings

2. **Security**:
   - Authentication for users
   - Validation of project paths
   - Protection against path traversal attacks

3. **Scalability**:
   - Stateless API design
   - Efficient Git operations
   - Proper caching

## Future Enhancements (Post-MVP)

1. **Team Collaboration**: Multi-user editing and permissions
2. **Advanced Search**: Full-text search across work orders
3. **Templates**: Pre-configured work order templates for common tasks
4. **Visualizations**: Visual representation of job execution flow
5. **Notifications**: Email or webhook notifications for job completion