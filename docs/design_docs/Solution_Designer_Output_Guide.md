# Solution Designer OpenLineage JSON Output Guide

## Output Requirements

As a solution designer agent, you must produce output following the **exact OpenLineage JSON structure** required by the coder agent. Your solution must be formatted as JSON within the `llm_output.content` field of the OpenLineage event.

## OpenLineage JSON Structure

The lineage file must match this exact structure:

```json
{
  "event_id": "44889ed0-90a4-4405-9aad-0cc779b6018c",
  "timestamp": "2025-03-30T12:55:33.139864+00:00",
  "agent": {
    "name": "solution_designer",
    "type": "solution_designer"
  },
  "workflow": {
    "run_id": "wf_2350_11c00ca0-7c98-4a7a-be14-c8e6245e967a",
    "parent_id": null,
    "step": null,
    "execution_path": [
      "solution_designer:44889ed0"
    ]
  },
  "llm_input": {
    // Input data (will be populated automatically)
  },
  "llm_output": {
    "content": "YOUR SOLUTION DESIGN JSON STRING GOES HERE - MUST BE PROPERLY ESCAPED",
    "finish_reason": "length",
    "model": "claude-3-7-sonnet-20250219",
    "usage": {
      "prompt_tokens": 62695,
      "completion_tokens": 4096,
      "total_tokens": 66791
    }
  },
  "metrics": {
    "token_usage": {
      "prompt_tokens": 62695,
      "completion_tokens": 4096,
      "total_tokens": 66791
    }
  },
  "error": null
}
```

## Solution Design in `llm_output.content`

Your solution design must be a properly escaped JSON string placed in the `llm_output.content` field. It should follow this structure:

```json
{
  "modification_plan": {
    "intent_analysis": "Analysis of the intent and what needs to be modified",
    "files_to_modify": [
      "path/to/file1.ext",
      "path/to/file2.ext"
    ],
    "detailed_changes": [
      {
        "file_path": "path/to/file1.ext",
        "description": "Description of changes needed for file1",
        "modifications": [
          {
            "original": "Code to remove",
            "modified": "Code to add"
          }
        ]
      }
    ],
    "summary": "Brief overview of all changes and how they fulfill the intent"
  }
}
```

## Critical Requirements

1. **Nested JSON**: Your solution must be a valid JSON string inside the `llm_output.content` field of the outer JSON structure.

2. **Properly Escaped**: All quotes, newlines, and special characters must be properly escaped according to JSON standards.

3. **Exact Structure**: Both the outer OpenLineage structure and inner solution design structure must be followed exactly.

4. **Field Requirements**:
   - `event_id`: A UUID for the event
   - `timestamp`: ISO format timestamp
   - `workflow.run_id`: The workflow run ID
   - All other fields shown in the template must be included

## Example

Here's a simplified example showing the required format (with abbreviated content for clarity):

```json
{
  "event_id": "44889ed0-90a4-4405-9aad-0cc779b6018c",
  "timestamp": "2025-03-30T12:55:33.139864+00:00",
  "agent": {
    "name": "solution_designer",
    "type": "solution_designer"
  },
  "workflow": {
    "run_id": "wf_2350_11c00ca0-7c98-4a7a-be14-c8e6245e967a",
    "parent_id": null,
    "step": null,
    "execution_path": [
      "solution_designer:44889ed0"
    ]
  },
  "llm_input": {},
  "llm_output": {
    "content": "{\"modification_plan\":{\"intent_analysis\":\"The intent is to add proper error handling to API endpoints.\",\"files_to_modify\":[\"src/controllers/userController.js\"],\"detailed_changes\":[{\"file_path\":\"src/controllers/userController.js\",\"description\":\"Add try/catch blocks\",\"modifications\":[{\"original\":\"async function getUser(req, res) {\\n  const user = await User.findById(req.params.id);\\n  res.json(user);\\n}\",\"modified\":\"async function getUser(req, res, next) {\\n  try {\\n    const user = await User.findById(req.params.id);\\n    if (!user) {\\n      return res.status(404).json({ message: 'User not found' });\\n    }\\n    res.json(user);\\n  } catch (error) {\\n    next(error);\\n  }\\n}\"}]}],\"summary\":\"Added consistent error handling\"}}",
    "finish_reason": "length",
    "model": "claude-3-7-sonnet-20250219",
    "usage": {
      "prompt_tokens": 62695,
      "completion_tokens": 4096,
      "total_tokens": 66791
    }
  },
  "metrics": {
    "token_usage": {
      "prompt_tokens": 62695,
      "completion_tokens": 4096,
      "total_tokens": 66791
    }
  },
  "error": null
}
```

## Command Usage

Your JSON output will be used directly in a command like:

```
python ./c4h_services/src/bootstrap/prefect_runner.py workflow --config \
/Users/jim/src/apps/c4h_projects/self_improvement/20250328_06_AddJobsAPI_c4h.yml \
--stage coder \
--lineage-file llm_provided_soln_design.json
```

The system expects to read a valid OpenLineage JSON file with your solution design properly encoded as a string within the `llm_output.content` field. Any deviation from this format will cause the workflow to fail.

## Important Note

The most challenging aspect is ensuring your solution design JSON is properly escaped within the outer JSON structure. All newlines must be `\n`, all quotes must be `\"`, and other special characters must be properly escaped according to JSON standards.