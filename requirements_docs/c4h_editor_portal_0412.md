Design Document: C4H Editor - Dynamic Portal Shell Architecture
Version: 1.0
Date: 2025-04-12

1. Introduction & Context
This document outlines a revised architecture for the C4H Editor frontend shell (c4h-micro). The initial version featured a static sidebar navigation loading predefined microfrontend applications (like Configuration Management and Job Management). Through iterative refactoring aimed at improving the job submission process, several issues arose:

API Mismatches: Initial refactoring workorders (e.g., WO 70002, 70003, 70004) led to temporary inconsistencies between the API contracts expected by the Frontend, the Editor Backend, and the underlying C4H Service regarding job submission payloads (raw list vs. dictionary wrapper for configurations).

Debugging Complexity: Troubleshooting errors required analyzing logs across multiple services (Frontend Console, Editor Backend, C4H Service) to pinpoint the source of mismatches (e.g., 422 Unprocessable Entity errors due to Pydantic validation failures, TypeError due to incorrect JSON serialization).

Inflexible UI: The original shell design with a fixed sidebar navigation lacked flexibility. Adding new tools/views or allowing user customization required significant code changes to the shell application itself.

This revised design addresses these points by introducing a more flexible, user-configurable portal architecture and clarifying service responsibilities, drawing upon the better API design patterns identified during the debugging process.

2. Goals
User Customization: Allow users to define their own workspace layout ("Frames" or tabs) within the shell.

Dynamic App Integration: Enable users to assign available microfrontend applications ("Apps") to their defined Frames.

Decoupling: Separate UI concerns (layout, preferences, service discovery) from backend domain logic (job/config management, C4H workflow execution).

Extensibility & Maintainability: Create a shell architecture that can easily accommodate new microfrontend apps and backend services without requiring modifications to the core shell code. Simplify the process of managing endpoints and UI structure through configuration.

Resolve API Inconsistencies: Formally adopt the {"configs": [...]} dictionary wrapper structure for job submissions to the C4H Service API, aligning the Editor Backend client accordingly.

Reduce Technical Debt: Eliminate legacy/unused code paths related to older job submission methods.

3. Proposed Architecture
The proposed architecture consists of the following key components:

Frontend Shell (c4h-micro/packages/shell):

Acts as the main container application.

Responsible for fetching UI configuration from the Preferences Shell Service on startup.

Dynamically renders a tab bar based on user-defined "Frames".

Dynamically loads and displays the microfrontend "App(s)" assigned to the currently active Frame using Module Federation (RemoteComponent loader).

Provides the UI for managing preferences (via a dialog/panel) which interacts with the Preferences Shell Service API.

Contains the shared apiService which is dynamically configured with the endpoint for the Job/Config Service.

Preferences Shell Service (New Backend Service):

A dedicated backend service acting as a Backend-for-Frontend (BFF) for the Shell UI.

Responsibilities:

Persistently store and manage user-specific UI preferences (Frames, App assignments, Frame order) in its own database.

Store and serve the list of "Available Apps" (microfrontends the shell can load, including their name, scope, module, and URL/path).

Store and serve endpoint URLs and potentially other connection details for other backend services (like the Job/Config Service).

API: Exposes endpoints for the Shell to fetch its configuration and save user preferences.

Job/Config Service (Refactored c4h_editor/backend):

Responsibilities:

Core domain logic for managing configurations (Workorders, Team Configs, Runtime Configs) via the Git repositories (CRUD operations).

Listing available configurations for the UI.

Handling job submissions: Receiving requests from the Shell (via its dynamic endpoint), preparing the list of configuration content, and calling the C4H Service using the correct {"configs": [...]} payload structure via its client (c4h_service.py).

Providing job status updates by querying the C4H Service.

API: Exposes endpoints for config CRUD, job submission (/api/v1/jobs/multi-config), and job status (/api/v1/jobs/{id}). It does not handle UI layout or shell preferences.

C4H Service (c4h_services / c4h_ai_dev):

Responsibilities: Executes the core code refactoring workflows. Accepts job requests containing the list of configuration content dictionaries ({"configs": [...]}), performs internal merging, and orchestrates the agent teams.

API: Exposes POST /api/v1/jobs (expecting MultiConfigJobRequest model) and GET /api/v1/jobs/{job_id} for status/results.

Microfrontend Apps (e.g., config-selector, job-management, yaml-editor):

Self-contained UI components representing specific tools/features.

Exposed via Module Federation for dynamic loading by the Shell.

Interact with the Job/Config Service via the shared apiService (which gets its endpoint from the Preferences Service via the Shell).

Interaction Flow (Example: Job Submission):

Shell starts, calls Preferences Service (GET /shell/configuration) to get UI layout, available apps, and the Job/Config Service URL.

Shell configures its apiService with the Job/Config Service URL.

User navigates to a "Job Submission" Frame (tab).

Shell dynamically loads the job-management microfrontend app into the content area.

User fills the JobCreator form (potentially loading defaults from a saved frame preference).

JobCreator calls submitJobConfigurations context function.

Context function calls apiService.submitJobConfigs.

apiService sends POST /api/v1/jobs/multi-config request (with {"configurations": [...]}) to the Job/Config Service URL it received during startup.

Job/Config Service endpoint receives the request, validates it using JobListRequest, starts the submit_multi_configs background task, passing the list of references.

Background task loads full config objects, extracts/serializes the content from each, creates config_list_for_service.

Background task calls the internal C4H Service client (c4h_service.py).

Internal client wraps the list: payload = {"configs": config_list_for_service}.

Internal client sends POST /api/v1/jobs with the payload to the C4H Service API URL.

C4H Service validates the payload against MultiConfigJobRequest, performs merge, starts workflow.

Responses flow back up the chain.

4. Key Features / Concepts
Frames: User-defined tabs providing customizable workspaces within the shell. Managed via the Preferences UI and stored by the Preferences Service.

App Assignment: Users associate specific microfrontend applications (defined in the Preferences Service) with each Frame. This determines what content is loaded when a Frame tab is selected.

Preferences Management: A dedicated UI (e.g., dialog accessed via gear icon) allows users to perform CRUD operations on Frames and manage App assignments. Changes are persisted via the Preferences Shell Service.

Dynamic App Loading: The Shell uses the frame configuration (fetched from the Preferences Service) to dynamically determine which microfrontend(s) to load for the active tab using Module Federation (e.g., via the RemoteComponent loader).

Service Discovery: The Shell learns the location (endpoint URL) of the Job/Config Service (and potentially others) from the Preferences Shell Service, decoupling the frontend from backend deployment details.

5. API Definitions (High-Level)
Preferences Shell Service:

GET /api/v1/shell/configuration: Returns { frames: Frame[], availableApps: AppDefinition[], serviceEndpoints: { jobConfigServiceUrl: string, ... } }.

PUT /api/v1/shell/preferences: Accepts { frames: Frame[], ... } to save UI layout.

GET /api/v1/shell/available-apps: (Optional) Returns AppDefinition[].

Job/Config Service:

GET /api/v1/configs/{type}

GET /api/v1/configs/{type}/{id}

POST /api/v1/configs/{type}

PUT /api/v1/configs/{type}/{id}

DELETE /api/v1/configs/{type}/{id}

POST /api/v1/jobs/multi-config: Accepts JobListRequest ({"configurations": [...]}).

GET /api/v1/jobs/{job_id}: Returns JobStatus.

C4H Service:

POST /api/v1/jobs: Accepts MultiConfigJobRequest ({"configs": [...]}).

GET /api/v1/jobs/{job_id}: Returns JobStatusResponse.

6. Data Models (Conceptual)
Frame: { id: string, name: string, order: number, assignedApps: AppAssignment[] }

AppAssignment: { appId: string, layoutInfo?: any } (LayoutInfo needed if multiple apps per frame)

AppDefinition: { id: string, name: string, scope: string, module: string, url?: string } (Info needed to load via Module Federation)

ShellConfigurationResponse: Contains frames, availableApps, serviceEndpoints.

JobListRequest (Editor Backend Input): { configurations: JobConfigReference[], user_id?: string, job_configuration?: object }

MultiConfigJobRequest (C4H Service Input): { configs: object[] }

7. Implementation Considerations
Storage: The Preferences Shell Service requires a database (e.g., PostgreSQL, MongoDB) to store user preferences reliably.

App Discovery: A mechanism is needed to populate the availableApps list served by the Preferences Service. Options:

Manual configuration file read by the Preferences Service.

Service registration/discovery pattern if the architecture grows more complex.

Frontend State Management: A robust solution (React Context or a dedicated state library like Zustand/Redux) is needed in the Shell to manage the fetched configuration, active frame, etc.

Authentication: How are preferences secured per user? This service adds a natural point to handle user sessions.

Initial Defaults: Define a default set of Frames and App assignments for new users or when preferences fail to load.

8. Future Enhancements
Allow multiple Apps per Frame with user-configurable layouts (split panes, grids).

Role-based access control for specific Frames or Apps.

Sharing/importing/exporting Frame configurations.