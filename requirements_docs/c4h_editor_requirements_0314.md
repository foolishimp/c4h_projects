# C4H Editor MVP Requirements Document

## Overview

The C4H Editor is a client frontend application for the C4H (Code for Humans) service, an intelligent code refactoring system that analyzes and modifies codebases according to specified intents. This document outlines the Minimum Viable Product (MVP) requirements for the C4H Editor.

## System Architecture

The C4H Editor follows a client-server architecture:

1. **C4H Service**: The core service that performs code refactoring using agent teams
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

### 3. Backend Architecture

#### 3.1 State Management
- C4H Editor Backend manages all application state
- Implement persistent storage for work orders and jobs
- Handle version control for work order changes

#### 3.2 C4H Service Integration
- All communication with the C4H service is handled by the backend
- Implement retry logic and error handling for service calls
- Translate between frontend data models and C4H service API

#### 3.3 Storage
- Store work orders as single files for version management
- Implement a version control system in the backend
- Support rollback to previous versions

### 4. Frontend Architecture

#### 4.1 WorkOrder Editor
- Create a tabbed interface aligned with configuration hierarchy sections
- Tabs should include:
  - Intent Configuration
  - Project Configuration
  - LLM Configuration
  - Orchestration Configuration
  - Runtime Configuration

#### 4.2 Content Editors
- Provide appropriate input controls for different configuration types
- Support free-form text editing for complex sections
- Maintain configuration constraints without explicit validation

#### 4.3 WorkOrders Grid
- Implement a data grid for managing work orders
- Support common operations: view, edit, copy, archive, submit
- Include sorting, filtering, and pagination

## Technical Specifications

### Backend

- **Framework**: FastAPI for API endpoints
- **Storage**: File-based storage with Git for version control
- **Authentication**: Basic authentication for MVP
- **Logging**: Structured logging with correlation IDs

### Frontend

- **Framework**: React with TypeScript
- **State Management**: React hooks with context for global state
- **UI Components**: Material-UI for consistent design
- **Code Editor**: Monaco Editor with YAML support

## Data Flow

1. User creates/edits a work order in the frontend
2. Frontend sends work order data to backend
3. Backend validates and stores the work order
4. User submits work order for processing
5. Backend sends work order to C4H service
6. Backend polls C4H service for job status
7. Frontend displays job status and results to user

## Version Management

- Work orders are stored as single files in a Git repository
- Each edit creates a new commit with the changes
- Backend provides access to version history
- Frontend allows viewing and restoring previous versions

## Future Enhancements (Post-MVP)

1. **Team Collaboration**: Multi-user editing and permissions
2. **Advanced Search**: Full-text search across work orders
3. **Templates**: Pre-configured work order templates for common tasks
4. **Visualizations**: Visual representation of job execution flow
5. **Notifications**: Email or webhook notifications for job completion