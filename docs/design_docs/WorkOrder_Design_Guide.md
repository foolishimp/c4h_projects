# **C4H Agent System: Work Order Design Guidelines**

## **Purpose of This Guide**

This document provides guidance on how to create effective prompts (work orders) for the C4H Agent System. It covers the structure, key components, and best practices for designing prompts that align with the system's design principles.

## **Config Structure Overview**

The C4H Agent System uses a hierarchical configuration model with three main sections:

1. **System Configuration** - Defines the agent infrastructure, providers, and global settings  
2. **Intent Configuration (Work Order)** - Specifies what you want the system to do

## **System Configuration Section**

The system configuration establishes the infrastructure and capabilities of the agent system:

- Path references within the project subdirectory should be relative to the project
- Path references outside the project, such as references to documents should be full paths.

project:  
  path: "/path/to/your/project"  
  workspace_root: "workspaces"

llm_config:  
  agents:  
    discovery_agent:  
      tartxt_config:  
        input_paths:  
          - "path/to/folder/to/analyze"
        exclusions:
          - "**/__pycache__/**"  
          - "**/.git/**"

    solution_designer:  
      provider: "anthropic"  
      model: "claude-3-7-sonnet-20250219"  
      temperature: 1  
      extended_thinking:  
        enabled: true  
        budget_tokens: 32000

    coder:  
      provider: "anthropic"  
      model: "claude-3-7-sonnet-20250219"  
      temperature: 0  
      backup_enabled: true

runtime:  
  workflow:  
    storage:  
      enabled: true  
      root_dir: "workspaces/workflows"  
      format: "yymmdd_hhmm_{workflow_id}"

logging:  
  level: "INFO"  
  format: "structured"  
  agent_level: "DEBUG"

### **Key System Configuration Components**

1. **Project Settings** (Required)  
   * path: Absolute path to the project being processed  
   * workspace_root: Where workspace and outputs are stored  
2. **LLM Configuration**  
   * **Discovery Agent**: Analyzes project structure  
     * tartxt_config.input_paths: Directories to analyze  
     * tartxt_config.exclusions: Patterns to exclude  
   * **Solution Designer**: Creates the refactoring plan  
     * provider: LLM provider (anthropic, openai, gemini)  
     * model: Specific model to use  
     * temperature: Creativity level (0-1)  
     * extended_thinking: For complex reasoning (Claude models)  
   * **Coder**: Implements the changes  
     * Similar settings to Solution Designer  
     * backup_enabled: Whether to backup files before modifying  
3. **Runtime Settings**  
   * Storage configuration for workflow artifacts  
   * Error handling and retry policies  
4. **Logging Configuration**  
   * Controls verbosity and format of logs

## **Intent Configuration (Work Order) Section (Required)**

The intent configuration defines what you want the system to do:

intent:  
  description: |  
    A clear, detailed description of what you want to accomplish.

    For example:  
    "Refactor the codebase to implement a consistent error handling system,  
    where all exceptions are logged with contextual information and proper  
    hierarchical categorization."

    Include:  
    1. The goal of the refactoring  
    2. Specific requirements and constraints  
    3. Implementation details if necessary  
    4. Any design principles that should be followed

    Be specific about what files or components should be modified.  
    Explain the desired outcome in detail.

## **Best Practices for Intent Description**

1. **Be Clear and Specific**  
   * State exactly what you want the system to do  
   * Specify which files or components should be modified  
   * Define the expected outcome  
2. **Provide Context**  
   * Explain why the change is needed  
   * Reference existing patterns or principles to follow  
   * Mention any constraints that must be respected  
3. **Structure with Numbered Points**  
   * Break down complex requirements into numbered sections  
   * Prioritize items if appropriate  
   * Use hierarchical structure for clarity  
4. **Include Implementation Guidelines**  
   * Specify technical approaches if needed  
   * Reference specific patterns or libraries to use  
   * Define any architectural boundaries  
5. **Specify Validation Criteria**  
   * How will success be measured?  
   * Any specific tests or checks that should be performed  
   * Expected performance characteristics  
6. **Provide Context and Background**  
   * Explain the current state of the code  
   * Describe any existing patterns or conventions  
   * Mention relevant design documents or principles  
7. **Use Structured Sections**  
   * Goal/Objective (what you want to accomplish)  
   * Current Implementation Context (what exists now)  
   * Required Changes (what needs to change)  
   * Design Principles to Follow (how to approach the changes)  
   * Files to Modify (where to make changes)  
8. **Reference Existing Code**  
   * Mention specific files, functions, or classes that need modification

## **Complete Work Order Template**

Here's a complete template combining both system and intent configurations:

# Work Order Configuration  
workorder:  
   # Project settings  
   project:  
   path: "/path/to/your/project"  
   workspace_root: "workspaces"

   # Intent description (the actual work order)  
   # sub headings beneath description are only for clarity to the LLM  
   intent:  
   description: |  
      # [TITLE OF WORK ORDER]

      ## Goal  
      [CLEAR STATEMENT OF END GOAL]

      ## Current Implementation Context  
      - [DESCRIPTION OF CURRENT CODE]  
      - [RELEVANT PATTERNS OR CONVENTIONS]  
      - [ISSUES WITH CURRENT IMPLEMENTATION]

      ## Required Changes  
      1. [FIRST CHANGE]  
      2. [SECOND CHANGE]  
      3. [THIRD CHANGE]

      **READ** and **ADHERE** to the principles outlined in the relevant design documents.

      Implementation Requirements:  
      - [REQUIREMENT 1]  
      - [REQUIREMENT 2]  
      - [REQUIREMENT 3]  
      - [REQUIREMENT 2]  
      - [REQUIREMENT 3]

      Design Principles to Follow:  
      - [PRINCIPLE 1]  
      - [PRINCIPLE 2]

      ## Files to Modify  
      - [FILE PATH 1] - [BRIEF DESCRIPTION OF CHANGES NEEDED]  
      - [FILE PATH 2] - [BRIEF DESCRIPTION OF CHANGES NEEDED]

# LLM configuration (Specific overrides for this Work Order)  
# This section is merged with the base system_config.yml  
llm_config:  
  agents:  
    discovery_agent:
      agent_type: "generic_single_shot"
      persona_key: "discovery"
      name: "discovery_phase"
      tartxt_config:
        input_paths:
          # Example: Include project source and a specific design doc
          - "src/" # Relative path within the project
          - "/Users/jim/src/apps/c4h_projects/docs/design_docs/Relevant_Design_Doc.md" # Absolute path for external doc  
        exclusions:  
          - "**/__pycache__/**"  
          - "**/.git/**"

    solution_designer:  
      # Example: Override model or temperature for this specific job  
      agent_type: "generic_single_shot"
      persona_key: "solution_designer"
      name: "solution_design_phase"
      provider: "anthropic"
      model: "claude-3-7-sonnet-20250219"
      temperature: 1
      extended_thinking:  
        enabled: true  
        budget_tokens: 32000

## **Work Order Examples**

Here are some example work orders for common tasks:

### **Example 1: Adding Error Handling**

intent:  
  description: |  
    Add comprehensive error handling to the application with proper logging.

    Goal:  
    Implement a consistent error handling system across all service functions that captures  
    context, provides clear error messages, and ensures proper logging.

    Required Changes:  
    1. Create a centralized error handling utility  
    2. Add try/catch blocks to all service functions  
    3. Implement context-aware error logging  
    4. Add user-friendly error messages for API responses

    Implementation Requirements:  
    - Use structured error objects with error codes  
    - Include stack traces in dev mode only  
    - Log errors with appropriate severity levels  
    - Ensure all errors are properly propagated

    Files to Modify:  
    - src/services/*.js  
    - src/api/routes/*.js  
    - src/utils/logger.js

### **Example 2: Refactoring for Performance**

intent:  
  description: |  
    Optimize database queries in the data access layer to improve performance.

    Goal:  
    Reduce API response times by optimizing database queries, implementing caching,  
    and reducing unnecessary data fetching.

    Required Changes:  
    1. Add indexes to frequently queried fields  
    2. Implement query batching for related data  
    3. Add Redis caching for frequently accessed data  
    4. Optimize SELECT queries to fetch only needed fields

    Implementation Requirements:  
    - Query execution time should not exceed 100ms  
    - Cache invalidation must be properly implemented  
    - No changes to API contracts or response formats  
    - Add performance metrics logging

    Design Principles:  
    - Minimize database round trips  
    - Cache at appropriate layers  
    - Prefer eager loading over N+1 queries

    Files to Modify:  
    - src/data/repositories/*.js  
    - src/models/data-access/*.js  
    - src/config/database.js

### **Example 3: Adding a New Feature**

intent:  
  description: |  
    Add user authentication with JWT tokens and role-based access control.

    Goal:  
    Implement a secure authentication system with JWT tokens, user roles,  
    and protected routes.

    Required Changes:  
    1. Create authentication middleware  
    2. Implement JWT token generation and validation  
    3. Add user roles and permissions system  
    4. Protect API routes based on user roles

    Implementation Requirements:  
    - Use bcrypt for password hashing  
    - Implement token refresh mechanism  
    - Store user roles in database  
    - Add route protection based on roles

    Design Principles:  
    - Separation of authentication and authorization  
    - Clean middleware architecture  
    - Security best practices (OWASP)

    Files to Create:  
    - src/auth/middleware.js  
    - src/auth/token-service.js  
    - src/auth/role-service.js

    Files to Modify:  
    - src/api/routes/*.js  
    - src/models/user.js  
    - src/config/app.js

### **Example 4: Frontend Component Refactoring**

intent:  
  description: |  
    # Refactor Frontend Grid Components

    ## Goal  
    Standardize and enhance the frontend grid components used for displaying data lists.  
    Implement features for sorting, pagination, dynamic filtering, and make rows clickable  
    for navigation, following Material UI best practices.

    ## Current Implementation Context  
    - The current grid components use basic Material UI Tables  
    - They lack consistent sorting, pagination, and filtering capabilities  
    - Each row has an explicit "Actions" column with buttons  
    - Different components use inconsistent patterns for data display

    ## Required Changes  
    **READ** and **ADHERE** to the principles outlined in the Typescript_Design_Principles.md document.

    1. Implement Sorting:  
       - Add clickable column headers with TableSortLabel  
       - Implement sort state management  
       - Apply sorting logic to displayed data

    2. Implement Pagination:  
       - Add TablePagination component  
       - Configure for 25, 50, and 100 rows per page  
       - Implement page state management

    3. Implement Filtering:  
       - Add search field with dynamic filtering  
       - Update displayed rows based on filter text

    4. Make Rows Clickable:  
       - Remove dedicated "Actions" column  
       - Make entire row clickable for navigation  
       - Add appropriate styling for clickable rows

    ## Design Principles to Follow  
    - Follow TypeScript design principles for type safety  
    - Use Material UI components according to documentation  
    - Maintain separation of concerns  
    - Ensure components remain reusable

    ## Files to Modify  
    - src/components/DataGrid.tsx - Implement base grid functionality  
    - src/components/UserList.tsx - Update to use enhanced grid  
    - src/components/ConfigList.tsx - Update to use enhanced grid  
    - src/contexts/DataContext.tsx - Update state management if needed

## **Workflow Sequence**

When the C4H Agent System processes your work order, it follows this sequence:

1. **Discovery Phase**  
   * Analyzes project structure and codebase based on tartxt_config.input_paths.  
   * Identifies relevant files and patterns.
   * Creates a codebase understanding model (the tartxt output).  
2. **Solution Design Phase**  
   * Interprets your intent description.  
   * Creates a detailed plan for changes based on the discovered code context.  
   * Identifies specific modifications needed.  
   * Designs the solution architecture.  
3. **Coding Phase**  
   * Implements the changes according to the plan.  
   * Makes precise modifications to the codebase.  
   * Creates new files if needed.  
   * Preserves existing functionality.

## **Tips for Optimal Results**

1. **Be Specific and Detailed**  
   * The more specific your intent description, the better the results.  
   * Include examples where helpful.  
2. **Reference Design Principles**  
   * Mention architectural patterns to follow.  
   * Reference code style or design principles.  
3. **Set Clear Boundaries**  
   * Specify which parts of the code should change.  
   * Indicate what should not change.  
4. **Start Small**  
   * Begin with focused, smaller work orders.  
   * Build confidence in the system before tackling major refactors.  
5. **Review the Plan**  
   * Check the solution design before proceeding to implementation.  
   * Provide feedback if necessary.  
6. **Use Structured Formatting**  
   * Markdown headings and lists improve readability.  
   * Numbered lists help prioritize changes.  
   * Code blocks for examples are helpful.  
7. **Reference Design Documents & Provide Context**  
   * Explicitly mention relevant design documents.  
   * Highlight key principles that should be followed.  
   * Explain *why* changes are needed.  
   * Describe the current implementation and its limitations.  
   * **Specify Context File Paths:** When referencing external documents (like design guides) needed for context, ensure they are included in the llm_config.agents.discovery.tartxt_config.input_paths list within your Work Order configuration.  
     * Paths relative to the project.path (e.g., src/, docs/internal_spec.md) will be resolved correctly.  
     * For documents *outside* the main project.path (e.g., shared design documents in a separate c4h_projects/docs directory), you **must provide the full, absolute path** in the input_paths list (e.g., /Users/jim/src/apps/c4h_projects/docs/design_docs/Agent_Design_Principles_v2.md). tartxt requires this to locate files outside the project's relative scope.

By following these guidelines, you'll create effective work orders that leverage the full capabilities of the C4H Agent System while adhering to its design principles.