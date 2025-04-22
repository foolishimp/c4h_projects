# Microfrontend Integration Recipe: Fixing Common Pitfalls

## Root Cause Analysis

The primary issues with our microfrontend system stemmed from:

1. **React Context Boundary Problems**: MUI components failed because they couldn't access ThemeProvider context
2. **Configuration Type Mismatches**: Shell used plural IDs (`workorders`) while components expected singular types (`workorder`)
3. **Missing Navigation Handling**: Shell-to-MFE communication for navigation actions wasn't properly connected

## Implementation Template for New MFEs

### 1. Always Include Required React Providers

```typescript
export function mount(props) {
  const { domElement, customProps } = props;
  
  const root = createRoot(domElement);
  root.render(
    <StyledEngineProvider injectFirst>
      <ThemeProvider theme={createTheme()}>
        <CssBaseline />
        <AppSpecificProvider>
          <YourComponent {...customProps} />
        </AppSpecificProvider>
      </ThemeProvider>
    </StyledEngineProvider>
  );
  
  return { unmount: () => root.unmount() };
}
```

### 2. Normalize Configuration Types

```typescript
// Map between shell app IDs and component config types
const appIdToConfigType = {
  'config-selector-workorders': 'workorder',
  'config-selector-teamconfigs': 'teamconfig'
};

// Extract correct type from app ID
function getConfigType(props) {
  // Check for explicit type in props first
  if (props.configType) return props.configType;
  
  // Then try mapping from app name/ID
  const appId = props.name || '';
  return appIdToConfigType[appId] || extractFromPattern(appId);
}
```

### 3. Implement Fallback Navigation

```typescript
// Add navigation handlers with fallbacks
const onNavigateTo = customProps.onNavigateTo || ((id) => {
  // Use event bus if callback not provided
  eventBus.publish('navigation:request', {
    source: 'your-mfe',
    payload: { action: 'navigateTo', target: id }
  });
});
```

### 4. In Shell App: Always Pass Navigation Callbacks

```typescript
// In App.tsx mount function
if (appDef.id.startsWith('config-selector-')) {
  Object.assign(customProps, {
    configId: activeConfigId,
    onNavigateBack: () => setActiveConfigId(null),
    onNavigateTo: (id) => setActiveConfigId(id)
  });
}
```

## Key Pitfalls to Avoid

1. **Don't assume React context works across MFE boundaries** - Each MFE needs its own providers
2. **Don't rely on import maps for shared code** - Bundle shared libraries into each MFE
3. **Don't miss type conversion between shell and MFEs** - Normalize IDs/types consistently
4. **Don't forget fallback navigation mechanisms** - Use event bus when direct callbacks aren't available
5. **Don't ignore typescript errors in window globals** - Properly type `window` augmentations

Following this recipe ensures that new microfrontends will properly integrate with the shell application, maintaining context isolation while enabling robust communication.