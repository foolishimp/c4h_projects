**C4H Editor Frontend: Microfrontend Architecture \- Current State & Guide**

**Version:** 1.5 (Current Working Development State \- April 2025\)

**1\. Introduction & Goal**

* **Purpose:** Document the implemented microfrontend (MFE) architecture for the C4H Editor frontend, serving as a standard for ongoing development and LLM collaboration.  
* **Revised Goal:** Achieve application modularity, runtime isolation, clear communication patterns, and LLM compatibility using Vite, React, TypeScript, and pnpm workspaces. Prioritize **development simplicity and runtime stability** over complex runtime dependency sharing for local packages.  
* **Current Approach:** A shell orchestrates MFEs. Third-party libraries are shared via import maps. Local shared code (shared package) is **bundled** into each consuming application during development (via Vite aliasing) and build. Communication relies on a globally exposed event bus. Development runs entirely using Vite dev servers.

**2\. Core Technologies**

* **Build Tool:** Vite  
* **Framework:** React 18+  
* **Language:** TypeScript  
* **Package Manager:** pnpm (with workspaces)  
* **UI Library:** Material UI (MUI) v5  
* **Styling:** Emotion (via MUI)  
* **Communication:** Custom Event Bus (based on EventTarget)

**3\. Architecture Overview (Current Development State)**

* **Shell Application (packages/shell):**  
  * Runs via vite dev (npx vite).  
  * Renders the main UI frame and navigation.  
  * Fetches configuration (frames, app URLs, endpoints) from shell\_service.  
  * Dynamically imports MFEs using source URLs (http://\<mfe\_host\>:\<port\>/src/main.tsx).  
  * Renders \<script type="importmap"\> in index.html **only** for stable, third-party libraries (React, MUI, Emotion) loaded from external CDNs (esm.sh).  
  * Initializes and exposes the singleton eventBus instance on window.\_\_C4H\_EVENT\_BUS\_\_.  
  * Uses resolve.alias to handle its *own* imports from shared source code.  
* **Microfrontends (MFEs) (e.g., packages/test-app, config-selector):**  
  * Independent React/Vite applications, run via vite dev (npx vite) on distinct ports.  
  * Export a standard interface (e.g., mount function or default React component).  
  * Use resolve.alias to resolve imports from shared source code. Vite bundles the necessary shared code during its dev server processing.  
  * Declare third-party dependencies (React, MUI, Emotion) as external in their vite.config.ts (for build output), relying on the shell's import map at runtime.  
  * Access the shared event bus primarily via window.\_\_C4H\_EVENT\_BUS\_\_.  
* **Shared Package (packages/shared):**  
  * Contains shared types, utils, eventBus, apiService, common components.  
  * Built using tsc (via pnpm \--filter shared run build) mainly for ensuring type consistency across the workspace during development and build steps. Requires correctly configured package.json (main, types, exports pointing to build output) and tsconfig.json (declaration: true, rootDir, outDir).  
  * **Source code** (not built output) is consumed directly by shell and MFEs during development via Vite's resolve.alias.

**4\. Key Principles & Decisions (Current State)**

* **Dev Mode First:** Use vite dev for all packages for optimal DX. Resolve shared local code via aliases.  
* **Runtime Sharing Strategy:**  
  * **Third-Party:** Shared via Import Maps \+ CDN (esm.sh). Externalized in MFE builds.  
  * **Local (shared package):** **Bundled** into consumers. Not externalized, not in import map. *Rationale: Significantly simplifies development setup and avoids complex runtime resolution issues encountered previously.*  
* **Event Bus:** Singleton exposed via window.\_\_C4H\_EVENT\_BUS\_\_.  
* **Configuration:** Dynamic via shell\_service. Dev URLs point to MFE /src/main.tsx.  
* **Build/Install:** Use pnpm from workspace root (pnpm install, pnpm \--filter \<pkg\> run build).  
* **Startup:** Orchestrated via start\_frontends.py, reading environments.json, checking ports, launching dev servers using npx vite \--port \<num\>.

**5\. Configuration Standards (Current State)**

* **pnpm-workspace.yaml:** Defines packages: \['c4h-micro/packages/\*'\].  
* **package.json:** Use type: "module". Use workspace:\* for dependencies on shared. scripts.dev includes \--port \<num\> \--strictPort. shared package requires correct main, types, exports pointing to dist/build/.  
* **tsconfig.json:** Use "module": "ESNext", "moduleResolution": "bundler". MFEs/Shell use "noEmit": true. shared uses "declaration": true, "rootDir": "src", "outDir": "./dist/build".  
* **vite.config.ts:**  
  * Shell: No custom middleware for /shared. Includes optimizeDeps.include and resolve.dedupe for React/MUI/Emotion. Has resolve.alias for shared.  
  * MFEs: **No** 'shared' in build.rollupOptions.external. Include React/MUI/Emotion etc. in external. Include resolve.alias for shared. Use explicit build.rollupOptions.output.entryFileNames. No server/preview port settings.  
* **environments.json:** Defines URLs for dev (pointing to /src/main.tsx) and production (pointing to built assets). Ensure consistent port usage for packages serving multiple logical apps (like config-selector).

**6\. Development Workflow (Current State)**

1. **Setup:** Run pnpm install from the project root.  
2. **Run:** Execute python start\_frontends.py (and backends) from the root.  
3. **Access:** Open the shell URL (e.g., http://localhost:3100).

**7\. Outstanding Issues & TODOs**

* **Resolve MFE Runtime Errors:** Debug the remaining "Invalid hook call" / useContext errors in config-selector and potentially other MUI-using MFEs, likely related to ensuring React/MUI/Emotion singletons are correctly enforced by the Vite dedupe/optimizeDeps strategy.  
* **Fix test-app Event Counter:** Identify/implement the publisher for the test:message event that test-app listens for.  
* **Implement Iframe MFEs:** Add logic to shell/src/App.tsx to render \<iframe\> elements based on appDefinition.type \=== 'iframe'. (Ref: Req 2.2.3)  
* **Implement Event Bus Bridge:** Add postMessage listeners/dispatchers in the shell to allow communication between iframe MFEs and the window.\_\_C4H\_EVENT\_BUS\_\_. (Ref: Req 2.1.7, 2.4.7)  
* **Implement Web Component MFEs (Optional):** Add logic to shell/src/App.tsx to handle loading and mounting MFEs defined as Web Components (appDefinition.type \=== 'wc'). (Ref: Req 2.2.2)  
* **Define Production Build/Deployment:** Finalize the strategy. Likely involves:  
  * Running pnpm run build for all packages.  
  * Serving built MFE assets (with shared bundled) from static hosting/CDN.  
  * Serving the shell app.  
  * Updating environments.json production URLs.  
  * Ensuring the production import map correctly loads shared 3rd party libs.  
* **Refine shared Build:** Ensure tsc build for shared is robust and consistently produces correct .js and .d.ts output in dist/build, even if only used for type-checking in dev.  
* **(Optional) Add PUT Mock Endpoint:** Implement mock for PUT /api/v1/shell/preferences in a shell Vite plugin for fully offline development. (Ref: Req 2.1.5)  
* **(Optional) TypeScript Hardening:** Perform a pass to remove any types and enforce stricter rules where appropriate, aligning with Typescript\_Design\_Principles.md.

---

This document reflects the current state and the path chosen to achieve stability. The next immediate step is resolving the remaining runtime hook errors in the complex MFEs.