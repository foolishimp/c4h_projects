I'll provide exhaustive Mermaid UML diagrams for the data domain models of WorkOrders and Jobs in the C4H Editor system. These diagrams will show all the attributes and relationships based on the codebase.

## WorkOrder Domain Model

Here's a comprehensive class diagram for the WorkOrder domain model:

```mermaid
classDiagram
    class WorkOrder {
        +string id
        +WorkOrderTemplate template
        +WorkOrderMetadata metadata
        +string parent_id
        +string[] lineage
    }

    class WorkOrderTemplate {
        +string text
        +WorkOrderParameter[] parameters
        +WorkOrderConfig config
    }

    class WorkOrderParameter {
        +string name
        +ParameterType type
        +string description
        +any default
        +boolean required
    }

    class WorkOrderMetadata {
        +string author
        +boolean archived
        +string created_at
        +string updated_at
        +string description
        +string[] tags
        +string version
        +string goal
        +string priority
        +string|null due_date
        +string assignee
        +string asset
        +string target_model
        +string|null intent
    }

    class WorkOrderConfig {
        +number temperature
        +number max_tokens
        +number|null top_p
        +number|null frequency_penalty
        +number|null presence_penalty
        +string[] stop_sequences
        +string|null service_id
        +string|null workflow_id
        +number|null max_runtime
        +boolean notify_on_completion
        +Record~string,any~ parameters
        +Record~string,any~ providers
        +Record~string,any~ backup
        +Record~string,any~ logging
        +Record~string,any~ project
    }

    class ParameterType {
        <<enumeration>>
        STRING
        NUMBER
        BOOLEAN
        ARRAY
        OBJECT
    }

    class WorkOrderVersionInfo {
        +string version
        +string commit_hash
        +string created_at
        +string author
        +string message
    }

    class WorkOrderHistoryResponse {
        +string workorder_id
        +WorkOrderVersionInfo[] versions
    }

    WorkOrder *-- WorkOrderTemplate : contains
    WorkOrder *-- WorkOrderMetadata : contains
    WorkOrderTemplate *-- "0..*" WorkOrderParameter : contains
    WorkOrderTemplate *-- WorkOrderConfig : contains
    WorkOrderParameter -- ParameterType : uses
```

## Job Domain Model

Here's a comprehensive class diagram for the Job domain model:

```mermaid
classDiagram
    class Job {
        +string id
        +string workOrderId
        +string workOrderVersion
        +JobStatus status
        +string serviceJobId
        +string createdAt
        +string updatedAt
        +string submittedAt
        +string completedAt
        +string userId
        +Record~string,any~ configuration
        +JobResult results
    }

    class JobStatus {
        <<enumeration>>
        CREATED
        SUBMITTED
        RUNNING
        COMPLETED
        FAILED
        CANCELLED
    }

    class JobResult {
        +string output
        +any[] artifacts
        +Record~string,any~ metrics
        +string error
    }

    class JobSubmitRequest {
        +string workOrderId
        +string userId
        +Record~string,any~ configuration
    }

    Job -- JobStatus : has status
    Job *-- JobResult : contains
```

## Entity Relationships

Here's a diagram showing the relationships between WorkOrders and Jobs:

```mermaid
classDiagram
    class WorkOrder {
        +string id
        +WorkOrderTemplate template
        +WorkOrderMetadata metadata
        +string parent_id
        +string[] lineage
    }

    class Job {
        +string id
        +string workOrderId
        +string workOrderVersion
        +JobStatus status
        +string serviceJobId
        +string createdAt
        +string updatedAt
        +string submittedAt
        +string completedAt
        +string userId
        +Record~string,any~ configuration
        +JobResult results
    }

    class User {
        +string id
        +string name
        +string email
    }

    WorkOrder <-- Job : is created from
    Job -- User : submitted by
```

## API and State Interfaces

Here are the key interfaces related to WorkOrder and Job operations:

```mermaid
classDiagram
    class WorkOrderApiHook {
        +WorkOrder[] workOrders
        +WorkOrder workOrder
        +boolean loading
        +Error error
        +fetchWorkOrders()
        +fetchWorkOrder(id)
        +createWorkOrder(workOrder)
        +updateWorkOrder(workOrder)
        +deleteWorkOrder(id)
        +archiveWorkOrder(id)
        +unarchiveWorkOrder(id)
        +cloneWorkOrder(id, newId)
        +getWorkOrderHistory(id)
        +testWorkOrder(id, parameters)
        +renderWorkOrder(id, parameters)
    }

    class JobApiHook {
        +Job[] jobs
        +Job job
        +boolean loading
        +Error error
        +fetchJobs()
        +fetchJob(id)
        +submitJob(data)
        +cancelJob(id)
        +pollJobStatus(id)
        +getJobLogs(jobId)
    }

    class WorkOrderContextState {
        +WorkOrder workOrder
        +WorkOrder originalWorkOrder
        +boolean loading
        +string error
        +boolean saved
        +string yaml
        +boolean hasUnsavedChanges
        +loadWorkOrder(id)
        +createNewWorkOrder()
        +updateWorkOrderId(id)
        +updateYaml(yaml)
        +saveWorkOrder()
        +submitWorkOrder()
        +resetSavedState()
    }

    WorkOrderApiHook --> WorkOrder : manages
    JobApiHook --> Job : manages
    WorkOrderContextState --> WorkOrder : manages state
```

## API Endpoints and Data Flow


```mermaid
classDiagram
    class API_ENDPOINTS {
        +WORKORDERS: string
        +WORKORDER(id): function
        +WORKORDER_HISTORY(id): function
        +WORKORDER_CLONE(id): function
        +WORKORDER_ARCHIVE(id): function
        +WORKORDER_UNARCHIVE(id): function
        +WORKORDER_RENDER(id): function
        +WORKORDER_TEST(id): function
        +JOBS: string
        +JOB(id): function
        +JOB_CANCEL(id): function
    }

    class BackendRequests {
        +GET workorders
        +POST workorders
        +GET workorderById
        +PUT workorderUpdate
        +DELETE workorderDelete
        +POST workorderClone
        +POST workorderArchive
        +POST workorderUnarchive
        +GET workorderHistory
        +POST createJob
        +GET listJobs
        +GET jobById
        +POST cancelJob
    }

    BackendRequests --> API_ENDPOINTS : uses
```

For more clarity, here's a table representation of the API endpoints:

| API Endpoint | HTTP Method | Path |
|--------------|-------------|------|
| List WorkOrders | GET | `/api/v1/workorders` |
| Create WorkOrder | POST | `/api/v1/workorders` |
| Get WorkOrder | GET | `/api/v1/workorders/{id}` |
| Update WorkOrder | PUT | `/api/v1/workorders/{id}` |
| Delete WorkOrder | DELETE | `/api/v1/workorders/{id}` |
| Clone WorkOrder | POST | `/api/v1/workorders/{id}/clone` |
| Archive WorkOrder | POST | `/api/v1/workorders/{id}/archive` |
| Unarchive WorkOrder | POST | `/api/v1/workorders/{id}/unarchive` |
| Get WorkOrder History | GET | `/api/v1/workorders/{id}/history` |
| Test WorkOrder | POST | `/api/v1/workorders/{id}/test` |
| Render WorkOrder | POST | `/api/v1/workorders/{id}/render` |
| List Jobs | GET | `/api/v1/jobs` |
| Create Job | POST | `/api/v1/jobs` |
| Get Job | GET | `/api/v1/jobs/{id}` |
| Cancel Job | POST | `/api/v1/jobs/{id}/cancel` |

These endpoints form the API interface that the frontend uses to communicate with the backend for managing WorkOrders and Jobs in the C4H Editor system.

## Config Design Hierarchical Structure

Based on the design principles, here's how the configuration hierarchy should be structured:

```mermaid
classDiagram
    class WorkOrderConfig {
        +project
        +intent
        +llm_config
        +orchestration
        +runtime
        +backup
        +logging
    }

    class ProjectConfig {
        +string path
        +string workspace_root
        +string source_root
        +string output_root
        +string config_root
    }

    class IntentConfig {
        +string description
        +string[] target_files
    }

    class LLMConfig {
        +Record~string,ProviderConfig~ providers
        +string default_provider
        +string default_model
        +Record~string,AgentConfig~ agents
    }

    class ProviderConfig {
        +string api_base
        +number context_length
        +string env_var
        +string default_model
        +string[] valid_models
        +ExtendedThinkingConfig extended_thinking
        +Record~string,any~ litellm_params
    }

    class AgentConfig {
        +string provider
        +string model
        +number temperature
        +Record~string,string~ prompts
    }

    class OrchestrationConfig {
        +boolean enabled
        +string entry_team
        +Record~string,TeamConfig~ teams
        +ErrorHandlingConfig error_handling
    }

    class RuntimeConfig {
        +WorkflowConfig workflow
        +LineageConfig lineage
    }

    class BackupConfig {
        +boolean enabled
        +string path
    }

    class LoggingConfig {
        +string level
        +string format
        +string agent_level
        +Record~string,any~ providers
        +Record~string,any~ truncate
    }

    WorkOrderConfig *-- ProjectConfig : contains
    WorkOrderConfig *-- IntentConfig : contains
    WorkOrderConfig *-- LLMConfig : contains
    WorkOrderConfig *-- OrchestrationConfig : contains
    WorkOrderConfig *-- RuntimeConfig : contains
    WorkOrderConfig *-- BackupConfig : contains
    WorkOrderConfig *-- LoggingConfig : contains
    LLMConfig *-- ProviderConfig : contains
    LLMConfig *-- AgentConfig : contains
```


These diagrams provide a comprehensive view of the data models in the C4H Editor system. Let me know if you'd like me to focus on any particular aspect in more detail.


```mermaid
classDiagram
    class WorkOrderConfig {
        +workOderID   
        +project
        +intent
        +llm_config
        +orchestration
        +runtime
        +backup
        +logging
    }

    class SystemConfig {
        +systemConfigID
    }

    class ProjectConfig {
        +string path
        +string workspace_root
        +string source_root
        +string output_root
        +string config_root
    }

    class IntentConfig {
        +string description
        +string[] target_files
    }

    class LLMConfig {
        +Record~string,ProviderConfig~ providers
        +string default_provider
        +string default_model
        +Record~string,AgentConfig~ agents
    }

    class ProviderConfig {
        +string api_base
        +number context_length
        +string env_var
        +string default_model
        +string[] valid_models
        +ExtendedThinkingConfig extended_thinking
        +Record~string,any~ litellm_params
    }

    class AgentConfig {
        +string provider
        +string model
        +number temperature
        +Record~string,string~ prompts
    }

    class OrchestrationConfig {
        +boolean enabled
        +string entry_team
        +Record~string,TeamConfig~ teams
        +ErrorHandlingConfig error_handling
    }

    class RuntimeConfig {
        +WorkflowConfig workflow
        +LineageConfig lineage
    }

    class BackupConfig {
        +boolean enabled
        +string path
    }

    class LoggingConfig {
        +string level
        +string format
        +string agent_level
        +Record~string,any~ providers
        +Record~string,any~ truncate
    }

    WorkOrderConfig *-- ProjectConfig : contains
    WorkOrderConfig *-- IntentConfig : contains
    SystemConfig *-- LLMConfig : contains
    SystemConfig *-- OrchestrationConfig : contains
    WorkOrderConfig *-- RuntimeConfig : contains
    WorkOrderConfig *-- BackupConfig : contains
    WorkOrderConfig *-- LoggingConfig : contains
    LLMConfig *-- ProviderConfig : contains
    LLMConfig *-- AgentConfig : contains
```

I'll create detailed UML sequence and state diagrams to illustrate the complete workflow from listing work orders to job status monitoring in the C4H Editor system.

## Sequence Diagram: Complete WorkOrder and Job Workflow

```mermaid
sequenceDiagram
    actor User
    participant UI as Shell App UI
    participant List as WorkOrderList
    participant Editor as WorkOrderEditor
    participant Context as WorkOrderContext
    participant API as Backend API
    participant YAML as YAML Editor
    participant JobList as JobsList
    participant JobDetails as JobDetails
    participant C4H as C4H Service

    %% List Work Orders
    User->>UI: Navigate to Work Orders
    UI->>List: Load Work Order List
    List->>API: GET /api/v1/workorders
    API-->>List: Return WorkOrder[] Array
    List-->>UI: Display Work Order List

    %% Edit a Work Order
    User->>List: Click Edit on a WorkOrder
    List->>UI: Navigate to /workorders/{id}
    UI->>Editor: Load WorkOrderEditor with id
    Editor->>Context: loadWorkOrder(id)
    Context->>API: GET /api/v1/workorders/{id}
    API-->>Context: Return WorkOrder object
    Context->>YAML: Generate YAML representation
    YAML-->>Editor: Display YAML for editing
    User->>YAML: Modify WorkOrder configuration
    YAML->>Context: updateYaml(newYaml)
    Context->>Context: Set yamlDirty flag

    %% Save a Work Order
    User->>Editor: Click Save button
    Editor->>Context: saveWorkOrder()
    Context->>Context: Parse YAML to WorkOrder object
    Context->>Context: Validate WorkOrder structure
    Context->>API: PUT /api/v1/workorders/{id}
    API-->>Context: Return updated WorkOrder
    Context->>Context: Reset yamlDirty flag
    Context->>UI: Publish workorder:saved event
    UI-->>User: Show success notification

    %% Submit as Job
    User->>Editor: Click "Submit Work Order"
    Editor->>Context: submitWorkOrder()
    Context->>Context: saveWorkOrder() if needed
    Note over Context,API: Work Order is saved first
    Context->>API: POST /api/v1/jobs (workOrderId)
    API->>C4H: Submit Job to C4H Service
    C4H-->>API: Return Job ID and initial status
    API-->>Context: Return Job object
    Context->>UI: Navigate to /jobs
    UI->>JobList: Load JobsList

    %% Monitor Job Status
    User->>JobList: View Jobs list
    JobList->>API: GET /api/v1/jobs
    API-->>JobList: Return Job[] array
    JobList-->>User: Display Jobs with status
    User->>JobList: Click on Job for details
    JobList->>JobDetails: Show Job Details (jobId)
    JobDetails->>API: GET /api/v1/jobs/{id}
    API->>C4H: Check job status in C4H Service
    C4H-->>API: Return updated Job status
    API-->>JobDetails: Return Job details
    JobDetails-->>User: Display detailed Job status

    %% Poll for updates
    loop While job is running
        JobDetails->>API: Poll GET /api/v1/jobs/{id}
        API->>C4H: Get updated status
        C4H-->>API: Return current status
        API-->>JobDetails: Update Job details
        JobDetails-->>User: Update status display
    end
```

## State Diagram: WorkOrder Lifecycle

```mermaid
stateDiagram-v2
    [*] --> WorkOrderList: Navigate to app
    
    WorkOrderList --> NewWorkOrder: Create New
    WorkOrderList --> EditWorkOrder: Edit Existing
    WorkOrderList --> CloneWorkOrder: Clone Existing
    
    NewWorkOrder --> EditingYAML: Initialize empty WorkOrder
    CloneWorkOrder --> EditingYAML: Load cloned WorkOrder
    EditWorkOrder --> EditingYAML: Load existing WorkOrder
    
    EditingYAML --> EditingYAML: Modify YAML
    EditingYAML --> ValidationFailed: Save with invalid YAML
    ValidationFailed --> EditingYAML: Return to editing
    
    EditingYAML --> UnsavedChanges: YAML modified
    UnsavedChanges --> SaveAttempt: Click Save
    
    SaveAttempt --> SavedWorkOrder: Validation passes
    SaveAttempt --> ValidationFailed: Validation fails
    
    SavedWorkOrder --> UnsavedChanges: Make changes
    SavedWorkOrder --> SubmitJob: Submit as Job
    SavedWorkOrder --> WorkOrderList: Close editor
    
    SubmitJob --> JobCreated: Job submission succeeds
    SubmitJob --> SubmitFailure: Job submission fails
    SubmitFailure --> SavedWorkOrder: Return to editor
    
    JobCreated --> JobsList: Navigate to Jobs view
    
    JobsList --> JobDetails: Select job
    
    JobDetails --> JobRunning: Status = RUNNING
    JobDetails --> JobSubmitted: Status = SUBMITTED
    JobDetails --> JobCompleted: Status = COMPLETED
    JobDetails --> JobFailed: Status = FAILED
    
    JobRunning --> JobRunning: Poll for updates
    JobRunning --> JobCompleted: Job finishes
    JobRunning --> JobFailed: Job fails
    JobRunning --> JobCancelled: Cancel job
    
    JobSubmitted --> JobSubmitted: Poll for updates
    JobSubmitted --> JobRunning: Job starts running
    JobSubmitted --> JobFailed: Job fails
    JobSubmitted --> JobCancelled: Cancel job
    
    JobsList --> WorkOrderList: Navigate back
    JobDetails --> JobsList: Close details
```

## State Diagram: Job Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Created: Job Created
    
    Created --> Submitted: Job Sent to C4H Service
    Submitted --> Running: C4H Service Processing
    
    Running --> Completed: Success
    Running --> Failed: Error
    Running --> Cancelled: User Cancels
    
    Submitted --> Failed: Submission Error
    Submitted --> Cancelled: User Cancels
    
    state Running {
        [*] --> Discovery: Discovery Phase
        Discovery --> SolutionDesign: Solution Design Phase
        SolutionDesign --> Coding: Coding Phase
        Coding --> Verification: Verification Phase
        Verification --> [*]
        
        Discovery --> DiscoveryFailed: Error
        SolutionDesign --> SolutionFailed: Error
        Coding --> CodingFailed: Error
        Verification --> VerificationFailed: Error
        
        DiscoveryFailed --> [*]
        SolutionFailed --> [*]
        CodingFailed --> [*]
        VerificationFailed --> [*]
    }
    
    Completed --> [*]
    Failed --> [*]
    Cancelled --> [*]
```

## Data Flow State Diagram: WorkOrder Editing

```mermaid
stateDiagram-v2
    [*] --> InitialLoad: Load WorkOrder
    
    InitialLoad --> WorkOrderLoaded: Success
    InitialLoad --> LoadError: API Error
    
    WorkOrderLoaded --> YAMLGenerated: Generate YAML
    YAMLGenerated --> Viewing: Display in Editor
    
    Viewing --> Editing: User modifies YAML
    Editing --> UnsavedChanges: YAML content changes
    UnsavedChanges --> ValidationCheck: Save requested
    
    ValidationCheck --> SyntaxError: Invalid YAML
    SyntaxError --> UnsavedChanges: Return to editing
    
    ValidationCheck --> SchemaValid: YAML parses to valid WorkOrder
    ValidationCheck --> SchemaInvalid: YAML parses but invalid schema
    SchemaInvalid --> UnsavedChanges: Return to editing
    
    SchemaValid --> SaveRequest: Send to API
    SaveRequest --> SaveSuccess: API Success
    SaveRequest --> SaveError: API Error
    
    SaveSuccess --> Saved: Update UI State
    SaveError --> UnsavedChanges: Show error
    
    Saved --> SubmitJobRequest: Submit as job
    SubmitJobRequest --> JobCreated: Success
    SubmitJobRequest --> JobError: Error

    JobCreated --> [*]: Navigate to Jobs view
    SaveSuccess --> [*]: Close editor (optional)
    LoadError --> [*]: Return to list
```

These diagrams provide a comprehensive view of the workflow and states involved in the C4H Editor system from listing work orders to monitoring job execution. The sequence diagram shows the interactions between components, while the state diagrams illustrate the various states and transitions that occur during the WorkOrder editing and Job execution processes.

The diagrams capture the complete lifecycle including:
1. Loading the list of work orders
2. Editing a work order in the YAML editor
3. Saving changes back to the server
4. Submitting a work order as a job
5. Monitoring job status through various phases



## Data Flow State Diagram: Coniguration Life cycle
The life cycle of a configuration from the editor through to the C4H services

The C4h service configuration can be broken down into three major sections

1. Work order - This carries the main Intent of a work order describing what it wants done, it may include details on how
2. System Config
3. Run Time Config

Finally there is a single container called the Job, that captures all these details

Sunnyday path through the service.

```mermaid
stateDiagram-v2
    state C4H_Editor {
        [*] --> Build_WorkOrder : Intent
        Build_WorkOrder --> ConfigureJob : Job = Tuple(WorkOrder, SystemConfig, RuntimeConfig)
        ConfigureJob --> Created: Job Tuple
        Created --> Submitted: Job(JobID) Sent to C4H Service
    }
    
    Submitted --> C4H_Service: C4H Service Processing
    
    state C4H_Service {
        [*] --> InitialiseJob: Job(JobID, WorkOrder, SystemConfig, RuntimeConfig)
        InitialiseJob --> MergeConfig: Merge C4h System Config with Job
        MergeConfig --> ExecuteWorkflow: Master Config
        ExecuteWorkflow --> [*]: Results
    }
    
    C4H_Service --> Completed: Success
    Completed --> [*]
```

This diagram illustrates the flow from the editor side to the service side and the important configuration merging process. It shows how:

In the C4H Editor:

The process starts with an intent
A WorkOrder is built based on that intent
A job is configured as a tuple of (WorkOrder, SystemConfig, RuntimeConfig)
The job is created and then submitted to the C4H Service


In the C4H Service:

The job is initialized with its ID, WorkOrder, SystemConfig, and RuntimeConfig
The C4H system config is merged with the job config to create a master config
The workflow is executed based on this master config
Results are produced


Upon successful completion, the job is marked as completed

This accurately represents the configuration hierarchy and merging behavior described in the design principles document, where the system configuration provides the foundation, and the job-specific configurations provide overrides that are merged according to specific rules.


Based on the Configuration Lifecycle diagram and description, I'll create a comprehensive target domain model for the C4H editor that properly represents the distinct components and their relationships.

Based on your requirements, I'll focus solely on the client-side domain model for the C4H Editor, which is responsible for creating the three configurations that make up a Job. Let me create a clean, focused domain model for just the client side.

# C4H Editor Client-Side Domain Model

```mermaid
classDiagram
    %% Main entities
    class WorkOrder {
        +string id
        +string description
        +string[] target_files
        +WorkOrderMetadata metadata
        
        +create()
        +load(id)
        +save()
        +validate()
    }

    class TeamConfig {
        +ProjectConfig project
        +LLMConfig llm_config
        +OrchestrationConfig orchestration
        +BackupConfig backup
        +LoggingConfig logging
        
        +create()
        +load()
        +save()
        +validate()
    }

    class RuntimeConfig {
        +WorkflowConfig workflow
        +LineageConfig lineage
        
        +create()
        +load()
        +save()
        +validate()
    }

    class Job {
        +string id
        +string workOrderId
        +JobStatus status
        +DateTime createdAt
        +DateTime updatedAt
        +DateTime submittedAt
        
        +submit()
        +getStatus()
        +cancel()
    }

    %% WorkOrder related classes
    class WorkOrderMetadata {
        +string author
        +boolean archived
        +DateTime created_at
        +DateTime updated_at
        +string[] tags
        +string version
        +string goal
        +string priority
        +DateTime due_date
        +string assignee
        +string asset
    }

    %% SystemConfig related classes
    class ProjectConfig {
        +string path
        +string workspace_root
        +string source_root
        +string output_root
        +string config_root
    }

    class LLMConfig {
        +ProviderConfig[] providers
        +string default_provider
        +string default_model
        +AgentConfig[] agents
    }

    class OrchestrationConfig {
        +boolean enabled
        +string entry_team
        +TeamConfig[] teams
        +ErrorHandlingConfig error_handling
    }

    class BackupConfig {
        +boolean enabled
        +string path
    }

    class LoggingConfig {
        +string level
        +string format
        +string agent_level
        +ProviderLoggingConfig[] providers
        +TruncateConfig truncate
    }

    %% RuntimeConfig related classes
    class WorkflowConfig {
        +boolean enabled
        +string root_dir
        +string format
        +string[] subdirs
        +RetentionConfig retention
        +ErrorHandlingConfig error_handling
    }

    class LineageConfig {
        +boolean enabled
        +string namespace
        +boolean separate_input_output
        +BackendConfig backend
        +ContextConfig context
        +RetryConfig retry
    }

    %% Job related classes
    class JobStatus {
        <<enumeration>>
        CREATED
        SUBMITTED
        RUNNING
        COMPLETED
        FAILED
        CANCELLED
    }

    class JobConfiguration {
        +WorkOrder workOrder
        +SystemConfig systemConfig
        +RuntimeConfig runtimeConfig
        
        +combine()
        +validate()
    }

    %% Relationships
    WorkOrder *-- WorkOrderMetadata
    WorkOrder *-- ProjectConfig

    TeamConfig *-- LLMConfig
    TeamConfig *-- OrchestrationConfig    
    TeamConfig *-- WorkflowConfig

    RuntimeConfig *-- LineageConfig
    RuntimeConfig *-- BackupConfig
    RuntimeConfig *-- LoggingConfig

    Job *-- JobStatus
    Job *-- JobConfiguration
    
    JobConfiguration *-- WorkOrder
    JobConfiguration *-- TeamConfig
    JobConfiguration *-- RuntimeConfig
```

## Client-Side Domain Model Explanation

### Core Components

1. **WorkOrder**
   - Contains the intent description and details about what needs to be done
   - Has metadata about the work (author, version, priority, etc.)
   - Focused on capturing the "what" of the refactoring task

2. **TeamConfig was System Config**
   - Controls the infrastructure and capabilities of the agent system
   - Includes project settings, LLM configuration, orchestration settings, backup, and logging
   - Defines "how" the system operates at a strategic level

3. **RuntimeConfig**
   - Manages execution behavior and artifact storage
   - Includes workflow and lineage configuration
   - Controls "how" the system operates at a tactical level

4. **Job**
   - Serves as the container for submitting work to the C4H Service
   - Bundles WorkOrder, SystemConfig, and RuntimeConfig together
   - Tracks status of submitted work

5. **JobConfiguration**
   - Helper class that combines the three config components
   - Validates that the combined configuration is valid
   - Prepares the payload for job submission

### Responsibilities of the C4H Editor

The C4H Editor is responsible for:

1. **Creating and Editing Configurations**
   - Providing UI for editing WorkOrder (intent, target files)
   - Offering structured editors for SystemConfig settings
   - Managing RuntimeConfig options

2. **Configuration Management**
   - Loading/saving configurations
   - Version tracking
   - Validating each configuration type

3. **Job Management**
   - Submitting jobs to the C4H Service
   - Monitoring job status
   - Displaying results

4. **Configuration Serialization**
   - Converting between object model and YAML representation
   - Handling imports/exports of configurations

This domain model focuses purely on the client-side responsibilities, excluding the C4H Service implementation details. The editor's primary job is to create, manage, and combine the three configuration types (WorkOrder, SystemConfig, RuntimeConfig) into a complete Job that can be submitted to the C4H Service.