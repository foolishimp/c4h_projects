# Vite Module Federation: Complete Setup Guide

## Core Principles

1. **Path Consistency**: Vite places federation files in `dist/assets/` by default
2. **Build + Preview Workflow**: Build remote apps → serve in preview mode → run shell in dev mode
3. **Standard Import Pattern**: Use React.lazy with direct imports
4. **Type Safety**: Declare module types in shell app
5. **Port Management**: Each microfrontend needs its own dedicated port

## Project Structure

```
project-root/
├── packages/
│   ├── shared/              # Shared types, utilities, config
│   ├── shell/               # Host application
│   └── microfrontend-name/  # Remote microfrontend
└── package.json             # Workspace configuration
```

## Configuration Files

### 1. Shell Configuration (vite.config.ts)

```typescript
// packages/shell/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';
import path from 'path';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'shell',
      remotes: {
        // IMPORTANT: Point to /assets/remoteEntry.js
        microappName: 'http://localhost:3001/assets/remoteEntry.js'
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' }
      }
    })
  ],
  build: {
    modulePreload: false,  // Critical for Module Federation
    target: 'esnext',      // Required for proper federation
    minify: false,         // Helpful during development
    cssCodeSplit: false    // Prevents CSS issues
  },
  server: {
    port: 3000,
    strictPort: true,
    cors: true             // Required for federation
  }
});
```

### 2. Remote Microfrontend Configuration (vite.config.ts)

```typescript
// packages/microfrontend-name/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';
import path from 'path';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'microappName',  // Must match the shell's remote key
      filename: 'remoteEntry.js',
      exposes: {
        // CRITICAL: Use './ComponentName' format for the key
        './ComponentName': './src/ComponentName.tsx'
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' }
      }
    })
  ],
  build: {
    modulePreload: false,
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  },
  server: {
    port: 3001,  // Unique port per microfrontend
    strictPort: true,
    cors: true
  },
  preview: {
    port: 3001,
    strictPort: true,
    cors: true
  }
});
```

### 3. Type Declarations in Shell

```typescript
// packages/shell/src/types.d.ts
declare module 'microappName/ComponentName' {
  import React from 'react';
  const Component: React.FC<any>; // Add proper props type if available
  export default Component;
}
```

### 4. Importing Remote Components in Shell

```tsx
// packages/shell/src/App.tsx
import React, { lazy, Suspense } from 'react';

// IMPORTANT: Format must match 'remoteName/ExposedComponent'
const RemoteComponent = lazy(() => import('microappName/ComponentName'));

function App() {
  return (
    <div>
      <h1>Shell Application</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <RemoteComponent />
      </Suspense>
    </div>
  );
}

export default App;
```

## Deployment & Execution Workflow

1. **Build Process**:
   ```bash
   # Build shared package first
   cd packages/shared && npm run build
   
   # Build all remote microfrontends
   cd packages/microfrontend-name && npm run build
   
   # Optionally build shell last
   cd packages/shell && npm run build
   ```

2. **Development Mode**:
   ```bash
   # Start remote microfrontends in preview mode
   cd packages/microfrontend-name && npm run preview
   
   # Start shell in development mode
   cd packages/shell && npm run dev
   ```

## Validation Checklist

Before integration:
1. ✅ Verify remoteEntry.js exists in expected locations:
   - Check `dist/assets/remoteEntry.js` in the microfrontend's build output
   - Access directly via browser: `http://localhost:3001/assets/remoteEntry.js`

2. ✅ Ensure component exports properly:
   - Component has default export: `export default ComponentName;`
   - Export path in vite.config.ts is correct: `'./ComponentName': './src/ComponentName.tsx'`

3. ✅ Check port configurations:
   - Each microfrontend has unique, dedicated port
   - Shell URLs match those ports exactly

## Troubleshooting Common Issues

| Issue | Solution |
|-------|----------|
| Module not found | Ensure module path in import matches exposed path exactly |
| Loading HTML instead of JS | Use `/assets/remoteEntry.js` path, not just `/remoteEntry.js` |
| CORS errors | Enable CORS in all vite.config.ts files; use preview mode for remotes |
| Shared dependencies conflict | Make sure all `shared` configurations match across apps |
| Hot reload issues | Use build+preview for remotes instead of dev mode |

## Testing New Microfrontends

1. Start by testing the microfrontend standalone
2. Verify the remoteEntry.js is accessible 
3. Use this shell component to test the integration:

```tsx
import React, { useState, lazy, Suspense } from 'react';

// Update with your module path
const RemoteComponent = lazy(() => import('microappName/ComponentName'));

export function TestLoader() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  return (
    <div>
      <h2>Remote Component Test</h2>
      <button 
        onClick={() => {
          setIsLoading(true);
          setError(null);
        }}
      >
        Load Remote Component
      </button>
      
      {isLoading && (
        <Suspense fallback={<div>Loading...</div>}>
          <ErrorBoundary onError={setError}>
            <RemoteComponent />
          </ErrorBoundary>
        </Suspense>
      )}
      
      {error && (
        <div style={{ color: 'red' }}>
          <h3>Error:</h3>
          <pre>{error.message}</pre>
        </div>
      )}
    </div>
  );
}

// Simple error boundary component
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error) {
    this.props.onError(error);
  }

  render() {
    if (this.state.hasError) {
      return null;
    }
    return this.props.children;
  }
}
```

## Final Checklist

- [ ] Each microfrontend exposes components with `./` prefix
- [ ] Shell imports components with `remoteName/ComponentName` pattern
- [ ] Type declarations match exposed component names
- [ ] Build settings configure `modulePreload: false` and `target: 'esnext'`
- [ ] Remote URLs include `/assets/` path for preview/production mode
- [ ] Ports are unique and match in both shell and microfrontend configs