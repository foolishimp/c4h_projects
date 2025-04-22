**LLM Guide: C4H Editor Shell Service**

**1\. Purpose & Intent**

* The shell\_service is a backend API (built with FastAPI/Python) that acts as the central configuration provider for the C4H Editor frontend shell.  
* Its primary responsibilities are:  
  * Telling the frontend shell which "frames" or tabs to display and in what order.  
  * Defining which microfrontend applications (MFEs) are available and where to load them from (their URLs).  
  * Persisting user preferences regarding frame layout and assigned applications.  
  * Providing URLs for other necessary backend services.

**2\. Core Data Structures**

*(These represent the JSON structures exchanged with the frontend)*

* **AppDefinition**: Describes an available MFE.  
  * id: string (Unique identifier, e.g., "test-app", "config-selector-workorders")  
  * name: string (Display name, e.g., "Test App", "Work Order Config")  
  * type: string (How to load the MFE: "esm", "iframe", "wc")  
  * url: string (The URL to load the MFE from. Resolved by the service based on environment.)  
* **AppAssignment**: Defines which MFE is assigned to a slot within a frame.  
  * appId: string (References the id of an AppDefinition)  
  * config?: any (Optional MFE-specific configuration \- currently seems unused)  
* **Frame**: Represents a tab or view container in the shell.  
  * id: string (Unique identifier for the frame, e.g., "new-1745237898812")  
  * name: string (Display name for the tab, e.g., "Test Tab", "Work Orders")  
  * order: int (Determines the display order in the tab bar)  
  * assignedApps: List\[AppAssignment\] (List of MFEs assigned to this frame. Currently, seems to handle only one app per frame).  
* **ShellConfigurationResponse**: The main object returned on initial load.  
  * frames: List\[Frame\] (The user's current frame layout)  
  * availableApps: List\[AppDefinition\] (All MFEs known to the system)  
  * serviceEndpoints: Dict\[string, string\] (Map of service names to URLs, e.g., "mainBackendUrl": "http://...", "prefsServiceUrl": "http://...")

**3\. Key API Endpoints**

* **GET /api/v1/shell/configuration**  
  * **Purpose:** Fetches the initial state needed by the frontend shell to render the UI and load MFEs.  
  * **User Context:** Implicitly uses a default user (default\_user) for retrieving preferences.  
  * **Logic:**  
    1. Retrieves the default list of availableApps.  
    2. Retrieves the default frame structure.  
    3. Loads user-specific saved frames preference from the database (SQLite). If found, these override the default frames.  
    4. Resolves the url for each AppDefinition in availableApps. It prioritizes URLs defined in the environments.json file (matching the AppDefinition.id or config-selector-\* pattern) for the current runtime environment (e.g., "development"). Falls back to default URLs if not found in environments.json.  
    5. Retrieves other backend service endpoints (e.g., the URL for the main backend, its own preferences endpoint URL).  
    6. Returns the data structured as ShellConfigurationResponse.  
  * **Response Body:** ShellConfigurationResponse (JSON)  
  * **Frontend Usage:** Called once by the shell on startup (ShellConfigContext) to initialize the application layout and know where to load MFEs from.  
* **PUT /api/v1/shell/preferences**  
  * **Purpose:** Saves the user's current layout preferences (which tabs exist, their names, order, and assigned MFE).  
  * **User Context:** Saves preferences for the default user (default\_user).  
  * **Request Body:** List\[Frame\] (JSON) \- An array representing the complete desired state of the frames (tabs). The structure matches the Frame object described above.  
  * **Logic:**  
    1. Receives the list of Frame objects.  
    2. Validates the input.  
    3. Overwrites the existing preferences for the user in the database with this new layout.  
  * **Response Body:** { "message": "Preferences saved successfully" } or an error message (JSON).  
  * **Frontend Usage:** Called by the shell's "Preferences" dialog when the user clicks "Save". The frontend sends the current state of the configured frames as the request body.

**4\. Interaction with Frontend**

* The frontend shell relies on the GET /configuration endpoint during initialization to build its tab bar and determine which MFE URLs to use for dynamic imports.  
* The frontend shell uses the PUT /preferences endpoint when the user saves changes made in the preferences dialog (adding, removing, reordering tabs, or changing the assigned app).  
* The serviceEndpoints provided by GET /configuration are used by the shared apiService to configure base URLs for other backend interactions.

**5\. Development Environment Considerations**

* The shell\_service reads an environments.json file at startup.  
* The url fields within the development section of this file directly override the default URLs for MFEs, allowing developers to point the shell to local vite dev servers for each MFE using http://localhost:\<port\>/src/main.tsx style URLs.

---

This guide should provide sufficient context for an LLM (or a developer) working on the frontend to understand how the shell interacts with its configuration backend, what data structures to expect, and the purpose of the key API calls, without needing the backend implementation details.