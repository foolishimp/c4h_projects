**LLM Guide: C4H Editor Main Backend Service (backend)**

**1\. Purpose & Intent**

* The backend service is the primary API for managing core application data within the C4H Editor, specifically focusing on **Configurations** (like Work Orders, Teams, Runtimes) and background **Jobs**.  
* It handles Create, Read, Update, Delete (CRUD) operations for these configurations.  
* It manages the lifecycle of background jobs, likely related to processing or utilizing the configurations.  
* It acts as an intermediary, communicating with an external **"C4H Service"** for certain operations (details likely specified in design documents).  
* It provides the necessary data persistence layer for configurations and jobs.

**2\. Technology Stack**

* Assumed: FastAPI (Python framework based on start\_backends.py using uvicorn).  
* Data Persistence: Likely a relational database (specifics depend on implementation, could be PostgreSQL, SQLite, etc.).

**3\. Key Responsibilities**

* CRUD operations for different configuration types (Work Orders, Teams, Runtimes, etc.).  
* Submitting, monitoring, and managing background jobs.  
* Validating configuration data.  
* Interfacing with the external C4H Service.  
* Serving data required by the config-selector-\* MFEs and the job-management MFE.

**4\. Core Data Structures (Anticipated)**

*(These are based on typical REST API patterns and likely detailed in c4h\_jobs\_config\_dmodel.md and other design guides. Exact field names might vary.)*

* **ConfigBase (Conceptual)**: Common fields for all config types.  
  * id: string (UUID or unique identifier)  
  * name: string  
  * description?: string  
  * config\_type: string (e.g., "workorder", "team", "runtime")  
  * created\_at: string (ISO 8601 timestamp)  
  * updated\_at: string (ISO 8601 timestamp)  
  * data: object (The actual configuration payload specific to the type)  
* **WorkOrderConfig**: Extends ConfigBase, data contains work order specific fields.  
* **TeamConfig**: Extends ConfigBase, data contains team specific fields.  
* **RuntimeConfig**: Extends ConfigBase, data contains runtime specific fields.  
* **Job**: Represents a background job.  
  * id: string (UUID or unique identifier)  
  * config\_id: string (ID of the config used by the job)  
  * config\_type: string  
  * status: string (e.g., "pending", "running", "completed", "failed", "cancelled")  
  * progress?: number (0-100)  
  * result?: any (Job output/result upon completion)  
  * error\_message?: string (If status is "failed")  
  * created\_at: string (ISO 8601 timestamp)  
  * updated\_at: string (ISO 8601 timestamp)  
* **JobListResponse**: For listing jobs.  
  * jobs: List\[Job\]  
  * total: int  
  * page?: int  
  * size?: int

**5\. Key API Endpoints (Inferred from Frontend Usage)**

*(Base Path: /api/v1)*

* **Configurations:**  
  * GET /configs/{config\_type}  
    * **Purpose:** List all configurations of a specific type (e.g., /configs/workorder).  
    * **Response:** List\[ConfigBase\] (or specific type list like List\[WorkOrderConfig\]).  
  * POST /configs/{config\_type}  
    * **Purpose:** Create a new configuration of the specified type.  
    * **Request Body:** Object containing fields for the new config (e.g., name, description, data).  
    * **Response:** The newly created ConfigBase (or specific type) object, including its assigned id.  
  * GET /configs/{config\_type}/{config\_id}  
    * **Purpose:** Retrieve the details of a single configuration by its type and ID.  
    * **Response:** The specific ConfigBase (or derived type) object.  
  * PUT /configs/{config\_type}/{config\_id}  
    * **Purpose:** Update an existing configuration.  
    * **Request Body:** Object containing the fields to update.  
    * **Response:** The updated ConfigBase (or derived type) object.  
  * DELETE /configs/{config\_type}/{config\_id}  
    * **Purpose:** Delete a specific configuration.  
    * **Response:** Success/failure message (e.g., { "message": "Config deleted" }).  
* **Jobs:**  
  * GET /jobs/  
    * **Purpose:** List background jobs. May support query parameters for filtering (e.g., by status, config\_type, pagination).  
    * **Response:** JobListResponse.  
  * POST /jobs/  
    * **Purpose:** Create/submit a new background job, likely referencing one or more configuration IDs.  
    * **Request Body:** Object specifying the job type, target configuration(s), and any parameters.  
    * **Response:** The newly created Job object, likely with status "pending" or "submitted".  
  * GET /jobs/{job\_id}  
    * **Purpose:** Get the current status and details of a specific job.  
    * **Response:** The Job object.  
  * DELETE /jobs/{job\_id}  
    * **Purpose:** Request cancellation or deletion of a job (depending on implementation).  
    * **Response:** Success/failure message or updated Job object.

**6\. Interaction with Frontend**

* The config-selector-\* MFEs use the /configs/\* endpoints extensively to list, view, create, edit, and delete configurations relevant to their type.  
* The job-management MFE uses the /jobs/\* endpoints to list jobs, view their status, submit new jobs (likely after selecting configurations), and potentially cancel jobs.  
* All interactions from the frontend go through the shared apiService, which should be configured by the ShellConfigContext with the base URL for this backend service (e.g., http://localhost:8011).

**7\. External C4H Service Interaction**

* This backend service communicates with another, presumably more authoritative or specialized "C4H Service".  
* The exact nature of this interaction (e.g., fetching data, validating configs, submitting final job requests) should be detailed in the specific design documents (WorkOrder\_Design\_Guide.md, Jobs\_Design\_Guide\_0402.md, c4h\_api\_doc\_v1.md). The frontend generally doesn't need to know the specifics, only that this backend handles that communication.

---

This guide provides the necessary context about the backend's role, data, and API surface for an LLM or developer working primarily on the frontend MFEs that interact with it. For exact request/response schemas and behaviour details, the referenced design documents should be consulted.