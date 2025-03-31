tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shell \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-selector \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/yaml-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shared \
   /Users/jim/src/apps/c4h_editor/backend \
  -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
  -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt

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
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
 -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_full.txt

tartxt.py \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shell \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-selector \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/config-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/yaml-editor \
   /Users/jim/src/apps/c4h_editor/c4h-micro/packages/shared \
  -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
  -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_micro.txt

tartxt.py \
  /Users/jim/src/apps/c4h_editor/backend/ \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**" \
 -f /Users/jim/src/apps/c4h_editor/backup_txt/c4h_editor_backend.txt


  uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000

  look at the prompt template

I want you to give me a yaml prompt based upon the template that will specifically fix this problem

I want you provide sufficient context such that the llm receiving this will be able to efficiently make the fixes, consider the design documents when creating the prompt




tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
   /Users/jim/src/apps/c4h/c4h_services \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_full.txt


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

### c4h agents
tartxt.py \
   /Users/jim/src/apps/c4h/c4h_agents \
 -x "**/__pycache__/**,**/.git/**,**/*.pyc,**/node_modules/**,**/package-lock.json,**/dist/**,**/.DS_Store,**/README.md,**/workspaces/**" \
  -f /Users/jim/src/apps/c4h/backup_txt/c4h_agents.txt

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


## Test case with load lineage

python ./c4h_services/src/bootstrap/prefect_runner.py  workflow \
--config tests/examples/config/0228_02_applylogging.yml \
--stage coder \
--lineage-file /Users/jim/src/apps/c4h/workspaces/lineage/20250331/wf_1531_58e009c7-2556-4596-89d9-d92560890c81/events/781778da-6773-4e21-9338-e3b08ac12ecc.json \
2>&1 | tee output.txt
