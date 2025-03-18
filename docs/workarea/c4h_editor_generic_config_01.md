# Refactoring Plan for C4H Editor with New Microfrontends

This document provides a detailed plan to refactor the C4H Editor into a modular, configuration-driven architecture. The plan introduces two new microfrontends: **ConfigSelector** for managing configurations (acting as an asset creator and manager) and **JobManagement** for handling job creation, submission, and monitoring. The existing **YamlEditor** is preserved as a specialized microfrontend for YAML-based configuration editing. The shell application integrates these microfrontends, providing navigation and layout. This design ensures modularity, scalability, and flexibility for future enhancements.

---

## 1. Evaluation: Current State vs. Future Domain Model

### Current State

- **ConfigEditor Microfrontend:**  
  - Manages WorkOrder configurations with an embedded YAML editor and version control.

- **Shell Application:**  
  - Provides navigation, job listing, and work order listing, integrating the ConfigEditor.

- **Shared Package:**  
  - Supplies types, utilities, and reusable components.

- **Job Submission:**  
  - Limited to a single WorkOrder configuration, not supporting multiple config types.

- **YAML Editor:**  
  - Currently embedded in ConfigEditor but will be treated as an existing, separate microfrontend.

### Future Domain Model Requirements

- **Three Config Types:**  
  A job consists of a tuple of WorkOrder, TeamConfig, and RuntimeConfig.

- **Flexibility:**  
  Support for adding new config types without code changes, driven by configuration metadata.

- **Separation of Concerns:**
  - **ConfigSelector Microfrontend:**  
    Manages CRUD operations for all config types (asset creator and manager).
  - **JobManagement Microfrontend:**  
    Handles job creation, submission, and monitoring.
  - **YamlEditor Microfrontend:**  
    Provides a specialized YAML editor for configurations (specialized asset editor).
  - **Shell Integration:**  
    Offers navigation and layout, seamlessly integrating the microfrontends.

---

## 2. Refactoring Plan

### Step 1: Establish YamlEditor Microfrontend

**Purpose:**  
A reusable, specialized YAML editor for editing configurations.

**Implementation:**

- **Create a new package:**  
  `packages/yaml-editor`

- **Extract the YAML editor (e.g., YAMLEditor.tsx) from ConfigEditor (if not already separate) into this package.**

- **Expose the YamlEditor component via Module Federation, accepting props for initial YAML content and save callbacks.**

#### Vite Configuration

```typescript
// packages/yaml-editor/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'yamlEditor',
      filename: 'remoteEntry.js',
      exposes: {
        './YamlEditor': './src/YamlEditor.tsx',
      },
      shared: ['react', 'react-dom', 'monaco-editor'],
    }),
  ],
  build: {
    target: 'esnext',
  },
});
```

#### YamlEditor Component

```tsx
// packages/yaml-editor/src/YamlEditor.tsx
import React, { useState } from 'react';
import Editor from '@monaco-editor/react';

interface YamlEditorProps {
  initialYaml: string;
  onSave: (yaml: string) => void;
}

const YamlEditor: React.FC<YamlEditorProps> = ({ initialYaml, onSave }) => {
  const [yaml, setYaml] = useState(initialYaml);

  const handleEditorChange = (value: string | undefined) => setYaml(value || '');
  const handleSaveClick = () => onSave(yaml);

  return (
    <div>
      <Editor height="500px" language="yaml" value={yaml} onChange={handleEditorChange} />
      <button onClick={handleSaveClick}>Save</button>
    </div>
  );
};

export default YamlEditor;
```

---

### Step 2: Create ConfigSelector Microfrontend

**Purpose:**  
Manages CRUD operations for all configuration types, acting as the asset creator and manager.

**Implementation:**

- **Create a new package:**  
  `packages/config-selector`

- **Expose a ConfigManager component to handle configurations for a specified configType.**

- **Integrate the remote YamlEditor microfrontend for editing.**

#### Vite Configuration

```typescript
// packages/config-selector/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'configSelector',
      filename: 'remoteEntry.js',
      exposes: {
        './ConfigManager': './src/ConfigManager.tsx',
      },
      shared: ['react', 'react-dom', 'shared'],
    }),
  ],
  build: {
    target: 'esnext',
  },
});
```

#### ConfigManager Component

```tsx
// packages/config-selector/src/ConfigManager.tsx
import React, { useState, useEffect } from 'react';
import { Box, Button, Typography } from '@mui/material';
import { api, configTypes } from 'shared';
import RemoteComponent from '../utils/RemoteComponent';

interface ConfigManagerProps {
  configType: string;
}

const ConfigManager: React.FC<ConfigManagerProps> = ({ configType }) => {
  const [configs, setConfigs] = useState<any[]>([]);
  const [editingConfig, setEditingConfig] = useState<any | null>(null);
  const metadata = configTypes[configType];

  useEffect(() => {
    fetchConfigs();
  }, [configType]);

  const fetchConfigs = async () => {
    const response = await api.get(metadata.apiEndpoints.list);
    setConfigs(response.data);
  };

  const handleCreateNew = () => setEditingConfig({ id: '', yaml: '' });
  const handleEdit = (config: any) => setEditingConfig(config);
  const handleSave = async (yaml: string) => {
    const endpoint = editingConfig.id ? metadata.apiEndpoints.update : metadata.apiEndpoints.create;
    await api.post(endpoint, { id: editingConfig.id, yaml });
    setEditingConfig(null);
    fetchConfigs();
  };

  return (
    <Box>
      <Typography variant="h5">{metadata.name} Manager</Typography>
      <Button onClick={handleCreateNew}>Create New</Button>
      <ul>
        {configs.map((config) => (
          <li key={config.id}>
            {config.id}
            <Button onClick={() => handleEdit(config)}>Edit</Button>
          </li>
        ))}
      </ul>
      {editingConfig && (
        <RemoteComponent
          url="http://localhost:3002/remoteEntry.js" // YamlEditor URL
          scope="yamlEditor"
          module="./YamlEditor"
          props={{ initialYaml: editingConfig.yaml, onSave: handleSave }}
        />
      )}
    </Box>
  );
};

export default ConfigManager;
```

---

### Step 3: Create JobManagement Microfrontend

**Purpose:**  
Manages job creation, submission, and monitoring.

**Implementation:**

- **Create a new package:**  
  `packages/job-management`

- **Expose a JobManager component for selecting configurations, submitting jobs, and displaying job status.**

#### Vite Configuration

```typescript
// packages/job-management/vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'jobManagement',
      filename: 'remoteEntry.js',
      exposes: {
        './JobManager': './src/JobManager.tsx',
      },
      shared: ['react', 'react-dom', 'shared'],
    }),
  ],
  build: {
    target: 'esnext',
  },
});
```

#### JobManager Component

```tsx
// packages/job-management/src/JobManager.tsx
import React, { useState, useEffect } from 'react';
import { Box, Button, Select, MenuItem, Typography } from '@mui/material';
import { api, configTypes } from 'shared';

const requiredConfigs = ['workorder', 'teamconfig', 'runtimeconfig'];

const JobManager: React.FC = () => {
  const [selectedConfigs, setSelectedConfigs] = useState<Record<string, string>>({});
  const [configs, setConfigs] = useState<Record<string, any[]>>({});
  const [jobs, setJobs] = useState<any[]>([]);

  useEffect(() => {
    fetchConfigs();
    fetchJobs();
  }, []);

  const fetchConfigs = async () => {
    const allConfigs: Record<string, any[]> = {};
    for (const type of requiredConfigs) {
      const response = await api.get(configTypes[type].apiEndpoints.list);
      allConfigs[type] = response.data;
    }
    setConfigs(allConfigs);
  };

  const fetchJobs = async () => {
    const response = await api.get('/api/v1/jobs');
    setJobs(response.data);
  };

  const handleSelectConfig = (type: string, configId: string) => {
    setSelectedConfigs((prev) => ({ ...prev, [type]: configId }));
  };

  const handleSubmitJob = async () => {
    if (requiredConfigs.every((type) => selectedConfigs[type])) {
      const jobData = {
        configs: requiredConfigs.reduce((acc, type) => {
          acc[type] = { id: selectedConfigs[type] };
          return acc;
        }, {} as Record<string, any>),
      };
      await api.post('/api/v1/jobs', jobData);
      fetchJobs();
      setSelectedConfigs({});
    } else {
      alert('Please select all required configurations');
    }
  };

  return (
    <Box>
      <Typography variant="h4">Job Manager</Typography>
      <Box>
        <Typography variant="h5">Create Job</Typography>
        {requiredConfigs.map((type) => (
          <Box key={type} mb={2}>
            <Typography>{configTypes[type].name}</Typography>
            <Select
              value={selectedConfigs[type] || ''}
              onChange={(e) => handleSelectConfig(type, e.target.value)}
              fullWidth
            >
              <MenuItem value="">Select {configTypes[type].name}</MenuItem>
              {configs[type]?.map((config) => (
                <MenuItem key={config.id} value={config.id}>
                  {config.id}
                </MenuItem>
              ))}
            </Select>
          </Box>
        ))}
        <Button variant="contained" onClick={handleSubmitJob}>
          Submit Job
        </Button>
      </Box>
      <Box mt={4}>
        <Typography variant="h5">Jobs</Typography>
        <ul>
          {jobs.map((job) => (
            <li key={job.id}>
              {job.id} - {job.status}
            </li>
          ))}
        </ul>
      </Box>
    </Box>
  );
};

export default JobManager;
```

---

### Step 4: Update Shell Application

**Purpose:**  
Integrate ConfigSelector and JobManagement microfrontends, providing navigation and layout.

**Implementation:**

- **Dynamic Sidebar:**  
  Create a sidebar component that reads configuration metadata from `configTypes` and renders navigation items for each config type.

#### ConfigTypeSidebar Component

```tsx
// packages/shell/src/components/ConfigTypeSidebar.tsx
import React from 'react';
import { Box, List, ListItem, ListItemButton, ListItemText } from '@mui/material';
import { configTypes } from 'shared/config/configTypes';

interface ConfigTypeSidebarProps {
  onSelect: (configType: string) => void;
  selectedType: string;
}

const ConfigTypeSidebar: React.FC<ConfigTypeSidebarProps> = ({ onSelect, selectedType }) => {
  return (
    <Box sx={{ width: 240, bgcolor: 'background.paper' }}>
      <List>
        {Object.entries(configTypes).map(([key, meta]) => (
          <ListItem key={key} disablePadding>
            <ListItemButton selected={selectedType === key} onClick={() => onSelect(key)}>
              <ListItemText primary={meta.name} />
            </ListItemButton>
          </ListItem>
        ))}
        <ListItem button component="a" href="/jobs">
          <ListItemText primary="Jobs" />
        </ListItem>
      </List>
    </Box>
  );
};

export default ConfigTypeSidebar;
```

#### App Component

```tsx
// packages/shell/src/App.tsx
import React, { useState } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import { Box, AppBar, Toolbar, Typography, Drawer, List, ListItem, ListItemText } from '@mui/material';
import { configTypes } from 'shared/config/configTypes';
import RemoteComponent from './utils/RemoteComponent';
import ConfigTypeSidebar from './components/ConfigTypeSidebar';

const drawerWidth = 240;

const App: React.FC = () => {
  const [selectedConfigType, setSelectedConfigType] = useState<string | null>(null);

  return (
    <BrowserRouter>
      <Box sx={{ display: 'flex' }}>
        <AppBar position="fixed" sx={{ zIndex: 1201 }}>
          <Toolbar>
            <Typography variant="h6">C4H Editor</Typography>
          </Toolbar>
        </AppBar>
        <Drawer
          variant="permanent"
          sx={{ width: drawerWidth, flexShrink: 0, '& .MuiDrawer-paper': { width: drawerWidth } }}
        >
          <Toolbar />
          <ConfigTypeSidebar onSelect={setSelectedConfigType} selectedType={selectedConfigType || ''} />
        </Drawer>
        <Box component="main" sx={{ flexGrow: 1, p: 3 }}>
          <Toolbar />
          {selectedConfigType && (
            <RemoteComponent
              url="http://localhost:3003/remoteEntry.js" // ConfigSelector URL
              scope="configSelector"
              module="./ConfigManager"
              props={{ configType: selectedConfigType }}
            />
          )}
          <Routes>
            <Route
              path="/jobs"
              element={
                <RemoteComponent
                  url="http://localhost:3004/remoteEntry.js" // JobManagement URL
                  scope="jobManagement"
                  module="./JobManager"
                />
              }
            />
            <Route path="/" element={<Typography>Select an option from the sidebar</Typography>} />
          </Routes>
        </Box>
      </Box>
    </BrowserRouter>
  );
};

export default App;
```

---

### Step 5: Enhance Shared Package

**Purpose:**  
Provide shared utilities and configuration metadata across microfrontends.

**Implementation:**

#### Example ConfigTypes

```typescript
// packages/shared/src/config/configTypes.ts
export const configTypes = {
  workorder: {
    name: 'Work Orders',
    apiEndpoints: { list: '/api/v1/workorders', create: '/api/v1/workorders', update: '/api/v1/workorders' },
  },
  teamconfig: {
    name: 'Team Configs',
    apiEndpoints: { list: '/api/v1/teamconfigs', create: '/api/v1/teamconfigs', update: '/api/v1/teamconfigs' },
  },
  runtimeconfig: {
    name: 'Runtime Configs',
    apiEndpoints: { list: '/api/v1/runtimeconfigs', create: '/api/v1/runtimeconfigs', update: '/api/v1/runtimeconfigs' },
  },
};
```

#### Example Remotes

```typescript
// packages/shared/src/config/remotes.ts
export const remotes = {
  yamlEditor: 'http://localhost:3002/remoteEntry.js',
  configSelector: 'http://localhost:3003/remoteEntry.js',
  jobManagement: 'http://localhost:3004/remoteEntry.js',
};
```

---

### Step 6: Update API and Backend

**Purpose:**  
Support CRUD operations for all config types and job submissions with configuration tuples.

**Implementation:**

- **Config Endpoints:**  
  Implement CRUD endpoints for each config type (e.g., `/api/v1/workorders`, `/api/v1/teamconfigs`, `/api/v1/runtimeconfigs`).

- **Job Endpoint:**  
  Update `/api/v1/jobs` to accept a payload like:

```json
{
  "configs": {
    "workorder": { "id": "wo-123" },
    "teamconfig": { "id": "tc-456" },
    "runtimeconfig": { "id": "rc-789" }
  }
}
```

---

### Step 7: Testing and Migration

**Purpose:**  
Ensure reliability and a smooth transition to the new architecture.

**Implementation:**

- **Unit Tests:**  
  Test individual components (e.g., YamlEditor, ConfigManager, JobManager).

- **Integration Tests:**  
  Verify microfrontend integration within the shell.

- **End-to-End Tests:**  
  Simulate the full workflow: editing configurations, submitting a job, and monitoring job status.

- **Migration Plan:**
  - Deploy the YamlEditor microfrontend and update ConfigEditor (if in use) to consume it.
  - Introduce ConfigSelector to replace legacy ConfigEditor functionality.
  - Add JobManagement, migrating job-related features from the shell.
  - Update shell navigation and remove redundant code.

---

## 3. Design Rationale

- **Modularity:**  
  Each microfrontend has a distinct responsibility, aligning with asset management and job handling concepts.

- **Scalability:**  
  New config types can be added by updating `configTypes.ts`, with the shell and ConfigSelector adapting automatically.

- **Reusability:**  
  The YamlEditor is a reusable editor, integrated into ConfigSelector for all config types.

- **Separation of Concerns:**  
  ConfigSelector manages configurations, JobManagement handles jobs, and the shell provides integration, reducing coupling.

- **API Extensibility:**  
  The backend is designed to support multiple config types and a tuple-based job submission with minimal changes for future additions.

---

## 4. Conclusion

This refactoring plan transforms the C4H Editor into a modular, microfrontend-based architecture. The **ConfigSelector** microfrontend manages all config types, the **JobManagement** microfrontend handles job workflows, and the **YamlEditor** microfrontend provides specialized YAML editing. The shell integrates these components into a dynamic, user-friendly interface with a dynamic sidebar for navigation. This design enhances flexibility, supports future growth, and meets the requirements for separate microfrontends with minimal code changes.