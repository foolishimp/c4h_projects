
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
```

```typescript
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
```

```typescript
// Safe date handling
const createMetadata = (): WorkOrderMetadata => ({
  author: getCurrentUser(),
  created_at: new Date().toISOString(),
  updated_at: new Date().toISOString(),
  version: '1.0.0',
  tags: []
});
```

```typescript
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
```
