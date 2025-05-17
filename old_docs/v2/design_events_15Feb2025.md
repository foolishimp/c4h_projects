# C4H Architecture Diagrams

## Core Class Model

This diagram shows the main class relationships and inheritance hierarchy:

```mermaid
classDiagram
    %% Base Classes and Core Infrastructure
    class BaseConfig {
        -config: Dict
        -project: Optional[Project]
        -metrics: Dict
        -log_level: LogDetail
        +_get_provider_config()
        +_get_agent_config()
        +_resolve_model()
        +ensure_paths()
        +resolve_path()
    }

    class BaseLLM {
        +_get_completion_with_continuation()
        +_process_response()
        +_get_model_str()
        +_setup_litellm()
    }

    class BaseAgent {
        -provider: LLMProvider
        -model: str
        -temperature: float
        +process()
        +_process()
        +_get_data()
        +_format_request()
        +_get_llm_content()
    }

    %% Core Agents
    class DiscoveryAgent {
        -workspace_root: Path
        -tartxt_config: Dict
        +_run_tartxt()
        +_parse_manifest()
        +_resolve_input_paths()
    }

    class SolutionDesigner {
        +_format_request()
        +_extract_context_data()
        +_validate_input()
        +_process_llm_response()
    }

    class Coder {
        -iterator: SemanticIterator
        -merger: SemanticMerge
        -asset_manager: AssetManager
        -operation_metrics: CoderMetrics
        +process()
    }

    class AssuranceAgent {
        -workspace_root: Path
        +_run_pytest()
        +_run_script()
    }

    %% Semantic Skills
    class SemanticIterator {
        -_state: ExtractorState
        -_fast_extractor: FastExtractor
        -_slow_extractor: SlowExtractor
        +configure()
        +__iter__()
        +__next__()
    }

    class SemanticMerge {
        -preserve_formatting: bool
        -allow_partial: bool
        +process()
        +_get_original_content()
    }

    class SemanticExtract {
        +extract()
        +_format_request()
    }

    class AssetManager {
        -project_path: Path
        -backup_dir: Path
        -backup_enabled: bool
        -merger: SemanticMerge
        +process_action()
    }

    %% Relationships
    BaseConfig <|-- BaseLLM
    BaseLLM <|-- BaseAgent
    BaseAgent <|-- DiscoveryAgent
    BaseAgent <|-- SolutionDesigner
    BaseAgent <|-- Coder
    BaseAgent <|-- AssuranceAgent
    BaseAgent <|-- SemanticIterator
    BaseAgent <|-- SemanticMerge
    BaseAgent <|-- SemanticExtract
    BaseAgent <|-- AssetManager
```

## Job and Event Model

This diagram shows the event-driven workflow model:

```mermaid
classDiagram
    %% Core Job/Event Classes
    class Job {
        +job_id: str
        +intent: Dict
        +project_path: Path
        +workspace_path: Path
        +config: Dict
        +timestamp: datetime
        +status: JobStatus
        +events: List[Event]
        +result: Optional[Dict]
        +error: Optional[str]
    }

    class Event {
        +event_id: str
        +job_id: str
        +agent_type: str
        +event_type: EventType
        +timestamp: datetime
        +input_data: Dict
        +output_data: Dict
        +error: Optional[str]
        +metadata: Dict
    }

    class EventType {
        <<enumeration>>
        DISCOVERY_STARTED
        DISCOVERY_COMPLETED
        SOLUTION_STARTED
        SOLUTION_COMPLETED
        CODER_STARTED
        CODER_COMPLETED
        ASSURANCE_STARTED
        ASSURANCE_COMPLETED
        ERROR
    }

    class JobStatus {
        <<enumeration>>
        PENDING
        IN_PROGRESS
        COMPLETED
        FAILED
    }

    %% Workflow Components
    class WorkflowOrchestrator {
        +job_store: JobStore
        +event_store: EventStore
        +submit_job(intent: Dict)
        +process_event(event: Event)
        +get_job_status(job_id: str)
    }

    class JobStore {
        +save_job(job: Job)
        +get_job(job_id: str)
        +update_job_status(job_id: str, status: JobStatus)
        +list_jobs(filters: Dict)
    }

    class EventStore {
        +save_event(event: Event)
        +get_events(job_id: str)
        +get_latest_event(job_id: str)
    }

    class AgentExecutor {
        +agent_registry: Dict
        +execute_agent(event: Event)
        +create_next_event(prev_event: Event)
    }

    %% Relationships
    Job "1" *-- "*" Event : contains
    Job --> JobStatus : has
    Event --> EventType : has
    WorkflowOrchestrator --> JobStore : uses
    WorkflowOrchestrator --> EventStore : uses
    WorkflowOrchestrator --> AgentExecutor : delegates
    AgentExecutor --> "1..*" BaseAgent : executes
```

## Job Processing Sequence

This diagram shows how jobs flow through the system:

```mermaid
sequenceDiagram
    participant Client
    participant WO as WorkflowOrchestrator
    participant JS as JobStore
    participant ES as EventStore
    participant AE as AgentExecutor
    participant Agent

    Client->>WO: submit_job(intent)
    WO->>JS: save_job(new_job)
    WO->>ES: save_event(DISCOVERY_STARTED)
    WO->>AE: execute_agent(event)
    AE->>Agent: process(event.input_data)
    Agent-->>AE: result
    AE->>ES: save_event(DISCOVERY_COMPLETED)
    AE->>WO: notify_completion(event)
    WO->>JS: update_job_status()
    WO->>AE: execute_agent(next_event)
    note right of WO: Continue through agent chain
```

## Key Design Elements

1. Core Infrastructure
- BaseConfig provides configuration management
- BaseLLM handles LLM interactions
- BaseAgent combines both with standard processing

2. Agent Hierarchy
- Core agents (Discovery, Solution, Coder, Assurance)
- Semantic skills (Iterator, Merge, Extract)
- Asset management for file operations

3. Job Processing
- Jobs represent complete units of work
- Events track state transitions
- Storage components maintain history
- Orchestration manages workflow

4. Agent Design Principles Alignment
- Clear component boundaries
- Single responsibility per class
- Observable behavior through events
- Forward-only event flow
- Stateless agent operations


# OpenLineage Integration with Prefect Workflow

## OpenLineage Event Model

```mermaid
classDiagram
    %% OpenLineage Core Classes
    class OpenLineageEvent {
        +eventType: EventType
        +eventTime: datetime
        +run: RunFacet
        +job: JobFacet
        +inputs: List[InputDataset]
        +outputs: List[OutputDataset]
        +producer: str
        +schemaURL: str
    }

    class RunFacet {
        +runId: str
        +facets: Dict
        +nominalTime: datetime
        +parent: Optional[RunLink]
    }

    class JobFacet {
        +namespace: str
        +name: str
        +facets: Dict
    }

    class CustomFacets {
        +agentType: str
        +agentConfig: Dict
        +metrics: Dict
        +projectContext: Dict
    }

    class Dataset {
        +namespace: str
        +name: str
        +facets: Dict
    }

    %% Prefect Integration
    class PrefectTask {
        +task_name: str
        +flow_id: str
        +ol_context: OpenLineageContext
        +run_task()
        +emit_lineage_event()
    }

    class OpenLineageContext {
        +run_id: str
        +parent_run_id: Optional[str]
        +job_namespace: str
        +job_name: str
        +emit_start()
        +emit_complete()
        +emit_fail()
    }

    %% Agent Integration
    class AgentEventEmitter {
        +ol_client: OpenLineageClient
        +emit_start(agent: BaseAgent, input: Dict)
        +emit_complete(agent: BaseAgent, output: Dict)
        +emit_fail(agent: BaseAgent, error: str)
        -create_agent_facets()
        -create_datasets()
    }

    %% Relationships
    OpenLineageEvent --> RunFacet
    OpenLineageEvent --> JobFacet
    OpenLineageEvent --> Dataset
    RunFacet --> CustomFacets
    JobFacet --> CustomFacets
    PrefectTask --> OpenLineageContext
    OpenLineageContext --> OpenLineageEvent
    AgentEventEmitter --> OpenLineageEvent
```

## Event Flow Sequence

```mermaid
sequenceDiagram
    participant PF as Prefect Flow
    participant PT as Prefect Task
    participant OL as OpenLineage
    participant Agent as BaseAgent
    participant DS as Dataset Store

    PF->>PT: start_task()
    PT->>OL: emit_start_run()
    PT->>Agent: process()
    
    alt Success Path
        Agent->>DS: read_input()
        DS-->>Agent: input_data
        Agent->>Agent: process_data()
        Agent->>DS: write_output()
        Agent-->>PT: success_result
        PT->>OL: emit_complete_run(outputs)
    else Error Path
        Agent->>Agent: error_occurs()
        Agent-->>PT: error_result
        PT->>OL: emit_failed_run(error)
    end

    PT-->>PF: task_result
```

## OpenLineage Custom Facets

These custom facets capture agent-specific metadata:

```json
{
    "agent": {
        "type": "agent_facet_v1",
        "agentType": "discovery",
        "provider": "anthropic",
        "model": "claude-3-opus-20240229",
        "metrics": {
            "total_requests": 1,
            "successful_requests": 1,
            "total_duration": 2.5
        }
    },
    "project": {
        "type": "project_facet_v1",
        "path": "/path/to/project",
        "workspace": "/path/to/workspace",
        "config": {}
    },
    "execution": {
        "type": "execution_facet_v1",
        "startTime": "2024-02-15T10:00:00Z",
        "endTime": "2024-02-15T10:01:00Z",
        "status": "COMPLETED"
    }
}
```

## Key Integration Points

1. Prefect Task Wrapper
```python
@task
def run_agent_task(agent_config: AgentTaskConfig, context: Dict[str, Any]):
    with OpenLineageContext() as ol:
        try:
            ol.emit_start(
                inputs=[create_input_dataset(context)],
                facets=create_agent_facets(agent_config)
            )
            
            result = agent_config.agent_class(config=agent_config.config).process(context)
            
            ol.emit_complete(
                outputs=[create_output_dataset(result)],
                metrics=result.metrics
            )
            return result
            
        except Exception as e:
            ol.emit_fail(error=str(e))
            raise
```

2. Dataset Management
```python
def create_input_dataset(context: Dict[str, Any]) -> Dataset:
    return Dataset(
        namespace="c4h",
        name=f"input_{context['job_id']}",
        facets={
            "schema": create_schema_facet(context),
            "dataSource": create_source_facet(context)
        }
    )
```

3. Agent Facets
```python
def create_agent_facets(agent: BaseAgent) -> Dict[str, Any]:
    return {
        "agent": {
            "type": "agent_facet_v1",
            "agentType": agent._get_agent_name(),
            "provider": str(agent.provider),
            "model": agent.model,
            "metrics": agent.metrics
        }
    }
```

## Benefits of OpenLineage Integration

1. Data Lineage Tracking
- Complete view of data flow through agents
- Input/output relationships
- Processing context and configuration

2. Observability
- Rich metadata via custom facets
- Agent performance metrics
- Error tracking and context

3. Debugging and Monitoring
- Full execution history
- Detailed failure context
- Agent performance patterns

4. Compliance and Auditing
- Complete processing history
- Configuration tracking
- Model usage tracking

## Gemini Attempted Model
```mermaid
graph TD
    subgraph Agents
        DiscoveryAgent["Discovery Agent"] --> LocateKeys
        DiscoveryAgent --> LocateConfig
        SolutionDesignerAgent["Solution Designer Agent"] --> BaseConfig
        SolutionDesignerAgent --> BaseLLM
        CoderAgent["Coder Agent"] --> BaseAgent
        CoderAgent --> SemanticMerge
        CoderAgent --> SemanticIterator
        AssuranceAgent["Assurance Agent"] --> BaseAgent
    end
    subgraph Skills
        LocateKeys["Locate Keys"]
        LocateConfig["Locate Config"]
        BaseConfig["Base Config"]
        BaseLLM["Base LLM"]
        BaseAgent["Base Agent"]
        SemanticMerge["Semantic Merge"]
        SemanticIterator["Semantic Iterator"]
    end
    subgraph Teams
        Team1["Refactoring Team"] --> DiscoveryAgent
        Team1 --> SolutionDesignerAgent
        Team1 --> CoderAgent
        Team1 --> AssuranceAgent
    end
    subgraph Other
        Project["Project"]
        ProjectPaths["Project Paths"]
        AgentResponse["Agent Response"]
        LogDetail["Log Detail"]
        LLMProvider["LLM Provider"]
    end
    BaseAgent --> AgentResponse
    BaseAgent --> LogDetail
    BaseAgent --> LLMProvider
    Project --> ProjectPaths
    LocateKeys --> Project
    LocateConfig --> Project
```


### Claude model
```mermaid
flowchart TB
    subgraph CLI["Prefect Runner CLI"]
        PR[prefect_runner.py]
        PR --> RunMode{Run Mode}
        RunMode -->|agent| AgentExec[Execute Single Agent]
        RunMode -->|workflow| WFExec[Execute Full Workflow]
    end

    subgraph Config["Configuration System"]
        SC[system_config.yml]
        AC[app_config.yml]
        SC --> CM[Config Merger]
        AC --> CM
        CM --> FC[Final Config]
        FC -->|Project Paths| PP[Project Configuration]
        FC -->|Agent Config| AConfig[Agent Configuration]
        FC -->|Workflow Config| WConfig[Workflow Configuration]
    end

    subgraph Workflow["Prefect Workflow"]
        WF[workflows.py]
        subgraph Flow["Basic Refactoring Flow"]
            Init[Initialize Flow] --> Discovery
            Discovery[Discovery Agent] --> Solution[Solution Designer]
            Solution --> Coder[Coder Agent]
            Coder --> Complete[Complete Flow]
        end
        
        subgraph Storage["Event Storage"]
            ES[Store Events] --> WS[Workflow State]
            ES --> AS[Agent States]
        end
        
        WF --> Flow
        Flow --> Storage
    end

    PR -->|Load| Config
    Config -->|Configure| Workflow
    
    subgraph Agents["Agent Tasks"]
        AT[Agent Task Config]
        RT[run_agent_task]
        AT --> RT
        RT -->|Execute| Agent[BaseAgent Process]
    end

    WF -->|Create| AT
    Flow -->|Run| RT

    subgraph BackupSystem["Backup System"]
        BS[Backup Service]
        BS -->|Store| Backup[(Backup Storage)]
    end

    Coder -->|Before Changes| BS
```


This updated version retains all of your original content and vernacular while enhancing clarity, structure, and consistency.


<svg xmlns="http://www.w3.org/2000/svg" width="800" height="600">
  <!-- BaseAgent class -->
  <rect x="50" y="50" width="200" height="100" fill="white" stroke="black"/>
  <text x="150" y="70" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">BaseAgent</text>
  <line x1="50" y1="80" x2="250" y2="80" stroke="black"/>
  <text x="55" y="95" font-family="Arial" font-size="10">+process(context: Dict)</text>
  <text x="55" y="110" font-family="Arial" font-size="10">#_get_agent_name()</text>
  <text x="55" y="125" font-family="Arial" font-size="10">#_format_request(context: Dict)</text>
  <text x="55" y="140" font-family="Arial" font-size="10">#_process_response(content: str)</text>

  <!-- Project class -->
  <rect x="300" y="50" width="200" height="100" fill="white" stroke="black"/>
  <text x="400" y="70" text-anchor="middle" font-family="Arial" font-size="12" font-weight="bold">Project</text>
  <line x1="300" y1="80" x2="500" y2="80" stroke="black"/>
  <text x="305" y="95" font-family="Arial" font-size="10">+paths: ProjectPaths</text>
  <text x="305" y="110" font-family="Arial" font-size="10">+metadata: ProjectMetadata</text>
  <text x="305" y="125" font-family="Arial" font-size="10">+config: Dict</text>
  <text x="305" y="140" font-family="Arial" font-size="10">+resolve_path(path: Path)</text>
  <text x="305" y="155" font-family="Arial" font-size="10">+get_agent_config(agent_name: str)</text>

  <!-- Subclasses -->
  <rect x="50" y="200" width="200" height="20" fill="white" stroke="black"/>
  <text x="150" y="215" text-anchor="middle" font-family="Arial" font-size="10">DiscoveryAgent</text>

  <rect x="50" y="230" width="200" height="20" fill="white" stroke="black"/>
  <text x="150" y="245" text-anchor="middle" font-family="Arial" font-size="10">SolutionDesigner</text>

  <rect x="50" y="260" width="200" height="20" fill="white" stroke="black"/>
  <text x="150" y="275" text-anchor="middle" font-family="Arial" font-size="10">Coder</text>

  <rect x="50" y="290" width="200" height="20" fill="white" stroke="black"/>
  <text x="150" y="305" text-anchor="middle" font-family="Arial" font-size="10">AssuranceAgent</text>

  <!-- Relationships -->
  <line x1="150" y1="150" x2="150" y2="200" stroke="black" marker-end="url(#arrowhead)"/>
  <line x1="150" y1="150" x2="150" y2="230" stroke="black" marker-end="url(#arrowhead)"/>
  <line x1="150" y1="150" x2="150" y2="260" stroke="black" marker-end="url(#arrowhead)"/>
  <line x1="150" y1="150" x2="150" y2="290" stroke="black" marker-end="url(#arrowhead)"/>

  <line x1="250" y1="100" x2="300" y2="100" stroke="black" marker-end="url(#diamond)"/>
  
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" fill="black"/>
    </marker>
    <marker id="diamond" markerWidth="10" markerHeight="10" refX="5" refY="5" orient="auto">
      <polygon points="5 0, 10 5, 5 10, 0 5" fill="black"/>
    </marker>
  </defs>
</svg>