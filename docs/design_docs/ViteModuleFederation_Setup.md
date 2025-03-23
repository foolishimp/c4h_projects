# Vite Module Federation Setup - Microfrontends Summary

## Root Cause
The primary issue with Vite and Module Federation is that the `remoteEntry.js` file (crucial for Module Federation to work) is placed in the `assets/` subdirectory during build, but the shell application typically looks for it at the root path. This path mismatch causes the "Failed to fetch dynamically imported module" error.

## Key Principles for Success

1. **Correct Remote URL Path**:
   - The shell application must reference the remote module using the actual build location: `/assets/remoteEntry.js` not just `/remoteEntry.js`
   - Example: `http://localhost:3001/assets/remoteEntry.js` instead of `http://localhost:3001/remoteEntry.js`

2. **Build + Preview Workflow**:
   - The most reliable workflow is: build the remote app → serve it in preview mode → run the shell in dev mode
   - Development mode server behavior can interfere with properly serving the Module Federation files

3. **Vite Configuration Best Practices**:
   - Set `modulePreload: false` in build options
   - Set `target: 'esnext'` for compatibility
   - Enable CORS in server configuration

4. **Directory Structure Knowledge**:
   - Understand that Vite places the federation files in `dist/assets/` by default
   - Either point to this location or customize the output path

## Implementation Steps

1. **Configure the remote app** (the microfrontend being consumed):
   ```typescript
   // remote-app/vite.config.ts
   export default defineConfig({
     plugins: [
       react(),
       federation({
         name: 'remoteApp',
         filename: 'remoteEntry.js',
         exposes: {
           './Component': './src/Component.tsx',
         },
         shared: ['react', 'react-dom']
       })
     ],
     build: {
       modulePreload: false,
       target: 'esnext',
       minify: false,
       cssCodeSplit: false
     }
   })
   ```

2. **Configure the shell app** (the app consuming the microfrontend):
   ```typescript
   // shell/vite.config.ts
   const remoteUrl = 'http://localhost:3001/assets/remoteEntry.js'; // Note the /assets/ path

   export default defineConfig({
     plugins: [
       react(),
       federation({
         name: 'shell',
         remotes: {
           remoteApp: remoteUrl
         },
         shared: ['react', 'react-dom']
       })
     ],
     build: {
       modulePreload: false,
       target: 'esnext'
     },
     server: {
       cors: true
     }
   })
   ```

3. **Workflow Scripts**:
   ```json
   "scripts": {
     "dev:federation": "npm run build:remote && npm run preview:remote & npm run dev:shell",
     "build:remote": "cd remote-app && npm run build",
     "preview:remote": "cd remote-app && npm run preview",
     "dev:shell": "cd shell && npm run dev"
   }
   ```

4. **Type Declarations**:
   ```typescript
   // shell/types.d.ts
   declare module 'remoteApp/Component' {
     import React from 'react';
     const Component: React.FC;
     export default Component;
   }
   ```

By following these principles, you can successfully implement Module Federation with Vite microfrontends without facing the common remoteEntry.js loading issues.