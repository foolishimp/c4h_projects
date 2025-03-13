# Agent Design Principles

## Overarching Principles

1. **LLM-First Processing**
   - Offload most logic and decision-making to the LLM
   - Use LLM for verification and validation where possible
   - Template Configuration Over Code:
     - Maintain all prompts in configuration files rather than hard-coding
     - Simplifies prompt engineering and updates
   - Agents focus on:
     - Managing intent prompts
     - Processing responses
     - Managing local environment side effects

2. **Minimal Agent Logic**
   - Keep agent code focused on infrastructure concerns
   - Avoid embedding business logic in agents
   - Let LLM handle complex decision trees
   - Only transform data if essential for the agent's infrastructure role

## Implementation Principles

### 1. Single Responsibility
- Each agent has one clear, focused task
- No processing of tasks that belong to other agents
- Pass through data without unnecessary interpretation
- Item-agnostic components:
  - Avoid assumptions about item structure
  - Let caller define what to extract or transform
- Example: Discovery agent handles only file analysis, Solution Designer only creates prompts

### 2. Minimal Processing
- Default to passing data through to LLM
- Only transform data if it's core to the agent's infrastructure role
- Don't duplicate validation or processing done by other agents
- Let LLM handle data interpretation where possible

### 3. Clear Boundaries
- Discovery agent handles file analysis and scoping
- Solution Designer creates optimal prompts
- Semantic Iterator handles response parsing and iteration
- Configuration Inheritance Chain:
  - System-level config flows downward to each agent
  - Agents pass complete config to child components
- Use placeholder variables and clear format requirements
- Each agent trusts other agents' output
- No cross-agent validation

### 4. Logging Over Validation
- Focus on detailed logging for debugging
- Let calling agents handle validation
- Structured Logging & Metrics:
  - Include timestamps and correlation IDs
  - Log key events, inputs, and outputs
  - Provide enough detail to reconstruct actions
- Make agent behavior observable
- Reserve validation for infrastructure concerns only

### 5. Error Handling
- Handle only errors specific to infrastructure tasks
- Pass through errors from external services (like LLM)
- Provide clear error context in logs
- Don't swallow or transform errors unnecessarily
- Let LLM handle business logic errors

### 6. Stateless Operation
- Agents don't maintain state between operations
- Each request is self-contained
- State management happens at orchestration level
- Makes testing and debugging simpler
- Enables clean sequential processing

### 7. Composability
- Agents can be chained together
- Output format matches input format of next agent
- No hidden dependencies between agents
- Clean interfaces between agents
- All assumptions about data structures are explicit in contracts

### 8. Observable Behavior
- Extensive structured logging
- Clear input/output contracts
- Traceable request/response flow
- Debuggable operation
- Include tracing info when needed

### 9. Focused Testing
- Test only the agent's infrastructure responsibility
- Don't test downstream agent behavior
- Mock external services appropriately
- Test logging and error handling
- Don't test LLM decision logic
- Keep agent tests lean and focused on I/O handling

### 10. Forward-Only Flow
- Data flows forward through agent chain
- No backward dependencies
- Each agent adds its specific value
- Clean sequential processing
- Simplifies testing and debugging

## Practical Examples

### Good Agent Design
```python
class SolutionDesigner:
    async def process(self, context: Dict[str, Any]) -> AgentResponse:
        # Log receipt
        self.logger.info("design_request_received", intent=context.get('intent'))
        
        # Pass to LLM for processing
        return await self.llm.process(self._format_request(context))
```

## Benefits
- Simpler codebase
- Easier to maintain
- More flexible and adaptable
- Better separation of concerns
- Clearer responsibility boundaries
- More testable infrastructure
- Leverages LLM capabilities optimally

## Application Guidelines
1. When adding validation, ask "Is this infrastructure or business logic?"
2. When adding processing, ask "Could the LLM handle this?"
3. Keep agent code focused on:
   - Managing I/O
   - Logging
   - Infrastructure error handling
   - Environment interactions
4. Let the LLM handle:
   - Business validation
   - Code analysis
   - Decision making
   - Content transformation

The principles above produce a robust, testable, and adaptable environment for coding tasks, data extraction, and other automation scenarios while maintaining clean, modular, and maintainable agent-based systems.