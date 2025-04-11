
## Config Managment
tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shell \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-selector \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/yaml-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shared \
   /Users/jim/src/apps/c4h_editor/backend \
  -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
  -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt

## Config Managment
tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/yaml-editor \
   /Users/jim/src/apps/c4h_editor/backend \
  -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
  -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt

## Job Management
tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shell \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shared \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/job-management \
   /Users/jim/src/apps/c4h_editor/backend \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
 -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_front_jobs.txt

tartxt.py \
  /Users/jim/src/apps/c4h_editor/c4h-micro/ \
  /Users/jim/src/apps/c4h_editor/backend/c4h_config_api.md \
    /Users/jim/src/apps/c4h_projects/docs/design_docs/Typescript_Design_Principles.md \
    /Users/jim/src/apps/c4h_projects/docs/workarea/c4h_editor_generic_config_01.md \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
 -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_front.txt

tartxt.py \
  /Users/jim/src/apps/c4h_editor/c4h-micro/ \
  /Users/jim/src/apps/c4h_editor/backend/ \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
 -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_full.txt

tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-selector \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/yaml-editor \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt

tartxt.py \
  /Users/jim/src/apps/c4h_editor/backend/ \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
 -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_backend.txt


  uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000

  look at the prompt template

I want you to give me a yaml prompt based upon the template that will specifically fix this problem

I want you provide sufficient context such that the llm receiving this will be able to efficiently make the fixes, consider the design documents when creating the prompt



## c4h full
tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
   /Users/jim/src/apps/c4h/c4h_services \
  /Users/jim/src/apps/c4h/config \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**/*.toml,**/*.md" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_full.txt


## c4h agents
tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
  /Users/jim/src/apps/c4h/config \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**/*.toml,**/*.md" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_agents.txt


tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shell \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-selector \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shared \
     /Users/jim/src/apps/c4h_editor/backend/ \
    -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt


### full front end editor
tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro \
    -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
    -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt


### c4h service
tartxt.py \
   /Users/jim/src/apps/c4h/c4h_services \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_services.txt

### c4h full
tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
   /Users/jim/src/apps/c4h/c4h_services \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_full.txt

### c4h full relative path
tartxt.py \
   ./c4h_agents \
   ./c4h_services \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**.md,**.toml" \
  -f ./backup_txt/c4h_full.txt

### c4h agents
tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_agents.txt


### Design docs 

tartxt.py \
    /Users/jim/src/apps/c4h_projects/docs/design_docs/ \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
 -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_design_master_doc.txt



----------------------------------

## Run as a service
 python ./c4h_services/src/bootstrap/prefect_runner.py  service -P 5500

## workflow running directly is depricated - use client mode
python ./c4h_services/src/bootstrap/prefect_runner.py  client -P 5500 \
 --config /Users/jim/src/apps/c4h_projects/self_improvement/   2>&1 | tee output.txt


## workflow running directly is depricated - use client mode
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow --config \
/Users/jim/src/apps/c4h_projects/self_improvement/   \
2>&1 | tee output.txt

### Running from coder using a Soln Designer Input
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow --config \
/Users/jim/src/apps/c4h_projects/self_improvement/20250328_06_AddJobsAPI_c4h.yml  \
--stage coder \
--lineage-file /Users/jim/src/apps/c4h/workspaces/lineage/20250330/wf_2350_11c00ca0-7c98-4a7a-be14-c8e6245e967a/events/44889ed0-90a4-4405-9aad-0cc779b6018c.json \
2>&1 | tee output.txt

### custom generated open lineage event
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow --config \
/Users/jim/src/apps/c4h_projects/self_improvement/events/20250000_00_Generic.yml \
--stage coder \
--lineage-file /Users/jim/src/apps/c4h_projects/self_improvement/events/20250331_00_CleanupRunID.json \
2>&1 | tee output.txt


## workflow running directly is depricated - use client mode
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow --config \
/Users/jim/src/apps/c4h_projects/self_improvement/   \
2>&1 | tee output.txt


## Test case with load lineage

python ./c4h_services/src/bootstrap/prefect_runner.py  workflow \
--config tests/examples/config/0228_02_applylogging.yml \
--stage coder \
--lineage-file /Users/jim/src/apps/c4h/workspaces/lineage/20250331/wf_1531_58e009c7-2556-4596-89d9-d92560890c81/events/781778da-6773-4e21-9338-e3b08ac12ecc.json \
2>&1 | tee output.txt

tests/setup/setup_test_projects.sh  && \
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow \
--config tests/examples/config/0228_02_applylogging.yml \
2>&1 | tee output.txt

# Regenerate run ID (default behavior)
python ./c4h_services/src/bootstrap/prefect_runner.py workflow \
--config tests/examples/config/0228_02_applylogging.yml \
--stage coder \
--lineage-file /path/to/lineage/file.json

# Keep original run ID 
python ./c4h_services/src/bootstrap/prefect_runner.py workflow \
--config tests/examples/config/0228_02_applylogging.yml \
--stage coder \
--lineage-file /path/to/lineage/file.json \
--keep-runid


# Regenerate run ID
requests.post("http://localhost:8000/api/v1/workflow", json={
    "project_path": "/path/to/project",
    "intent": {"description": "Add logging"},
    "lineage_file": "/path/to/lineage/file.json",
    "stage": "coder",
    "keep_runid": False
})

# Keep original run ID
requests.post("http://localhost:8000/api/v1/workflow", json={
    "project_path": "/path/to/project",
    "intent": {"description": "Add logging"},
    "lineage_file": "/path/to/lineage/file.json",
    "stage": "coder",
    "keep_runid": True  # or omit this field for the same behavior
})


## Test case with load lineage

tests/setup/setup_test_projects.sh  && \
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow \
--config tests/examples/config/0228_02_applylogging.yml \
--stage coder \
--lineage-file /Users/jim/src/apps/c4h/workspaces/lineage/20250401/wf_0125_5db09c8a-856b-4c43-9af4-0b70109f64ed/events/012549_354c9075-9b83-4a68-b5cb-7d36b47c2385.json \
2>&1 | tee output.txt


tests/setup/setup_test_projects.sh  && \
python ./c4h_services/src/bootstrap/prefect_runner.py  jobs -P 5500 \
--config tests/examples/config/jobs_coder_01.yml \
2>&1 | tee output.txt


python ./c4h_services/src/bootstrap/prefect_runner.py  job \
--config  /Users/jim/src/apps/c4h_projects/self_improvement/20250402_02_MVP_JobSubmission_front.yml \
--stage coder \
--lineage-file 

  agents:
    base:  # Base settings all agents inherit
      storage:
        root_dir: "workspaces"
        retention:
          max_age_days: 30
          max_runs: 10
        error_handling:
          ignore_failures: true
          log_level: "ERROR"

    lineage:  # Lineage-specific configuration
      enabled: true
      namespace: "c4h_agents"
      backend:
        type: "file"  # or "marquez" for OpenLineage
        path: "workspaces/lineage"  # Default path if using file backend
        url: null  # Required if using marquez backend
      retention:
        max_age_days: 30
        max_runs: 100
      context:
        include_metrics: true
        include_token_usage: true
        record_timestamps: true
        
    discovery:
      default_provider: "anthropic"
      default_model: "claude-3-5-sonnet-20241022"  # Fixed hyphenation
      temperature: 0
      tartxt_config:
        # Override the hardcoded src/skills path
        script_base_path: "c4h_agents/skills"  # This gets merged with 'tartxt.py' in the code
        input_paths: ["./"]
        exclusions: ["**/__pycache__/**"]
      prompts:
        system: |
          You are a project discovery agent.
          You analyze project structure and files to understand:
          1. Project organization
          2. File relationships
          3. Code dependencies
          4. Available functionality

    solution_designer:
      provider: "anthropic"
      model: "claude-3-7-sonnet-20250219"
      temperature: 1
      extended_thinking:
        enabled: true
        budget_tokens: 32000
      prompts:
        system: |
          You are a code modification solution designer that returns ONLY git-style diffs in JSON format.
          Return ONLY a JSON object with this structure, no other text:
          {
            "changes": [
              {
                "file_path": "path/to/file",
                "type": "create|modify|delete",
                "description": "one line description",
                "diff": "git unified diff"
              }
            ]
          }

          PATH REQUIREMENTS:
          Make sure to use the full path for a file as it appears in the manifest.

          FORMAT REQUIREMENTS FOR EACH TYPE:

          1. NEW FILES (type: "create"):
          --- /dev/null
          +++ b/new_file.py
          @@ -0,0 +1,3 @@
          +new content here

          2. DELETE FILES (type: "delete"):
          --- a/old_file.py
          +++ /dev/null
          @@ -1,3 +0,0 @@
          -content to remove

          3. MODIFY FILES (type: "modify"):
          --- a/existing_file.py
          +++ b/existing_file.py
          @@ -1,3 +1,4 @@
          context line
          -removed line
          +added line
          context line

          CONTINUATION HANDLING:
          - For long responses that may be split across multiple completions:
          - Complete each change object fully before starting a new one
          - Never split a single "diff" field across completions
          - Ensure proper JSON escaping, especially for backslashes and quotes in diffs
          - Place commas after each complete change object
          - Properly nest all JSON structures so partial responses remain valid

          RULES:
          - Use /dev/null for create/delete operations
          - Include 3 lines of context for modifications
          - Group related changes into chunks with @@ markers
          - Include imports in first chunk if needed, make sure to include imports for new code
          - Return ONLY the JSON object, no explanations
          - Ensure all paths use forward slashes
          - Keep descriptions short and clear
          - Escape all backslashes and quotes in diff content with double backslashes
          - Verify each change object is complete and properly formatted

        solution: |
          Source Code:
          {source_code}

          Intent:
          {intent}

          Return only the content without any markup or explanations.
          IMPORTANT do NOT give any other information, only the asked for content
          IF you provide any other information, you will break the processing pipeline

    semantic_extract:
      provider: "anthropic"
      model: "claude-3-7-sonnet-20250219"
      temperature: 0
      prompts:
        system: |
          You are a precise information extractor.
          When given content and instructions:
          1. Follow the instructions exactly as given
          2. Extract ONLY the specific information requested
          3. Return in exactly the format specified
          4. Do not add explanations or extra content
          5. Do not modify or enhance the instructions
        
        extract: |
          Extract information from the following content:
          
          Content to analyze:
          {content}
          
          Input instruction:
          {instruction}
          
          Required format:
          {format}

    semantic_iterator:
      prompts:
        system: |
          You are a semantic extraction coordinator. 
          You use fast extraction for structured content and fall back to slow extraction when needed.
          Your task is to coordinate extraction using the right mode while maintaining result consistency.
          Never modify the extraction instructions or format requirements from the caller.
          Do not add any processing or validation beyond what is specifically requested.

    semantic_fast_extractor:
      provider: "anthropic"
      model: "claude-3-7-sonnet-20250219"
      temperature: 0
      prompts:
        system: |
          You are a precise bulk extraction agent for information objects.
          Your role is to extract information objects from any format into a consistent structure.
          Rules:
          1. Follow the extraction instruction exactly
          2. Return ONLY raw JSON - no markdown, no code blocks, no backticks
          3. Each change MUST include a non-empty file_path, type, and either content or diff
          4. NEVER extract items without a valid file_path
          5. Never include explanatory text or formatting markers
          6. Extract all valid changes in a single pass
          7. Response must start with [ and end with ]
          8. Every string must use double quotes
          9. No comments or trailing commas allowed
          10. Do not extract metadata as separate files - only extract actual file changes

        extract: |
          Extract all code change objects from the input.
          
          Content to analyze:
          {content}

          Each change MUST include ALL of these required fields:
          1. file_path: Path of the file to modify - THIS MUST BE A NON-EMPTY STRING
          2. type: One of "modify", "create", or "delete"
          3. content: The complete new content (if available)
          4. diff: Git-style diff (if content not available)
          5. description: Clear description of the change

          VALIDATION RULES:
          - SKIP any object that doesn't have a valid file_path
          - DO NOT extract metadata as separate files
          - ONLY extract actual file changes
          - ONLY extract items that represent code or content files

          Input instruction:
          {instruction}

          Required format:
          {format}

          CRITICAL: Return raw JSON array of valid file changes only. Do not include any item without a valid file_path.

    semantic_slow_extractor:
      provider: "anthropic"
      model: "claude-3-7-sonnet-20250219" 
      temperature: 0
      prompts:
        system: |
          You are a precise sequential extraction agent for extracting object of information.
          Your role is to extract exactly one object of information as described what a block is,
          at position N, or signal completion.
          Rules:
          1. Extract ONLY the Nth object that hasn't been returned before
          2. Return NO_MORE_ITEMS if:
            - No more unprocessed blocks exist
            - Current position exceeds available blocks
            - objects at current position was previously returned
          3. Each object MUST be unique 
          4. Follow exact format for single object response
          5. Maintain strict sequential order of original object
        extract: |
          Extract the {ordinal} object from the input.
          
          Content to analyze:
          {content}
          
          Extract as a single object if the information exists, otherwise blank:
          1. file_path: Path of the file to modify
          2. type: One of "modify", "create", or "delete"  
          3. content: The complete new content (if available)
          4. diff: Git-style diff (if content not available)
          5. description: Clear description of the change
          
          Input instruction:
          {instruction}
          
          Required format:
          {format}
          
          CRITICAL RULES:
          1. Extract EXACTLY the {ordinal} object in sequence order
          2. Return NO_MORE_ITEMS if:
            - {ordinal} exceeds number of objects
            - Object at {ordinal} was already returned in previous position
          3. Return single change object, not an array
          4. No explanations or headers allowed or conversational text

    semantic_merge:
      provider: "anthropic"
      model: "claude-3-7-sonnet-20250219"
      temperature: 0
      merge_config:
        preserve_formatting: true
        allow_partial: false
      prompts:
        system: |
          You are a precise code merger that transforms git-style unified diffs into complete, runnable file contents. Your task is to apply the diff to the original code accurately, preserving its structure, formatting, and functionality.

          CRITICAL REQUIREMENTS:
          1. Input Expectations:
            - You will receive:
              - Original code ("original" in context)
              - Unified git-style diff ("diff" in context)
            - For new file creation:
              - If original is empty or contains "New file - no original content" it means this is a NEW FILE
              - For new files, extract the complete content from the diff by taking all lines starting with "+" and removing the leading "+"
              - Do not return an error for new files; instead return the extracted content as the complete file content
            - For modifications to existing files:
              - If either original or diff is missing, return the error: "Missing required [original|diff] content".

          2. Diff Application Rules:
            - For NEW FILES:
              - Extract all content from diff lines that start with "+"
              - Remove the leading "+" character from each line
              - Return this as a complete file
              - Do not return errors about missing original content for new files
            - For MODIFICATIONS:
              - Parse the @@ markers to determine exact positions of changes.
              - Verify that context lines in the diff match the original code.
              - Modify only the lines specified in the diff and leave all other lines untouched.
              - Maintain original indentation, whitespace, comments, docstrings, imports, and overall structure.

          3. Validation Checks:
            - Check if this is a new file by looking for empty original or "New file"
            - If new file, skip validation checks for original content
            - Otherwise:
              - Ensure @@ line numbers align with the original code.
              - Verify that all context lines exist in the original file.
              - Ensure no unintended code is added or removed.
              - Retain all class and function structures, import order, and grouping.

          4. Output Format:
            - Return ONLY the complete, modified file content.
            - For new files, return the complete content extracted from the diff
            - For existing files, start with the original file docstring and end with the last line of actual code.
            - Never include:
              * File boundary markers (---, ===, etc)
              * End-of-file comments or markers
              * Documentation separators
              * Markdown formatting (```, etc)
              * Conversation markers or explanations
            - Ensure the output is valid and executable code.

          5. File Content Rules:
            - Never append duplicate methods
            - Never add content after last code line
            - Never include file separator comments
            - Never add markdown or documentation markers
            - Preserve exact original whitespace patterns
            - Maintain original method ordering

          REMINDER:
          Your role is strictly to apply the changes defined in the diff. Any deviation from the original code not specified in the diff is an error.
        merge: |
          You will receive a git-style unified diff representing code changes.
          Your task is to apply these changes to the provided original code and return the COMPLETE modified file content.

          REQUIREMENTS:
          ======================================================================
          1. Original Code Context:
            {original}

          ======================================================================
          2. Diff Patch to Apply:
            {diff}

          ======================================================================
          3. NEW FILE DETECTION:
            - If original is empty or contains "New file", this is a NEW FILE creation
            - For new files, extract content directly from the diff by taking all lines that start with "+" and removing the leading "+"
            - DO NOT return an error about missing original content for new files

          ======================================================================
          4. Preservation Rules:
            - For existing files:
              - Keep original code structure intact.
              - Apply all modifications from the diff.
              - Maintain existing imports, comments, and docstrings.
              - Preserve original functionality.
              - Keep original whitespace patterns.
              - Maintain exact method ordering.
            - For new files:
              - Extract the complete content from lines starting with "+" in the diff
              - Remove the leading "+" character from each line
              - Return the complete clean content as the file content

          CRITICAL OUTPUT INSTRUCTIONS:
            - Return ONLY the complete final file content.
            - NO file boundary markers.
            - NO end-of-file comments.
            - NO markdown formatting.
            - NO documentation separators.
            - NO conversation text or explanations.
            - Output must be valid code that could be directly saved to a file.

          Any non-code content will break the processing pipeline.

    coder:
      provider: "anthropic"
      model: "claude-3-opus-20240229"
      temperature: 0
      prompts:
        system: |
          You are an expert code modification agent. Your task is to safely and precisely apply code changes.
          You receive changes in this exact JSON structure:
          {
            "changes": [
              {
                "file_path": "exact path to file",
                "type": "modify",
                "description": "change description",
                "content": "complete file content"
              }
            ]
          }

          Rules:
          1. Always expect input in the above JSON format
          2. If input is a string, parse it as JSON first
          3. Preserve existing functionality unless explicitly told to change it
          4. Maintain code style and formatting
          5. Apply changes exactly as specified
          6. Handle errors gracefully with backups
          7. Validate code after changes


python3 /Users/jim/src/apps/c4h/c4h_agents/skills/tartxt.py  -f changed_files.txt $(git status --porcelain | awk '{print $2}')

python3 /Users/jim/src/apps/c4h/c4h_agents/skills/tartxt.py -o backend/models/job.py backend/services/job_repository.py c4h-micro/packages/job-management/src/contexts/JobContext.tsx c4h-micro/packages/shared/src/services/apiService.ts c4h-micro/packages/shared/src/types/job.ts




python ./c4h_services/src/bootstrap/prefect_runner.py  jobs -P 5500 \
--config /Users/jim/src/apps/c4h_projects/self_improvement/20250402_03_MVP_JobSubmission_back.yml \
--stage coder --lineage-file workspaces/lineage/20250408/wf_1303_aeaea888-d4d3-436e-9183-37702cea13b5/events/130546_5de4ff87-a5e6-47ff-b34f-1727d7b2e435.json



python ./c4h_services/src/bootstrap/prefect_runner.py  jobs -P 5500 \
--config /Users/jim/src/apps/c4h_projects/self_improvement/20250402_03_MVP_JobSubmission_back.yml \
--stage coder --lineage-file workspaces/lineage/20250409/wf_0242_9ac05a34-cb5f-4ee2-8b7a-6f8e305ef1ce/events/024413_e9b29496-e8f5-43cf-99b7-29f1aabea904.json


tests/setup/setup_test_projects.sh  && \
python ./c4h_services/src/bootstrap/prefect_runner.py  workflow \
--config tests/examples/config/0228_02_applylogging.yml \
2>&1 | tee output.txt

tests/setup/setup_test_projects.sh  && \
python ./c4h_services/src/bootstrap/prefect_runner.py apply_diff \
    --project-path "./test/test_projects" \
    --diff-file tests/setup/apply_diff.yml \
    --config tests/examples/config/0228_02_applylogging.yml \
    --poll

export PYTHONPATH="/Users/jim/src/apps/c4h" && \
tests/setup/setup_test_projects.sh  && \ 
python ./c4h_services/src/bootstrap/prefect_runner.py apply_diff \
    --project-path "." \
    --diff-file tests/setup/jobs_coder_01_actual.diff \
    --config tests/setup/apply_config.yml \
    --poll



python /Users/jim/src/apps/c4h/c4h_services/src/bootstrap/prefect_runner.py apply_diff \
    --project-path "." \
    --diff-file tests/setup/jobs_coder_01_actual.diff \
    --config tests/setup/apply_config.yml \
    --poll


prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250411_00_fixjobs.diff \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --poll