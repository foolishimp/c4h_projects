# Appendix D: Generic Project Structure and Document Templates

## D.1 Generic Folder Structure Template

The original framework defines five semantic layers but doesn't provide a practical folder structure. Here's the generic template:

```
/project-root/
├── /00-source-materials/              # External inputs (read-only once baselined)
│   ├── /authoritative-sources/        # Laws, standards, regulations, etc.
│   ├── /reference-materials/          # Industry guides, best practices
│   └── /baselines/                    # Snapshot versions for traceability
│
├── /01-strategic-layer/               # Layer 1: Vision and constraints
│   ├── /vision-and-goals/            
│   ├── /constraints/                  # Budget, timeline, regulatory, technical
│   ├── /success-metrics/             
│   ├── /stakeholder-analysis/        
│   └── /ambiguity-catalog/           # Explicit unknowns and assumptions
│
├── /02-functional-layer/              # Layer 2: What the system must do
│   ├── /features/                    
│   ├── /user-stories/                
│   ├── /use-cases/                   
│   ├── /acceptance-criteria/         
│   └── /functional-models/           # Process flows, state machines
│
├── /03-logical-layer/                 # Layer 3: How it's organized (no tech)
│   ├── /domain-models/               
│   ├── /component-models/            
│   ├── /interface-definitions/       
│   ├── /data-models/                 
│   └── /interaction-patterns/        
│
├── /04-technical-layer/               # Layer 4: Specific technical decisions
│   ├── /technology-selections/       
│   ├── /api-specifications/          
│   ├── /data-schemas/                
│   ├── /infrastructure-design/       
│   └── /integration-specifications/  
│
├── /05-implementation-layer/          # Layer 5: Execution artifacts
│   ├── /code-generation-specs/      # For AI-driven development
│   ├── /configuration-templates/     
│   ├── /deployment-specifications/   
│   ├── /test-specifications/         
│   └── /operational-procedures/      
│
├── /06-transformation-metadata/       # How we got from A to B
│   ├── /decision-log/                
│   ├── /traceability-matrix/         
│   ├── /transformation-records/      
│   ├── /constraint-resolutions/      # Pullback documentation
│   └── /iteration-history/           
│
└── /07-governance/                    # Cross-cutting concerns
    ├── /approval-records/            
    ├── /review-cycles/               
    ├── /compliance-evidence/         
    ├── /quality-gates/               
    └── /change-control/              
```

## D.2 Document Naming Conventions

```
[LAYER]-[CATEGORY]-[IDENTIFIER]-[VERSION].[FORMAT]

Examples:
L1-VIS-ProjectVision-v1.0.md
L2-REQ-UserAuthentication-v2.1.yaml
L3-COMP-AuthenticationService-v1.0.puml
L4-API-AuthAPI-v3.0.openapi.yaml
L5-IMPL-AuthServiceSpec-v1.0.md
```

## D.3 Generic Document Templates

### D.3.1 Strategic Intent Document Template (Layer 1)

```markdown
# Strategic Intent: [Project Name]

## Document Metadata
- **Document ID**: L1-STRAT-[ID]-v[X.Y]
- **Status**: [Draft|Review|Approved|Baseline]
- **Created**: [Date]
- **Last Modified**: [Date]
- **Approvers**: [List]

## 1. Vision Statement
[High-level description of desired future state]

## 2. Strategic Objectives
- **Objective 1**: [Description]
  - Success Metric: [Measurable outcome]
  - Target Date: [Date]

## 3. Constraints
### 3.1 External Constraints
- **Regulatory**: [List applicable regulations]
- **Market**: [Market conditions, competition]
- **Industry Standards**: [Relevant standards]

### 3.2 Internal Constraints
- **Budget**: [Amount and conditions]
- **Timeline**: [Key milestones]
- **Resources**: [Team size, skills]
- **Technical**: [Existing systems, platforms]

## 4. Stakeholder Context
| Stakeholder | Interest | Influence | Requirements |
|-------------|----------|-----------|--------------|
| [Name/Role] | [Why they care] | [High/Med/Low] | [What they need] |

## 5. Success Criteria
- **Criterion 1**: [Specific, measurable]
- **Criterion 2**: [Specific, measurable]

## 6. Ambiguity Catalog
### 6.1 Undefined Terms
| Term | Context | Possible Interpretations | Resolution Needed By |
|------|---------|-------------------------|---------------------|
| [Term] | [Where used] | [List options] | [Layer X] |

### 6.2 Implicit Assumptions
- **Assumption 1**: [Description]
  - Risk if invalid: [Impact]
  - Validation method: [How to verify]

### 6.3 Open Questions
- **Question 1**: [Question]
  - Impact: [What depends on answer]
  - Target resolution: [Date/Milestone]

## 7. Traceability
- **Traces From**: [Source documents]
- **Traces To**: [L2 documents that will interpret this]
```

### D.3.2 Functional Requirement Template (Layer 2)

```markdown
# Functional Requirement: [Feature Name]

## Document Metadata
- **Document ID**: L2-FUNC-[ID]-v[X.Y]
- **Traces From**: [L1 document IDs]
- **Status**: [Draft|Review|Approved|Baseline]

## 1. Requirement Overview
### 1.1 Purpose
[Why this feature exists - link to strategic objective]

### 1.2 Scope
- **In Scope**: [What's included]
- **Out of Scope**: [What's explicitly excluded]

## 2. Functional Specification
### 2.1 User Story
As a [user type]
I want [capability]
So that [benefit]

### 2.2 Acceptance Criteria
- [ ] **AC1**: [Specific, testable condition]
- [ ] **AC2**: [Specific, testable condition]

### 2.3 Business Rules
| Rule ID | Description | Condition | Action |
|---------|-------------|-----------|--------|
| BR-001 | [Name] | [When this happens] | [Do this] |

## 3. Behavioral Specification
### 3.1 Scenarios (Gherkin)
```gherkin
Feature: [Feature name]
  
  Scenario: [Scenario name]
    Given [initial context]
    When [action occurs]
    Then [expected outcome]
    And [additional outcomes]
```

### 3.2 Process Flow
[Include diagram or structured description]

## 4. Data Requirements
### 4.1 Input Data
| Data Element | Type | Validation | Source |
|--------------|------|------------|--------|
| [Name] | [Type] | [Rules] | [Where from] |

### 4.2 Output Data
| Data Element | Type | Format | Destination |
|--------------|------|--------|-------------|
| [Name] | [Type] | [Format] | [Where to] |

## 5. Non-Functional Constraints
- **Performance**: [Specific metrics]
- **Security**: [Specific requirements]
- **Usability**: [Specific requirements]

## 6. Ambiguity Resolution
| Original Ambiguity | Resolution | Rationale |
|-------------------|------------|-----------|
| [From L1] | [Specific interpretation] | [Why this choice] |

## 7. Traceability
- **Traces From**: [L1 documents with specific sections]
- **Traces To**: [L3 components that will implement this]
```

### D.3.3 Component Specification Template (Layer 3)

```markdown
# Logical Component: [Component Name]

## Document Metadata
- **Document ID**: L3-COMP-[ID]-v[X.Y]
- **Traces From**: [L2 requirement IDs]
- **Status**: [Draft|Review|Approved|Baseline]

## 1. Component Overview
### 1.1 Purpose
[What this component does and why]

### 1.2 Responsibilities
- **Primary**: [Core responsibility]
- **Secondary**: [Supporting responsibilities]

### 1.3 Boundaries
- **Handles**: [What it's responsible for]
- **Delegates**: [What it passes to others]
- **Assumes**: [What it expects from environment]

## 2. Interface Definition
### 2.1 Provided Interfaces
```
Interface: [Name]
  Operation: [operationName]
    Input: [Type/Structure]
    Output: [Type/Structure]
    Behavior: [What it does]
    Pre-conditions: [What must be true before]
    Post-conditions: [What's true after]
```

### 2.2 Required Interfaces
```
Dependency: [Component/Service Name]
  Interface: [Name]
  Purpose: [Why needed]
  Fallback: [What if unavailable]
```

## 3. Data Model (Logical)
### 3.1 Entities
| Entity | Purpose | Key Attributes | Relationships |
|--------|---------|----------------|---------------|
| [Name] | [Why exists] | [List] | [To other entities] |

### 3.2 State Management
[State diagram or description]

## 4. Interaction Patterns
### 4.1 Sequence Flows
[Key interaction scenarios]

### 4.2 Event Handling
| Event | Source | Response | Side Effects |
|-------|--------|----------|--------------|
| [Name] | [Where from] | [What to do] | [What changes] |

## 5. Quality Attributes
- **Cohesion**: [How focused]
- **Coupling**: [Dependencies and why]
- **Complexity**: [Cyclomatic complexity target]

## 6. Traceability
- **Implements**: [L2 requirements list]
- **Decomposes To**: [L4 technical specs]
```

### D.3.4 Technical Specification Template (Layer 4)

```markdown
# Technical Specification: [Component/API Name]

## Document Metadata
- **Document ID**: L4-TECH-[ID]-v[X.Y]
- **Traces From**: [L3 component IDs]
- **Status**: [Draft|Review|Approved|Baseline]

## 1. Technology Decisions
### 1.1 Technology Stack
| Layer | Technology | Version | Justification |
|-------|------------|---------|---------------|
| [Layer] | [Tech] | [Version] | [Why chosen] |

### 1.2 Key Libraries/Frameworks
| Library | Purpose | License | Risk Assessment |
|---------|---------|---------|-----------------|
| [Name] | [Why needed] | [Type] | [Risks] |

## 2. API Specification
### 2.1 RESTful Endpoints
```yaml
/resource/{id}:
  get:
    summary: [Description]
    parameters:
      - name: id
        type: string
        required: true
    responses:
      200:
        schema: [Reference]
      404:
        description: Not found
```

### 2.2 Data Contracts
```json
{
  "schema": "v1.0",
  "type": "object",
  "properties": {
    "field": {
      "type": "string",
      "validation": "[Rules]"
    }
  }
}
```

## 3. Infrastructure Requirements
### 3.1 Compute Resources
- **CPU**: [Requirements]
- **Memory**: [Requirements]
- **Storage**: [Requirements]

### 3.2 Network Requirements
- **Bandwidth**: [Requirements]
- **Latency**: [Tolerances]
- **Protocols**: [Required protocols]

## 4. Security Implementation
### 4.1 Authentication
[Specific implementation details]

### 4.2 Authorization
[Specific implementation details]

### 4.3 Data Protection
- **At Rest**: [Encryption methods]
- **In Transit**: [TLS configuration]
- **Key Management**: [How handled]

## 5. Performance Specifications
| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| [Metric] | [Value] | [How measured] |

## 6. Deployment Configuration
### 6.1 Environments
| Environment | Purpose | Configuration |
|-------------|---------|---------------|
| [Name] | [Use] | [Key settings] |

### 6.2 Feature Flags
| Flag | Purpose | Default | Rollout Strategy |
|------|---------|---------|------------------|
| [Name] | [Why] | [Value] | [How deployed] |

## 7. Traceability
- **Implements**: [L3 components]
- **Realized By**: [L5 implementation specs]
```

### D.3.5 AI Code Generation Specification Template (Layer 5)

```markdown
# Implementation Specification: [Component Name]

## Document Metadata
- **Document ID**: L5-IMPL-[ID]-v[X.Y]
- **Traces From**: [L4 technical spec IDs]
- **Status**: [Draft|Review|Approved|Baseline]
- **AI Model**: [Claude/GPT/etc version]

## 1. Implementation Context
### 1.1 Purpose
[What this implements and why]

### 1.2 Technical Context
- **Framework**: [Specific framework and version]
- **Dependencies**: [Required libraries]
- **Constraints**: [Technical limitations]

## 2. AI Generation Instructions
### 2.1 System Prompt
```
You are implementing [component] with these constraints:
- Architecture: [Reference L3]
- Technical Spec: [Reference L4]
- Coding Standards: [Reference]
- Security Requirements: [Reference]
```

### 2.2 Specific Constraints
- **MUST**: [Required elements]
- **MUST NOT**: [Prohibited patterns]
- **SHOULD**: [Preferred approaches]
- **MAY**: [Optional elements]

## 3. Interface Implementation
### 3.1 API Contract
[Exact interface to implement]

### 3.2 Error Handling
| Error Condition | Response | Recovery |
|----------------|----------|----------|
| [Condition] | [What to return] | [How to handle] |

## 4. Code Structure
### 4.1 Module Organization
```
component/
├── interfaces/     # Contract definitions
├── core/          # Business logic
├── adapters/      # External integrations
└── tests/         # Test specifications
```

### 4.2 Naming Conventions
- **Classes**: [Pattern]
- **Methods**: [Pattern]
- **Variables**: [Pattern]

## 5. Test Specifications
### 5.1 Unit Tests
| Test Case | Input | Expected Output | Coverage Goal |
|-----------|-------|-----------------|---------------|
| [Name] | [Data] | [Result] | [What it proves] |

### 5.2 Integration Tests
[Test scenarios that verify integration points]

## 6. Quality Gates
### 6.1 Code Quality
- **Coverage**: [Minimum %]
- **Complexity**: [Maximum cyclomatic]
- **Duplication**: [Maximum %]

### 6.2 Security Scans
- **Static Analysis**: [Tools and rules]
- **Dependency Check**: [Vulnerability threshold]

## 7. Validation Criteria
- [ ] Implements all L4 specifications
- [ ] Passes all defined tests
- [ ] Meets quality gates
- [ ] No security vulnerabilities
- [ ] Performance within bounds

## 8. Traceability
- **Implements**: [L4 technical specifications]
- **Validates Against**: [L2 acceptance criteria]
- **Traces To Root**: [L1 strategic objective]
```

### D.3.6 Transformation Record Template

```markdown
# Transformation Record: [Source] → [Target]

## Document Metadata
- **Document ID**: TR-[Source]-[Target]-v[X.Y]
- **Date**: [Transformation date]
- **Performed By**: [Person/Team]
- **Reviewed By**: [Reviewer]

## 1. Source Artifact
- **Document**: [ID and version]
- **Entropy Score**: [Calculated ambiguity]
- **Key Elements**: [List of main points]

## 2. Target Artifact
- **Document**: [ID and version]
- **Entropy Score**: [Should be ≤ source]
- **Key Elements**: [Refined points]

## 3. Transformation Details
### 3.1 Ambiguities Resolved
| Ambiguous Element | Resolution | Justification |
|-------------------|------------|---------------|
| [Element] | [Specific choice] | [Why this choice] |

### 3.2 Decisions Made
| Decision Point | Options Considered | Choice | Rationale |
|----------------|-------------------|---------|-----------|
| [What] | [List] | [Selected] | [Why] |

### 3.3 Constraints Applied
- **From Higher Layers**: [What constrained this]
- **Environmental**: [External constraints]
- **Organizational**: [Internal constraints]

## 4. Pullback Resolutions (if any)
### 4.1 Conflicting Requirements
- **Requirement A**: [Description]
- **Requirement B**: [Description]
- **Common Goal**: [Abstraction]
- **Resolution**: [How satisfied both]

## 5. Validation
### 5.1 Information Preservation
- [ ] All source elements addressed
- [ ] No arbitrary additions
- [ ] Meaning preserved

### 5.2 Entropy Reduction
- **Source Entropy**: [Score]
- **Target Entropy**: [Score]
- **Reduction**: [Percentage]

### 5.3 Traceability
- [ ] All target elements trace to source
- [ ] Links are explicit and clear
- [ ] Rationale documented

## 6. Approval
- **Technical Review**: [Name, Date]
- **Business Review**: [Name, Date]
- **Quality Gate Passed**: [Yes/No]
```

## D.4 Metadata Standards

Every document should include:

```yaml
metadata:
  document_id: "[LAYER]-[TYPE]-[ID]-v[VERSION]"
  title: "[Descriptive Title]"
  status: "[Draft|Review|Approved|Baseline|Deprecated]"
  version: "[MAJOR.MINOR.PATCH]"
  created_date: "[YYYY-MM-DD]"
  modified_date: "[YYYY-MM-DD]"
  author: "[Name/Team]"
  reviewers: ["[List of reviewers]"]
  approvers: ["[List of approvers]"]
  traces_from: ["[Source document IDs]"]
  traces_to: ["[Target document IDs]"]
  tags: ["[Searchable tags]"]
  entropy_score: "[0.0-1.0]"
  change_log:
    - version: "[VERSION]"
      date: "[DATE]"
      author: "[NAME]"
      changes: "[Description]"
      rationale: "[Why changed]"
```

## D.5 File Format Guidelines

| Document Type | Preferred Format | Rationale |
|--------------|------------------|-----------|
| Strategic Documents | Markdown (.md) | Version control friendly, readable |
| Requirements | YAML or Markdown | Structured data with narrative |
| Models | PlantUML or Mermaid | Text-based, versionable diagrams |
| API Specs | OpenAPI YAML | Industry standard, tooling support |
| Test Specs | Gherkin (.feature) | Executable specifications |
| Data Schemas | JSON Schema | Validation tooling available |
| Configurations | YAML | Human readable, structured |
| Traceability Matrix | CSV or YAML | Tool processable |

## D.6 Version Control Practices

### Branching Strategy for Documents
```
main (baseline documents)
├── draft/[document-id] (work in progress)
├── review/[document-id] (under review)
└── release/[version] (approved bundles)
```

### Commit Message Format
```
[LAYER] [ACTION]: Brief description

- Added: [What was added]
- Changed: [What was modified]
- Removed: [What was deleted]
- Traces: [Updated traceability]

Resolves: [Issue/Ambiguity ID]
```

This appendix provides the practical templates and structures that were missing from the original framework document, making it actionable for real projects while maintaining the theoretical rigor of the semantic transformation approach.