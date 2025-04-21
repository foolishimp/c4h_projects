
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


###
export PYTHONPATH="/Users/jim/src/apps/c4h" && \
tests/setup/setup_test_projects.sh  && \ 
python ./c4h_services/src/bootstrap/prefect_runner.py apply_diff \
    --project-path "." \
    --diff-file tests/setup/jobs_coder_01_actual.diff \
    --config tests/setup/apply_config.yml \
    --poll


###
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


  prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250411_01_fixjobs_backend.diff \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --poll  

prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250411_02_fixjobs_datetime.diff \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --poll  



prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250411_03_fixjobs_config.diff

prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250411_04_add_widgets.diff

prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250411_05_add_midwindow.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" &&
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h_editor" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250412_01_fixdebug.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" &&                                                      
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250412_03_configmerge.diff


export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250412_04_configmerge.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250412_05_fixsemantic_iterator.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250415_00_UI_modifieddate.diff


export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250416_00_UI_sortpiclist.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/general_diff_runner.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file     /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250417_wo001_dynamicportal.diff

## Config Managment
tartxt.py \
   /Users/jim/src/apps/c4h_editor/shell_service \
  -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
  -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_shell_service.txt

workorder:
  project:
    path: /Users/jim/src/apps/c4h/tests/test_projects/
    workspace_root: workspaces
  intent:
    description: |
      Add logging to all functions with lineage tracking:
      - Replace print statements with logging
      - Add logging configuration
      - Enable lineage tracking for observability

team:
  llm_config:
    agents:
      discovery:
        temperature: 0
        tartxt_config:
          script_path: /Users/jim/src/apps/c4h/c4h_agents/skills/tartxt.py
          input_paths:
            - packages/job-management/src/
            - packages/config-selector/src/
            - packages/shared/src/
            - /Users/jim/src/apps/c4h_projects/docs/design_docs/Typescript_Design_Principles.md
            - /Users/jim/src/apps/c4h_projects/docs/design_docs/c4h_api_doc_v1.md
            - /Users/jim/src/apps/c4h_projects/docs/design_docs/Jobs_Design_Guide_0402.md
          exclusions:
            - '**/node_modules/**'
            - '**/dist/**'
            - '**/*.css'
      solution_designer:
        provider: anthropic
        model: claude-3-7-sonnet-20250219



tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
   /Users/jim/src/apps/c4h/c4h_services \
  /Users/jim/src/apps/c4h/config \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**/*.toml,**/*.md" \
  -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_full.txt

tartxt.py \
  /Users/jim/src/apps/c4h_editor/c4h-micro/packages \
  /Users/jim/src/apps/c4h_editor/backend/ \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**/*.toml,**/*.md" \
 -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_editor_full.txt

tartxt.py \
    /Users/jim/src/apps/c4h_projects/docs/design_docs/ \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
 -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_design_master_doc.txt


tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shell \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shared \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
 -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_editor_front_shell.txt


export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250417_wo003_dynamicportal.diff


export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250417_wo009_dynamicportal.diff


export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/general_diff_runner.diff

export PYTHONPATH="/Users/jim/src/apps/c4h" && \
prefect_runner.py apply_diff \
    --project-path "/Users/jim/src/apps/c4h" \
    --config /Users/jim/src/apps/c4h_projects/self_improvement/apply_config.yml \
    --diff-file /Users/jim/src/apps/c4h_projects/self_improvement/diffs/20250421_00_MF_refactor3.diff


tartxt.py \
  /Users/jim/src/apps/c4h_editor_aidev/c4h-micro/packages \
    -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_editor_micro.txt


 tartxt.py \
  /Users/jim/src/apps/c4h_editor_aidev/shell_service \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**/*.toml,**/*.md" \
 -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_shell_servicex_aidev.txt

tartxt.py \
  test-app \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**,**/*.toml,**/*.md" \
 -f /Users/jim/src/apps/c4h_projects/backup_txt/c4h_testapp.txt