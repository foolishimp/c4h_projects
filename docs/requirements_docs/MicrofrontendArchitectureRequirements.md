# **Microfrontend Architecture Requirements: C4H Editor (ESM-Based)**

Version: 1.0  
Date: 2025-04-19  
**1\. Introduction**

1.1. Purpose: This document defines the requirements for a revised microfrontend (MFE) architecture for the C4H Editor application. This architecture prioritizes simplicity, maintainability, standard browser features (ES Modules, Dynamic Import, Import Maps), and robust inter-application communication to support both internally developed and third-party components. It aims to replace previous architectures based on Module Federation or SystemJS/Single-SPA, addressing complexities encountered with those approaches.  
1.2. Goals:  
\* Simplicity: Reduce build configuration and runtime complexity compared to previous MFE approaches.  
\* Isolation: Ensure clear separation between the shell and MFEs, and between MFEs, minimizing unintended side effects.  
\* Maintainability: Promote a well-structured codebase where MFEs can be updated or replaced with minimal impact on other parts of the system.  
\* Independent Deployment: Enable separate, independent build and deployment pipelines for the shell and each MFE.  
\* Heterogeneous MFE Support: Natively support loading MFEs developed as standard ESM bundles, optionally wrapped as Web Components, and third-party applications loaded within Iframes.  
\* Robust Communication: Provide a reliable and clearly defined event bus mechanism for communication between the shell, internal MFEs, and sandboxed (iframe) MFEs.  
\* LLM-Friendly: Structure MFEs as relatively self-contained units that are potentially easier to analyze or interact with programmatically.  
1.3. Non-Goals:  
\* Cross-MFE boundary Hot Module Replacement (HMR) during development. A full page refresh might be required after an MFE is rebuilt.  
\* Support for browsers lacking native support for ES Modules, dynamic import(), or \<script type="importmap"\> without appropriate polyfills. The target environment is modern evergreen browsers.  
\* Complex, deeply synchronized state sharing between MFEs. Communication and state updates should primarily be event-driven.  
**2\. Functional Requirements**

2.1. Shell Application:  
\* 2.1.1. The shell shall be the main container application, built using React and Vite.  
\* 2.1.2. The shell shall fetch its initial layout and application configuration (frames, available apps list with loading URLs, service endpoints) from a designated backend configuration service (e.g., /api/v1/shell/configuration) on startup.  
\* 2.1.3. The shell shall render a primary navigation interface (e.g., vertical tabs) corresponding to the "frames" defined in the fetched configuration.  
\* 2.1.4. The shell shall dynamically load the MFE(s) assigned to a frame when that frame becomes active. Loading mechanisms will differ based on MFE type (see Section 4).  
\* 2.1.5. The shell shall provide UI for users to manage frame configurations (add, remove, reorder frames; assign applications to frames). User preferences shall be persisted via the configuration service (PUT /api/v1/shell/preferences).  
\* 2.1.6. The shell shall initialize and provide access to the central Event Bus instance.  
\* 2.1.7. The shell shall act as a message broker for MFEs hosted in Iframes, relaying messages between the Iframe (postMessage) and the central Event Bus.  
2.2. Microfrontends (MFEs):  
\* 2.2.1. ESM MFEs: The primary MFE type shall be standard ES Modules built with Vite (build.format: 'esm'). They export necessary functions or framework components (e.g., React).  
\* 2.2.2. Web Component MFEs (Optional): The architecture shall allow for MFEs packaged as standard Web Components (Custom Elements) for enhanced isolation. These will also be loaded via an ESM entry point.  
\* 2.2.3. Iframe MFEs: The architecture shall support loading third-party or legacy applications within sandboxed \<iframe\> elements. The src URL for the iframe will be provided by the configuration service.  
\* 2.2.4. Independence: Each MFE shall be independently buildable, testable (in isolation), and deployable. Dependencies on other MFEs are disallowed; communication must occur via the Event Bus.  
\* 2.2.5. Configuration: MFEs shall receive environment-specific configuration (e.g., API endpoints) from the shell, either through direct injection (e.g., window.\_\_SHELL\_CONFIG\_\_), framework context (if applicable), or dedicated configuration events on the Event Bus.  
\* 2.2.6. API Access: MFEs shall primarily use the shared apiService instance (provided via a shared module) for backend communication. This instance will be pre-configured by the shell with the correct base URL.  
\* 2.2.7. Communication: All intentional cross-MFE or MFE-shell communication shall occur via the central Event Bus (except for Iframes, which use postMessage bridged by the shell).  
\* 2.2.8. Supported MFEs: The architecture must support the existing C4H Editor MFEs (config-selector, job-management, yaml-editor) refactored to ESM, plus a third-party chat-client (loaded via Iframe).  
2.3. Configuration Management:  
\* 2.3.1. The backend configuration service (shell\_service) provides the single source of truth for runtime configuration via the /api/v1/shell/configuration endpoint.  
\* 2.3.2. The configuration response must include:  
\* List of user-defined frames with their assigned app IDs.  
\* List of all available applications, including their unique ID, name, type (ESM, WC, Iframe), and the environment-specific URL required to load them (e.g., ESM bundle URL, Iframe src URL).  
\* Necessary backend service endpoint URLs (e.g., main backend API, preferences service API if different).  
\* 2.3.3. The configuration service shall derive the runtime configuration by merging base defaults (from its database/defaults.json) with environment-specific overrides (from environments.json).  
2.4. Event Bus / Inter-App Communication:  
\* 2.4.1. Mechanism: A singleton instance of an event bus, based on the browser's EventTarget API or a similar robust pub/sub implementation, shall be used. It will be exported from the shared package.  
\* 2.4.2. Accessibility: The event bus instance must be easily accessible by the shell and all non-iframe MFEs.  
\* 2.4.3. Message Structure: Events dispatched shall use the CustomEvent interface. The detail property shall contain an object with a defined structure:  
typescript interface EventDetail { source: string; // Unique identifier of the sender (e.g., 'shell', 'config-editor', 'chat-client') payload: any; // Data specific to the event type } // Example Dispatch: // eventBus.dispatchEvent(new CustomEvent('config:saved', { // detail: { source: 'config-editor', payload: { configId: 'id123', configType: 'team' } } // }));  
\* 2.4.4. Event Types: A documented registry of standard event types (e.g., config:loaded, config:saved, chat:provide\_context, notification:show, navigation:request) and their expected detail.payload structure shall be maintained in the shared package.  
\* 2.4.5. Subscription: Components shall subscribe using eventBus.addEventListener(eventType, callback). Callbacks must handle potential errors internally. Subscriptions must be cleaned up (using removeEventListener) when the subscribing component unmounts.  
\* 2.4.6. Error Handling: The event bus implementation itself should log errors if a subscriber callback fails but should not prevent other subscribers from receiving the event. Subscribers are responsible for their own error handling.  
\* 2.4.7. Iframe Communication Bridge:  
\* Iframe \-\> Bus: The shell shall listen for message events on the window. When a message is received from a trusted/configured iframe origin, the shell shall validate the message format and dispatch a corresponding CustomEvent onto the internal event bus. The source in the event detail should identify the iframe MFE (e.g., chat-client).  
\* Bus \-\> Iframe: The shell shall subscribe to specific event types on the internal event bus that are designated for iframe targets. When such an event is received, the shell shall format a message according to the agreed-upon postMessage protocol and send it to the target iframe's contentWindow using postMessage(message, targetOrigin). The targetOrigin must be configured securely.  
\* Protocol: A clear protocol for postMessage data (e.g., { type: string, payload: any, source: string }) needs to be defined and agreed upon between the shell and the iframe application.  
**3\. Non-Functional Requirements**

3.1. Performance: Optimize for fast initial load of the shell. Dynamic MFE loading should feel responsive. Minimize bundle sizes for MFEs. Shared libraries (React, MUI) loaded once via import map.  
3.2. Scalability: Adding new MFEs should not require modifications to other unrelated MFEs.  
3.3. Maintainability: Utilize TypeScript for strong typing. Enforce linting and formatting standards. Code should be modular and well-documented.  
3.4. Testability: MFEs must be unit-testable and integration-testable in isolation. End-to-end tests should cover shell-MFE interactions via the event bus.  
3.5. Security: Iframes must run with appropriate sandbox attributes. postMessage communication must validate message origin and data structure.  
**4\. Technical Architecture Overview**

4.1. Shell: React/Vite. Renders main layout. Fetches config. Manages import map. Uses dynamic import() to load MFE entry points. Hosts Event Bus. Bridges Iframe communication.  
4.2. MFEs (ESM): Vite builds (format: 'esm'). Export React components or lifecycle functions. Externalize shared dependencies (React, MUI, shared utils) expected to be in the import map.  
4.3. MFEs (Web Component): Optional. Standard Custom Element definition within an ESM bundle. Loaded via dynamic import().  
4.4. MFEs (Iframe): Standard \<iframe\> element. src provided by config service. Communicates via postMessage.  
4.5. Module Loading: Shell generates/fetches \<script type="importmap"\> defining shared dependencies and potentially MFE entry points (though dynamic import(URL) is preferred for MFEs).  
4.6. Configuration Service: Existing shell\_service (FastAPI/Python) adapted to provide configuration matching the ShellConfigurationResponse structure, including environment-specific MFE URLs.  
4.7. Event Bus: Singleton class instance based on EventTarget, exported from the shared package.  
4.8. Shared Package: Workspace package containing TypeScript types, the Event Bus implementation, the configured apiService, and potentially shared React components/hooks/utils.  
**5\. Deployment**

5.1. Monorepo: Project managed using pnpm workspaces (or Yarn).  
5.2. Build Pipelines: Separate CI/CD pipeline for the shell and each MFE.  
5.3. Artifacts: Pipelines produce versioned, static JS/CSS assets for each MFE and the shell.  
5.4. Hosting: Assets hosted on a CDN or static file server.  
5.5. Activation: Deploying a new MFE version involves uploading its assets and updating the URL returned by the configuration service for the relevant environments. The shell will load the new version on its next configuration fetch.