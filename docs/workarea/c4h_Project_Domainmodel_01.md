# C4H Project Domain Model
Describing the representational meta model over the Teams, Projects, WorkOrders, Jobs

## Use case

1. Project Picker 
    a. Create a new Project
    b. Open an existing Project
    c. Delete a Project (projects setting the state to deleted and not visible no actual deletion happens)

2. Picked Project Context
    a. LLM Chat window, 

```mermaid
classDiagram

   class Team {
      capabilityDescription: String
      intent: Prompt
      workflow: Workflow
      roles: Role[*]

      subTeams: Team[*]

      llmConfig: Config "System LLM Config"
      runtimeConfig: Config "System Run Time Config"
   }

   class Workflow {
      workflow: Graph<Node>[*]
      nodes: Node[*]
   }

   class Project {
      workOrders: WorkOrder[*]
      assets: Asset[*]
      jobs: Job[*]
    }

   class Asset {
      <<Asset Location>>
      asset: Asset_URI
    }

   class WorkOrder {
      <<Requirments>>
      intent: Prompt
      asset: Asset
      goal: Prompt
      validationCritera: Prompt
      subOrder: WorkOrder[*]
   }

   class Prompt {
      <<Prompt Action>>
      description: string
   }

   class Role {
      <<BaseAgent Persona>>
      intent: Intent
      llmConfig: LLMConfig
   }

   class Skill {
      <<BaseAgent, Action>>
   }

   class Job {
      jobID: GUID
      order: WorkOrder
      teamReference: Team
      teamConfig: Team "config overrides"

      workflow_state: Workflow_State

   }

   class LineageEvent {
      <<State Snapshot>>
   }

    %% Relationships
   Project "1" -- "0..*" WorkOrder
   Project "1" -- "0..*" Job
   Project "1" -- "0..*" Asset

   Team -- Workflow
   Team "0..*" -- "0..*" Role 
   Role "0..*" -- "0..*" Skill
   WorkOrder -- Asset

   Job -- WorkOrder

```