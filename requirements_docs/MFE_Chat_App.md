# **Requirements Document: LLM Chat Client Microfrontend (MFE) for C4H Editor**

Version: 1.0  
Date: 2025-04-24  
**1. Introduction**

* **1.1. Purpose:** This document outlines the functional and technical requirements for developing and integrating a new Microfrontend (MFE) into the C4H Editor application. This MFE will provide users with an interactive chat interface powered by Large Language Models (LLMs).  
* **1.2. Chosen Library:** The implementation will utilize the open-source assistant-ui React library ([https://github.com/assistant-ui/assistant-ui](https://github.com/assistant-ui/assistant-ui)) for its chat UI components and features.  
* **1.3. Goal:** To create a chat client MFE that seamlessly integrates with the existing C4H Editor shell, adheres to established MFE guidelines, supports interactions with OpenAI, Anthropic, and Google LLM providers, and potentially interacts with other C4H components via the event bus.

**2. References**

* C4H Editor MFE Guidelines (MFE_Guidelines_v2.md) - *See Appendix A*  
* C4H Editor Shell Service API Documentation (c4h_shellservice_api_doc_v1.md) - *See Appendix B*  
* C4H Editor Main Backend API Documentation (c4h_api_doc_v1.md, c4h_configjobs_api_doc_v1.md) - *See Appendix C & D*  
* assistant-ui Documentation & GitHub Repository - *External Reference: [https://github.com/assistant-ui/assistant-ui](https://github.com/assistant-ui/assistant-ui)*  
* C4H Editor Shared Package (c4h-micro/packages/shared) implementation details. - *Code Reference*  
* C4H TypeScript Design Principles (Typescript_Design_Principles.md) - *See Appendix E*

**3. Core Functional Requirements**

* **3.1. Chat Interface:** Provide a user interface for sending text prompts and receiving responses from configured LLMs.  
* **3.2. LLM Provider Support:** The MFE must support configuration and communication with APIs from:  
  * OpenAI  
  * Anthropic  
  * Google (Gemini)  
* **3.3. Basic Chat Features:** Leverage assistant-ui to provide standard chat features, including:  
  * Message input area.  
  * Display of conversation history (user prompts and LLM responses).  
  * Markdown rendering for responses.  
  * Code highlighting within responses.  
  * Auto-scrolling.  
* **3.4. C4H Context Awareness (Optional):** The chat client should ideally be able to receive context information from other C4H Editor components (e.g., current file, selected work order) via the event bus to provide more relevant LLM responses.  
* **3.5. C4H Action Integration (Optional):** The chat client could potentially trigger actions within C4H based on LLM responses (e.g., applying generated code to an editor) by publishing events on the event bus.

**4. Technical Requirements**

* **4.1. Technology Stack:**  
  * Frontend Framework: React  
  * Language: TypeScript (adhering to Typescript_Design_Principles.md)  
  * Build Tool: Vite  
  * Chat UI Library: assistant-ui  
  * State Management: Local component state or shared context if necessary.  
* **4.2. Licensing:**  
  * The assistant-ui library is licensed under the MIT License, which is compatible with the project's usage. All custom code developed for this MFE will adhere to the project's overall licensing.  
* **4.3. Architecture:**  
  * The chat client must be implemented as a distinct MFE package within the c4h-micro/packages/ directory (suggested name: chat-client).  
  * It must strictly adhere to the principles and patterns defined in MFE_Guidelines_v2.md, including independent initialization and runtime isolation.  
* **4.4. MFE Implementation:**  
  * **4.4.1. Wrapper Component:** A primary React component (e.g., ChatClientWrapper.tsx) shall be created to host and configure the assistant-ui components.  
  * **4.4.2. Bootstrap Function:** The MFE must export an async function bootstrapConfig(mfeId: string) as per guidelines. This function must fetch the central shell configuration via window.__C4H_SHELL_SERVICE_URL__ and configure the shared apiService using configureApiService(config.serviceEndpoints.jobConfigServiceUrl).  
  * **4.4.3. Mount Function:** The MFE must export a function mount(props: { domElement: HTMLElement, customProps?: any }) as per guidelines. This function must render the ChatClientWrapper within the domElement, ensuring it is wrapped with standard C4H providers (ThemeProvider, CssBaseline). It must return an unmount function.  
  * **4.4.4. Dependencies:** The chat-client MFE's package.json must list assistant-ui and its peer dependencies. It must also depend on the C4H shared package ("shared": "workspace:*").  
* **4.5. Backend & Service Integration:**  
  * **4.5.1. LLM API Configuration:** Configuration for LLM providers (API endpoints, models) and credentials (API Keys) must be handled securely. **Credentials must not be hardcoded in the frontend code.** Potential approaches include fetching from a secure backend endpoint or using environment variables configured during deployment.  
  * **4.5.2. C4H Backend (apiService):** The MFE must correctly initialize the shared apiService via bootstrapConfig. Any necessary interactions with the C4H main backend (e.g., saving chat history associated with C4H entities) must use this service and check checkApiServiceReady() beforehand.  
* **4.6. C4H Shell Integration:**  
  * **4.6.1. MFE Definition (shell_service):** An entry for id: "chat-client" must be added to the available_apps configuration in the shell_service (database and potentially defaults.json), specifying type: "ESM".  
  * **4.6.2. Environment Configuration (environments.json):** An entry for "chat-client" must be added to the root environments.json file. The "development" URL must point to the MFE's local Vite dev server (e.g., "http://localhost:3107/src/main.tsx").  
  * **4.6.3. Shell UI:** The "chat-client" MFE must be assignable to a Frame (tab) via the Shell Preferences dialog.  
* **4.7. Communication (eventBus):**  
  * The MFE must use the global window.__C4H_EVENT_BUS__ for any necessary cross-MFE communication, subscribing to or publishing events as defined in the shared package or newly specified event types.

**5. Non-Functional Requirements**

* **5.1. UI/UX:**  
  * The chat interface must visually align with the C4H Editor's Material UI theme.  
  * The chat experience should be responsive and intuitive.  
  * LLM response streaming should be clearly indicated.  
* **5.2. Performance:**  
  * The MFE load time should be minimal.  
  * Chat interactions should feel fluid.  
  * Resource consumption (memory, CPU) should be reasonable.  
* **5.3. Error Handling:**  
  * Gracefully handle errors from LLM API calls (e.g., network issues, API key errors, rate limits).  
  * Provide informative feedback to the user when errors occur.  
  * Implement React Error Boundaries to prevent UI crashes.  
* **5.4. Security:**  
  * API keys and sensitive configuration must be handled securely and not exposed in the frontend bundle.  
  * Input sanitization should be considered if user input is used in ways other than direct LLM prompts.

**6. Future Considerations (Optional)**

* Integration with C4H Work Orders/Jobs for context.  
* Ability to save/load chat sessions.  
* Support for file uploads/analysis within the chat.  
* Advanced agentic features leveraging assistant-ui capabilities (tool calls, generative UI).

**Appendix**

**Appendix A: MFE Guidelines (MFE_Guidelines_v2.md)**

# Microfrontend Integration Guidelines

## 1. Core Principles

- **Independent Initialization**: Each MFE is responsible for its own configuration and initialization  
- **Runtime Isolation**: MFEs should function independently with minimal runtime dependencies  
- **Clear Communication Patterns**: Use standardized methods for cross-MFE communication  
- **Self-contained Builds**: Bundle shared code into consumers rather than relying on runtime resolution

## 2. Architecture Overview

- **Shell Application**: Orchestrates MFEs and exposes minimal global resources  
- **Microfrontends (MFEs)**: Independent applications following a consistent initialization pattern  
- **Shell Service**: Central configuration source accessed directly by all MFEs  
- **Event Bus**: Global communication mechanism accessible by all applications

## 3. Shell Responsibilities

- Render the main UI frame and provide navigation controls  
- Inject shell service URL as a global: `window.__C4H_SHELL_SERVICE_URL__ = 'http://localhost:8011'`  
- Mount MFEs and trigger their bootstrap process: `mfe.bootstrapConfig?.(appId)`  
- Provide navigation callbacks as props to MFEs  
- Initialize and expose the singleton event bus instance: `window.__C4H_EVENT_BUS__`

## 4. MFE Implementation Requirements

- Export a standard bootstrap interface alongside mount function  
- Always check configuration state before performing API operations  
- Include required React context providers when mounting  
- Handle cases where configuration isn't immediately available  
- Manage internal state independently of other MFEs

## 5. Self-Initialization Pattern

All MFEs must implement this bootstrap pattern:

```typescript  
export async function bootstrapConfig(mfeId: string) {  
  // Get shell service URL from window global  
  const shellServiceUrl = window.__C4H_SHELL_SERVICE_URL__;  
  if (!shellServiceUrl) {  
    console.error(`[${mfeId}] Shell service URL not found`);  
    return { success: false, error: "Shell service URL not found" };  
  }

  try {  
    const response = await fetch(`${shellServiceUrl}/api/v1/shell/configuration`);  
    const config = await response.json();

    // Configure this MFE's services  
    configureApiService(config.serviceEndpoints.jobConfigServiceUrl);

    console.log(`[${mfeId}] Successfully bootstrapped with config`);  
    return { success: true, config };  
  } catch (error) {  
    console.error(`[${mfeId}] Bootstrap failed:`, error);  
    return { success: false, error };  
  }  
}

## **6. Communication Patterns**

* **Direct Props**: For parent-child communication (Shell to MFE)  
* **Event Bus**: For cross-MFE communication using standard event types  
* **URL Parameters**: For initial state or deep linking  
* **Shell Service API**: For configuration and preference management

## **7. Common Failures and Risks**

### **API Service Configuration Issues**

**Problem**: MFEs attempt API operations before being properly configured with endpoints

* **Symptoms**: API calls fail with "API service not ready" errors  
* **Solution**: Always check checkApiServiceReady() before operations, retry or block UI as needed

### **Multiple Service Instances**

**Problem**: Each MFE using its own instance of shared services loses configuration state

* **Symptoms**: One MFE configures a service but another finds it unconfigured  
* **Solution**: Use self-initialization pattern or global instances

### **React Context Boundary Issues**

**Problem**: MUI components fail because they can't access ThemeProvider context

* **Symptoms**: "Invalid hook call" errors when crossing MFE boundaries  
* **Solution**: Always include required providers when mounting MFEs

### **Race Conditions During Initialization**

**Problem**: Operations attempted during partial initialization state

* **Symptoms**: Inconsistent behavior, timing-dependent failures  
* **Solution**: Implement proper loading states and initialization completion checks

### **Configuration Type Mismatches**

**Problem**: Shell and MFEs using different naming conventions for configs

* **Symptoms**: Empty data or "not found" errors  
* **Solution**: Normalize config types between shell and components

## **8. Implementation Template for New MFEs**

// src/main.tsx

import React from 'react';  
import { createRoot } from 'react-dom/client';  
import { ThemeProvider, createTheme, StyledEngineProvider } from '@mui/material/styles';  
import CssBaseline from '@mui/material/CssBaseline';  
import MyComponent from './MyComponent';

// Export the bootstrap function for shell to call  
export async function bootstrapConfig(mfeId) {  
  const shellServiceUrl = window.__C4H_SHELL_SERVICE_URL__;  
  if (!shellServiceUrl) return { success: false, error: "Shell service URL not found" };

  try {  
    const response = await fetch(`${shellServiceUrl}/api/v1/shell/configuration`);  
    const config = await response.json();

    // Configure API service with correct endpoint  
    configureApiService(config.serviceEndpoints.jobConfigServiceUrl);  
    return { success: true, config };  
  } catch (error) {  
    return { success: false, error };  
  }  
}

// Standard mount function with required providers  
export function mount(props) {  
  const { domElement, customProps = {} } = props;

  // Bootstrap if not already done (can be called again by shell)  
  if (typeof bootstrapConfig === 'function') {  
    bootstrapConfig(customProps.appId || 'unknown-mfe');  
  }

  const root = createRoot(domElement);  
  root.render(  
    <StyledEngineProvider injectFirst>  
      <ThemeProvider theme={createTheme()}>  
        <CssBaseline />  
        <MyComponent {...customProps} />  
      </ThemeProvider>  
    </StyledEngineProvider>  
  );

  return {  
    unmount() {  
      root.unmount();  
    }  
  };  
}

export default MyComponent;

**Appendix B: Shell Service API Documentation (`c4h_shellservice_api_doc_v1.md`)**

```markdown  
**LLM Guide: C4H Editor Shell Service**

**1. Purpose & Intent**

* The shell_service is a backend API (built with FastAPI/Python) that acts as the central configuration provider for the C4H Editor frontend shell.  
* Its primary responsibilities are:  
  * Telling the frontend shell which "frames" or tabs to display and in what order.  
  * Defining which microfrontend applications (MFEs) are available and where to load them from (their URLs).  
  * Persisting user preferences regarding frame layout and assigned applications.  
  * Providing URLs for other necessary backend services.

**2. Core Data Structures**

*(These represent the JSON structures exchanged with the frontend)*

* **AppDefinition**: Describes an available MFE.  
  * id: string (Unique identifier, e.g., "test-app", "config-selector-workorders")  
  * name: string (Display name, e.g., "Test App", "Work Order Config")  
  * type: string (How to load the MFE: "esm", "iframe", "wc")  
  * url: string (The URL to load the MFE from. Resolved by the service based on environment.)  
* **AppAssignment**: Defines which MFE is assigned to a slot within a frame.  
  * appId: string (References the id of an AppDefinition)  
  * config?: any (Optional MFE-specific configuration - currently seems unused)  
* **Frame**: Represents a tab or view container in the shell.  
  * id: string (Unique identifier for the frame, e.g., "new-1745237898812")  
  * name: string (Display name for the tab, e.g., "Test Tab", "Work Orders")  
  * order: int (Determines the display order in the tab bar)  
  * assignedApps: List[AppAssignment] (List of MFEs assigned to this frame. Currently, seems to handle only one app per frame).  
* **ShellConfigurationResponse**: The main object returned on initial load.  
  * frames: List[Frame] (The user's current frame layout)  
  * availableApps: List[AppDefinition] (All MFEs known to the system)  
  * serviceEndpoints: Dict[string, string] (Map of service names to URLs, e.g., "mainBackendUrl": "http://...", "prefsServiceUrl": "http://...")

**3. Key API Endpoints**

* **GET /api/v1/shell/configuration**  
  * **Purpose:** Fetches the initial state needed by the frontend shell to render the UI and load MFEs.  
  * **User Context:** Implicitly uses a default user (default_user) for retrieving preferences.  
  * **Logic:**  
    1. Retrieves the default list of availableApps.  
    2. Retrieves the default frame structure.  
    3. Loads user-specific saved frames preference from the database (SQLite). If found, these override the default frames.  
    4. Resolves the url for each AppDefinition in availableApps. It prioritizes URLs defined in the environments.json file (matching the AppDefinition.id or config-selector-* pattern) for the current runtime environment (e.g., "development"). Falls back to default URLs if not found in environments.json.  
    5. Retrieves other backend service endpoints (e.g., the URL for the main backend, its own preferences endpoint URL).  
    6. Returns the data structured as ShellConfigurationResponse.  
  * **Response Body:** ShellConfigurationResponse (JSON)  
  * **Frontend Usage:** Called once by the shell on startup (ShellConfigContext) to initialize the application layout and know where to load MFEs from.  
* **PUT /api/v1/shell/preferences**  
  * **Purpose:** Saves the user's current layout preferences (which tabs exist, their names, order, and assigned MFE).  
  * **User Context:** Saves preferences for the default user (default_user).  
  * **Request Body:** List[Frame] (JSON) - An array representing the complete desired state of the frames (tabs). The structure matches the Frame object described above.  
  * **Logic:**  
    1. Receives the list of Frame objects.  
    2. Validates the input.  
    3. Overwrites the existing preferences for the user in the database with this new layout.  
  * **Response Body:** { "message": "Preferences saved successfully" } or an error message (JSON).  
  * **Frontend Usage:** Called by the shell's "Preferences" dialog when the user clicks "Save". The frontend sends the current state of the configured frames as the request body.

**4. Interaction with Frontend**

* The frontend shell relies on the GET /configuration endpoint during initialization to build its tab bar and determine which MFE URLs to use for dynamic imports.  
* The frontend shell uses the PUT /preferences endpoint when the user saves changes made in the preferences dialog (adding, removing, reordering tabs, or changing the assigned app).  
* The serviceEndpoints provided by GET /configuration are used by the shared apiService to configure base URLs for other backend interactions.

**5. Development Environment Considerations**

* The shell_service reads an environments.json file at startup.  
* The url fields within the development section of this file directly override the default URLs for MFEs, allowing developers to point the shell to local vite dev servers for each MFE using http://localhost:<port>/src/main.tsx style URLs.

---

This guide should provide sufficient context for an LLM (or a developer) working on the frontend to understand how the shell interacts with its configuration backend, what data structures to expect, and the purpose of the key API calls, without needing the backend implementation details.

**Appendix C: Main Backend API Documentation (c4h_api_doc_v1.md)**

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

POST /api/v1/jobs

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

#### **Response**

{  
  "job_id": "job_20250414_110500_f3a1b2c5",  
  "status": "submitted", // Or 'created' depending on immediate processing  
  "storage_path": "workspaces/lineage/wf_...", // Path if lineage enabled  
  "error": null  
}

### **Check Job Status**

GET /api/v1/jobs/{job_id}

Retrieves the current status and results (if completed) for a specific job.

#### **Response (JobStatus model)**

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

### **List Jobs**

GET /api/v1/jobs

This endpoint returns a list of all jobs, with optional pagination and filtering.

#### **Query Parameters**

|

| Parameter | Type | Description |  
| page | integer | Page number (default: 1) |  
| limit | integer | Items per page (default: 25) |  
| status | string | Filter by status (e.g., "submitted", "success") |

#### **Response**

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

### **Cancel Job**

POST /api/v1/jobs/{job_id}/cancel

Attempts to cancel a job that is currently in a submitted or running state. Cancellation is best-effort.

#### **Response**

{  
  "job_id": "job_20250414_110500_f3a1b2c5",  
  "status": "cancelled", // Or the status at the time of cancellation attempt  
  "storage_path": "workspaces/lineage/wf_...",  
  "error": null // May contain an error if cancellation failed immediately  
}

### **Retrieve Logs**

GET /api/v1/logs?job_id={job_id}&level=INFO&start_time=2025-04-02T12:00:00Z

#### **Query Parameters**

| Parameter | Type | Description |  
| job_id | string | Filter logs by job ID |  
| level | string | Filter by log level (DEBUG, INFO, WARNING, ERROR) |  
| start_time | ISO datetime | Filter logs after this time |  
| end_time | ISO datetime | Filter logs before this time |  
| limit | integer | Maximum number of log entries to return |

#### **Response**

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

### **Merge Configurations Utility**

POST /api/v1/configs/merge

Utility endpoint to test how configuration fragments will be merged by the server. Useful for debugging configuration overrides.

#### **Request Body**

{  
    "configs": [  
        // List of config dicts to merge (later items override earlier ones)  
        {"runtime": {"logging": {"level": "INFO"}}},  
        {"runtime": {"logging": {"level": "DEBUG", "format": "structured"}}}  
    ],  
    "include_system_config": true // Optional: Merge onto base system config (default: true)  
}

#### **Response (MergeResponse model)**

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

### **Health Check**

GET /health

Provides a simple health status check for the API service.

#### **Response**

{  
  "status": "healthy",  
  "version": "1.1.0", // Example version  
  "workflows_tracked": 15, // Example count  
  "teams_available": 4 // Example count  
}

### **Get Configuration Types**

GET /api/v1/config-types

Retrieves the list of configuration types known and managed by the backend (e.g., workorder, teamconfig).

#### **Response**

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

## **Configuration Structure**

While the API accepts fragments in the configs list, these fragments typically correspond to sections of the overall configuration structure used internally (Workorder, Team, Runtime). Understanding this structure helps in creating effective fragments. See the SystemConfig_Design_Guide.md for full details on the internal configuration structure.

## **Frontend Job API Implementation**

The frontend implementation for the Jobs API follows these design patterns:

### **1. Job Context Management**

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

### **2. Type Definitions**

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

### **3. API Service Implementation**

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

### **4. Job Listing Component**

*(... Job Listing Component example remains largely the same, focusing on displaying data from listJobs ...)*

### **5. Job Details Component**

*(... Job Details Component example remains largely the same, focusing on displaying data from getJobStatus and potentially getLogs ...)*

### **6. Job Request Form Component**

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

### **7. Log Viewer Component**

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

### **8. Lineage Viewer Component**

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

### **7. Error Handling Utilities**

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

### **8. Implementation Notes**

1. **Type Safety**  
   * All components use TypeScript interfaces that match the backend API  
   * Strict prop validation with React PropTypes as a secondary check  
   * Comprehensive logging type definitions  
   * Runtime type checking for API responses  
2. **State Management**  
   * Job context maintains global job state  
   * Component state for UI controls and loading states  
   * Local storage for persistent job history  
3. **Error Handling**  
   * Consistent error handling patterns across components  
   * Error boundaries for component-level failures  
   * Detailed error messages with user-friendly suggestions  
   * Clear visual indicators for error states  
4. **Performance Considerations**  
   * Optimized rendering with React.memo and useMemo  
   * Efficient polling with dynamic intervals  
   * Pagination for job history  
   * Cached responses for frequently accessed data  
   * Optimized log filtering and rendering  
5. **Observability Features**  
   * Structured logging with consistent format  
   * Lineage tracking for workflow traceability  
   * Correlation IDs across components  
   * Filterable log viewer with search capabilities  
6. **Security Considerations**  
   * API key management with environment variables  
   * Input validation for all form submissions  
   * Secure storage of job history  
   * Authentication integration points  
   * Rate limiting for API requests  
7. **Accessibility Features**  
   * ARIA attributes for interactive components  
   * Keyboard navigation support  
   * Screen reader compatible status indicators  
   * Color contrast compliance  
   * Focus management for modal dialogs  
8. **Deployment Considerations**  
   * Environment-specific configuration  
   * API base URL configuration  
   * Error tracking integration  
   * Analytics integration  
   * Feature flagging support

**Appendix D: Config/Jobs Backend API Documentation (`c4h_configjobs_api_doc_v1.md`)**

```markdown  
**LLM Guide: C4H Editor Main Backend Service (backend)**

**1. Purpose & Intent**

* The backend service is the primary API for managing core application data within the C4H Editor, specifically focusing on **Configurations** (like Work Orders, Teams, Runtimes) and background **Jobs**.  
* It handles Create, Read, Update, Delete (CRUD) operations for these configurations.  
* It manages the lifecycle of background jobs, likely related to processing or utilizing the configurations.  
* It acts as an intermediary, communicating with an external **"C4H Service"** for certain operations (details likely specified in design documents).  
* It provides the necessary data persistence layer for configurations and jobs.

**2. Technology Stack**

* Assumed: FastAPI (Python framework based on start_backends.py using uvicorn).  
* Data Persistence: Likely a relational database (specifics depend on implementation, could be PostgreSQL, SQLite, etc.).

**3. Key Responsibilities**

* CRUD operations for different configuration types (Work Orders, Teams, Runtimes, etc.).  
* Submitting, monitoring, and managing background jobs.  
* Validating configuration data.  
* Interfacing with the external C4H Service.  
* Serving data required by the config-selector-* MFEs and the job-management MFE.

**4. Core Data Structures (Anticipated)**

*(These are based on typical REST API patterns and likely detailed in c4h_jobs_config_dmodel.md and other design guides. Exact field names might vary.)*

* **ConfigBase (Conceptual)**: Common fields for all config types.  
  * id: string (UUID or unique identifier)  
  * name: string  
  * description?: string  
  * config_type: string (e.g., "workorder", "team", "runtime")  
  * created_at: string (ISO 8601 timestamp)  
  * updated_at: string (ISO 8601 timestamp)  
  * data: object (The actual configuration payload specific to the type)  
* **WorkOrderConfig**: Extends ConfigBase, data contains work order specific fields.  
* **TeamConfig**: Extends ConfigBase, data contains team specific fields.  
* **RuntimeConfig**: Extends ConfigBase, data contains runtime specific fields.  
* **Job**: Represents a background job.  
  * id: string (UUID or unique identifier)  
  * config_id: string (ID of the config used by the job)  
  * config_type: string  
  * status: string (e.g., "pending", "running", "completed", "failed", "cancelled")  
  * progress?: number (0-100)  
  * result?: any (Job output/result upon completion)  
  * error_message?: string (If status is "failed")  
  * created_at: string (ISO 8601 timestamp)  
  * updated_at: string (ISO 8601 timestamp)  
* **JobListResponse**: For listing jobs.  
  * jobs: List[Job]  
  * total: int  
  * page?: int  
  * size?: int

**5. Key API Endpoints (Inferred from Frontend Usage)**

*(Base Path: /api/v1)*

* **Configurations:**  
  * GET /configs/{config_type}  
    * **Purpose:** List all configurations of a specific type (e.g., /configs/workorder).  
    * **Response:** List[ConfigBase] (or specific type list like List[WorkOrderConfig]).  
  * POST /configs/{config_type}  
    * **Purpose:** Create a new configuration of the specified type.  
    * **Request Body:** Object containing fields for the new config (e.g., name, description, data).  
    * **Response:** The newly created ConfigBase (or specific type) object, including its assigned id.  
  * GET /configs/{config_type}/{config_id}  
    * **Purpose:** Retrieve the details of a single configuration by its type and ID.  
    * **Response:** The specific ConfigBase (or derived type) object.  
  * PUT /configs/{config_type}/{config_id}  
    * **Purpose:** Update an existing configuration.  
    * **Request Body:** Object containing the fields to update.  
    * **Response:** The updated ConfigBase (or derived type) object.  
  * DELETE /configs/{config_type}/{config_id}  
    * **Purpose:** Delete a specific configuration.  
    * **Response:** Success/failure message (e.g., { "message": "Config deleted" }).  
* **Jobs:**  
  * GET /jobs/  
    * **Purpose:** List background jobs. May support query parameters for filtering (e.g., by status, config_type, pagination).  
    * **Response:** JobListResponse.  
  * POST /jobs/  
    * **Purpose:** Create/submit a new background job, likely referencing one or more configuration IDs.  
    * **Request Body:** Object specifying the job type, target configuration(s), and any parameters.  
    * **Response:** The newly created Job object, likely with status "pending" or "submitted".  
  * GET /jobs/{job_id}  
    * **Purpose:** Get the current status and details of a specific job.  
    * **Response:** The Job object.  
  * DELETE /jobs/{job_id}  
    * **Purpose:** Request cancellation or deletion of a job (depending on implementation).  
    * **Response:** Success/failure message or updated Job object.

**6. Interaction with Frontend**

* The config-selector-* MFEs use the /configs/* endpoints extensively to list, view, create, edit, and delete configurations relevant to their type.  
* The job-management MFE uses the /jobs/* endpoints to list jobs, view their status, submit new jobs (likely after selecting configurations), and potentially cancel jobs.  
* All interactions from the frontend go through the shared apiService, which should be configured by the ShellConfigContext with the base URL for this backend service (e.g., http://localhost:8011).

**7. External C4H Service Interaction**

* This backend service communicates with another, presumably more authoritative or specialized "C4H Service".  
* The exact nature of this interaction (e.g., fetching data, validating configs, submitting final job requests) should be detailed in the specific design documents (WorkOrder_Design_Guide.md, Jobs_Design_Guide_0402.md, c4h_api_doc_v1.md). The frontend generally doesn't need to know the specifics, only that this backend handles that communication.

---

This guide provides the necessary context about the backend's role, data, and API surface for an LLM or developer working primarily on the frontend MFEs that interact with it. For exact request/response schemas and behaviour details, the referenced design documents should be consulted.

**Appendix E: TypeScript Design Principles (Typescript_Design_Principles.md)**

# TypeScript Development Principles for Code Generation

## Core Architecture

### Component Design  
- **Single Responsibility Principle**: One component = one responsibility  
- **Pure Render Functions**: Components should be pure, deterministic renderers  
- **Props-Only Communication**: Components interact only through typed props  
- **Container/Presenter Pattern**: Separate data handling from presentation  
- **Composition Over Inheritance**: Favor component composition over class inheritance  
- **Interface-Driven Development**: Define component interfaces before implementation

### Type System  
- **Zero Tolerance for `any`**: Never use `any` or implicit any  
- **Never Type Assert**: Avoid `as` casts; use type guards instead  
- **Discriminated Unions**: Use tagged unions for multi-state data  
- **Readonly by Default**: Use readonly for arrays and object properties  
- **Exhaustive Checks**: Ensure all cases are handled with type narrowing  
- **No Partial Types**: Always require complete objects; use Omit/Pick for variations

### State Management  
- **Lowest Level State**: Keep state at lowest component level possible  
- **Single Source of Truth**: Never duplicate state across components  
- **Immutable Updates**: Always use immutable patterns for state updates  
- **Normalized State**: Store relational data in normalized form  
- **State Machines**: Model complex state transitions as state machines

### API Design  
- **Contract-First Development**: Define API interfaces before implementation  
- **Functional Core**: Pure business logic functions with immutable data  
- **Centralized API Access**: Use hooks for API access, never call directly from components  
- **Error Boundaries**: Wrap async operations in try/catch with typed errors  
- **Loading States**: Model loading with discriminated unions, not booleans

## Implementation Patterns

### Export/Import  
- **Named Exports Only**: Use named exports exclusively, never default exports  
- **Export Interface with Component**: Always export component props interface  
- **Re-export from Barrels**: Use index files to simplify imports  
- **Import Sorting**: Group imports by source: external, internal, relative  
- **No Wildcard Imports**: Always destructure imports explicitly

### Functions & Methods  
- **Function over Class**: Prefer pure functions to methods  
- **Arrow Functions**: Use arrow functions for callbacks to preserve `this`  
- **Named Functions**: Name all functions for debugging, even callbacks  
- **Parameter Destructuring**: Destructure objects in parameters  
- **Explicit Return Types**: Always specify return types on functions  
- **Factory Functions**: Use factories for complex object creation

### Component Props  
- **Required vs Optional**: Make props optional only when truly optional  
- **Handler Naming**: `onX` in props interfaces maps to `handleX` in parent  
- **Comprehensive PropTypes**: Document all props with JSDoc comments  
- **Prop Defaults**: Handle defaults with destructuring, not component internals  
- **Callback Signatures**: Use explicit parameter types on callbacks

### Error Handling  
- **Early Returns**: Use guard clauses for invalid states  
- **Error Objects**: Use structured error objects, not strings  
- **Fallback UI**: Provide fallback UI for error states  
- **Typed Errors**: Use discriminated union for different error types  
- **Error Boundaries**: Implement React error boundaries at logical points

### Performance  
- **Memoization**: Use React.memo for expensive pure components  
- **Callback Memoization**: useCallback for all handler functions passed as props  
- **Derived State**: Calculate derived state with useMemo, not in render  
- **Virtual Lists**: Use virtualization for long lists  
- **Lazy Loading**: Use React.lazy for code-splitting large components

## Enhanced Type System Guidelines

### Enum Types & Constants  
- **Centralized Type Definitions**: Define all enums, constants, and shared types in dedicated type files  
- **Export All Types**: Never use types locally when they should be shared across components  
- **String Literal Unions**: For simple cases, prefer string literal unions over enums  
- **Runtime Type Safety**: Ensure enum values can be used at runtime if needed

### Third-Party Library Integration  
- **Material-UI Type Constraints**:  
  - `MenuItem` values must be strings/numbers, never booleans  
  - Always check component prop requirements in library documentation  
- **Library-Specific Types**: Import and use library-provided types rather than creating duplicates  
- **Type Conversion**: When passing data to external libraries, explicitly convert to expected types

### Value Type Handling  
- **Boolean Representation**: Convert booleans to strings/numbers when library components require it  
- **Type-Safe Conversion**: Use explicit conversion functions rather than type casting  
- **Consistent Conventions**: Use 'true'/'false' strings or 1/0 numbers consistently

## Code Quality

### Style & Formatting  
- **Prettier Configuration**: Use consistent auto-formatting  
- **Naming Convention**: PascalCase for components/types, camelCase for functions/variables  
- **Consistent Quotes**: Use single quotes for strings, double for JSX attributes  
- **Function Expression Style**: Use arrow functions for variable assignment  
- **Component Delineation**: Use // ---- comments to separate logical sections

### Modularity & Testability  
- **Pluggable Dependencies**: Use dependency injection patterns for external services  
- **Pure Business Logic**: Extract pure logic functions from components  
- **Interface Boundaries**: Define clear interfaces between modules  
- **Mock-Friendly Design**: Design components and services to be easily mockable  
- **Small Surface Areas**: Keep component API surface areas small and focused  
- **Stateless When Possible**: Prefer stateless components when functionality allows

### Project Structure  
- **Feature Folders**: Organize by feature, not type  
- **Co-location**: Place related files together  
- **Shared Components**: Use `/components/shared` for reusable components  
- **Consistent Paths**: Use path aliases to avoid `../../../` imports  
- **Module Boundaries**: Enforce clear boundaries between modules  
- **Circular Dependencies**: Prevent circular dependencies with architecture

## Specific Patterns

```typescript  
// Component definition with exported props interface  
export interface ButtonProps {  
  label: string;  
  onClick: () => void;  
  variant?: 'primary' | 'secondary';  
  disabled?: boolean;  
}

export const Button = ({  
  label,  
  onClick,  
  variant = 'primary',  
  disabled = false  
}: ButtonProps): JSX.Element => {  
  // Implementation  
};

// Discriminated union for API state  
type ApiState<T> =  
  | { status: 'idle' }  
  | { status: 'loading' }  
  | { status: 'success'; data: T }  
  | { status: 'error'; error: Error };

// Usage with type narrowing  
if (state.status === 'success') {  
  return <DataDisplay data={state.data} />;  
}

// Safe date handling  
const createMetadata = (): WorkOrderMetadata => ({  
  author: getCurrentUser(),  
  created_at: new Date().toISOString(),  
  updated_at: new Date().toISOString(),  
  version: '1.0.0',  
  tags: []  
});

// Extract reusable functions instead of duplicating  
const formatDateString = (isoString: string): string => {  
  const date = new Date(isoString);  
  return new Intl.DateTimeFormat('en-US', {  
    dateStyle: 'medium',  
    timeStyle: 'short'  
  }).format(date);  
};

// Use everywhere dates need formatting, never inline the logic  
<Typography>{formatDateString(order.created_at)}</Typography>

---

