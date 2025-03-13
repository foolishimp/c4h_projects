# C4H Project Domain Model

```mermaid
classDiagram

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

   WorkOrder -- Asset
   WorkOrder -- Prompt
   Job -- WorkOrder

```


```mermaid
journey
    title My Day
    section Morning
        Wake up: 5: Me
        Coffee: 3: Me
    section Evening
        Sleep: 4: Me
```

```mermaid
gantt
    title Project Timeline
    dateFormat  YYYY-MM-DD
    section Phase 1
        Task 1 :a1, 2025-03-01, 10d
        Task 2 :after a1, 5d
```
```mermaid
requirementDiagram
    requirement R1 {
        id: R1
        text: System shall boot
    }
    element E1 {
        type: Component
    }
    E1 - satisfies -> R1
```

```mermaid
mindmap
    Root
        A
            A1:i have an idea
            A2
        B
            B1
```

```mermaid
graph TD
    Person[User] --> System[Web App]
    System --> Container[API]
    Container --> Database[DB]
```

```mermaid
gitGraph
    commit
    branch develop
    commit
    checkout main
    merge develop
```

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--o{ ITEM : contains
    CUSTOMER {
        string name
        int id PK
    }
```

```mermaid
graph TD
    A[Start] --> B{Decision}
    B -->|Yes| C[End]
    B -->|No| A
```

```mermaid
graph TB
    subgraph "Application"
        subgraph Presentation_Layer["Presentation Layer"]
            UI[User Interface]
            API_Client[API Client]
            UI --> API_Client
        end
        subgraph Business_Layer["Business Layer"]
            Services[Business Services]
            Logic[Core Logic]
            Services --> Logic
        end
        subgraph Data_Layer["Data Layer"]
            DB[Database]
            Cache[Cache]
            DB --> Cache
        end

        Presentation_Layer --> Business_Layer
        Business_Layer --> Data_Layer
    end

    %% Styling to emphasize layers
    classDef layer fill:#e6f3ff,stroke:#333,stroke-width:2px
    classDef app fill:#f0f0f0,stroke:#666,stroke-width:4px
    class Presentation_Layer,Business_Layer,Data_Layer layer
    class Application app
```

```mermaid
graph TB
    subgraph "MyApp"
        subgraph UI_Layer["UI Layer"]
            Web[Web App]
            Mobile[Mobile App]
        end
        subgraph Logic_Layer["Logic Layer"]
            API[API Gateway]
            Core[Business Core]
            API --> Core
        end
        subgraph Storage_Layer["Storage Layer"]
            SQL[SQL DB]
            NoSQL[NoSQL DB]
        end

        UI_Layer --> Logic_Layer
        Logic_Layer --> Storage_Layer
        Web --> API
        Core --> SQL
        Core --> NoSQL
    end

    classDef layer fill:#d4edda,stroke:#155724,stroke-width:2px
    classDef app fill:#f8d7da,stroke:#721c24,stroke-width:4px
    class UI_Layer,Logic_Layer,Storage_Layer layer
    class MyApp app
```

```mermaid
graph TB
    App[Application Window]
    
    TopMenu[Top Menu]
    Sidebar[Sidebar]
    Editor[Main Edit Window]
    RightPanel[Right Side Panel]
    
    App --> TopMenu
    App --> Sidebar
    App --> Editor
    App --> RightPanel
    
    TopMenu -->|below| Sidebar
    TopMenu -->|below| Editor
    TopMenu -->|below| RightPanel
    
    Sidebar -->|right of| Editor
    Editor -->|left of| RightPanel
    
    RightPanel --> Win1[Window 1]
    RightPanel --> Win2[Window 2]
    RightPanel --> Win3[Window 3]
    
    Win1 -->|below| Win2
    Win2 -->|below| Win3

    %% Basic styling for clarity
    classDef app fill:#f0f0f0,stroke:#333,stroke-width:2px
    classDef component fill:#e0e0e0,stroke:#666
    class App app
    class TopMenu,Sidebar,Editor,RightPanel,Win1,Win2,Win3 component
```

# UI Diagram for LLM Communication

This is a UI layout for an application window:
- "Top Menu" spans the top horizontally.
- Below it, the layout splits into three vertical sections:
  - "Sidebar" on the left.
  - "Main Edit Window" in the center.
  - "Right Side Panel" on the right, containing three stacked windows ("Window 1", "Window 2", "Window 3").
The Mermaid diagram below uses TB direction to stack elements vertically and arrows to show spatial relationships.

```mermaid
graph TB
    App[Application Window]
    
    TopMenu[Top Menu]
    Sidebar[Sidebar]
    Editor[Main Edit Window]
    RightPanel[Right Side Panel]
    
    App --> TopMenu
    App --> Sidebar
    App --> Editor
    App --> RightPanel
    
    TopMenu -->|below| Sidebar
    TopMenu -->|below| Editor
    TopMenu -->|below| RightPanel
    
    Sidebar -->|right of| Editor
    Editor -->|left of| RightPanel
    
    RightPanel --> Win1[Window 1]
    RightPanel --> Win2[Window 2]
    RightPanel --> Win3[Window 3]
    
    Win1 -->|below| Win2
    Win2 -->|below| Win3

    %% Basic styling for clarity
    classDef app fill:#f0f0f0,stroke:#333,stroke-width:2px
    classDef component fill:#e0e0e0,stroke:#666
    class App app
    class TopMenu,Sidebar,Editor,RightPanel,Win1,Win2,Win3 component
```