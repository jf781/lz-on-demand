# REST API Call Reference
This is a list of REST API calls I used and found helpful while creating the associated GitHub Actions

## GitHub REST API Calls

### Define GitHub Variables
```bash
pat='--- Enter your GitHub Personal Access Token ---'
org='--- Enter your GitHub Org name ---'
repo='--- Enter your GitHub Repo that contains the actions ---'
```

### Get workflow ID
```bash
curl --request GET "https://api.github.com/repos/$org/$repo/actions/workflows" \
--header "X-GitHub-Api-Version: 2022-11-28" \
--header "Accept: application/vnd.github+json" \
--header "Authorization: Bearer $pat" | jq '.workspaces[]'
```

### Define Workflow ID
```bash
workflowId=`--- ID of the GitHub action you are calling ---`
```

### Get GitHub Import Status
```bash
curl \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $pat"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/$org/$repo/import"
```

### Initiate GitHub ActionWorkflow (PoC Workflow)
```bash
curl -X POST "https://api.github.com/repos/$org/$repo/actions/workflows/$workflowId/dispatches" \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $pat"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  -d '{"ref":"main", "inputs": { "REGION":"CentralUS", "LANDING_ZONE_NAME":"exp-lz-repo-03", "TAGS":"{\"costcenter\": \"1234\", \"businessunit\": \"Engineering\", \"dayofweek\": \"Tuesday\"}", "TEMPLATE_REPO_URL":"https://github.com/jf781/lz-infra-bootstrap-repo" }}'
```

### Initiate GitHub ActionWorkflow using a parameter file (PoC Workflow)
```bash
curl -X POST "https://api.github.com/repos/$org/$repo/actions/workflows/$workflowId/dispatches" \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $pat"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  -d @example-input.json
```

### Create Repo
```bash
  curl -X POST "https://api.github.com/orgs/$org/repos" \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $pat"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  -d '{"name":"Hello-World-TEST","description":"This is your first repository","homepage":"https://github.com","private":true,"has_issues":true,"has_projects":true,"has_wiki":true}'
```

### Test if Repo Exists
```bash
repoTestOutput=$(curl -X GET "https://api.github.com/repos/$org/$repo" \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $pat"\
  -H "X-GitHub-Api-Version: 2022-11-28" )
repoExists=$( echo $repoTestOutput | jq '.id' -r )
if [ "$repoExists" != 'null' ]; then 
  repo_exists="true"
else
  repo_exists="false"
fi 
```

## Terraform Cloud REST API Calls

### Define TFC Variables
```bash
tfpat='--- Specify your Terraform Cloud Personal Access Token ---'
org='--- Enter your Terraform Cloud Org name ---'
workspace='--- Enter your Terraform Cloud Workspace ---'
```

### Get Variable Sets
```bash
curl -X GET "https://app.terraform.io/api/v2/organizations/$org/varsets" \
  --header "Authorization: Bearer $tfpat" \
  --header "Content-Type: application/vnd.api+json" 
```

### Get Workspace ID
```bash
workspaces=$( curl -X GET "https://app.terraform.io/api/v2/organizations/$org/workspaces" \
  --header "Authorization: Bearer $tfpat" \
  --header "Content-Type: application/vnd.api+json" )
workspaceId=$( echo $workspaces | jq '.data[] | select ( .attributes.name == "$workspace" ) | .id ' -r
```

### Assign Variable set to Workspace
```bash
varSetId='--- Specify Variable Set ID ---'
workspaceId='--- Specify Workspace ID ---'

curl -X POST "https://app.terraform.io/api/v2/varsets/$varSetId/relationships/workspaces" \
  --header "Authorization: Bearer $tfpat" \
  --header "Content-Type: application/vnd.api+json" \
  --data-raw '{"data": [{"type": "workspaces", "id": "$workspaceId"}]}'
```

### Associate Terraform Cloud Workspace with GitHub Repo
```bash
repo='--- GitHub Repo Name.  Must be in Org/Repo format.  For example: jf781/sandbox ----'
repoBranch='--- The branch of the repo that you want to associate with the workspace ---'
oauthTokenId='--- This is specific the GitHub Org.  It can be found in the VCS settings of the TFC org ---'

curl -X PATCH "https://app.terraform.io/api/v2/organizations/$org/workspaces/$repo" \
  --header "Authorization: Bearer $tfpat" \
  --header "Content-Type: application/vnd.api+json" \
  --data-raw '{"data":{"attributes":{"vcs-repo":{"identifier":"$repo","oauth-token-id":"$oauthTokenId","branch":"$repoBranch"}}}}'
```