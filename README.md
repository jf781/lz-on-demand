# Landing Zone on Demand

## Variables and secrets to define 

| Name | Type | Description | Specified In |
| ---- | ---- | ----------- | ------------ | 
| GH_ORG | Repo Variable | Used to determine the correct GitHub Org | Core Repo |
| GH_TOKEN | Repo Secret | Used to authenticate to GitHub to create repos/actions/etc... | Core Repo |
| TFC_ORG_NAME | Repo Variable | Used to determine the correct Terraform Cloud Org | Core Repo |
| TFC_TOKEN | Repo Secret | Used to authenticate with Terraform Cloud | Core Repo |
| TFC_AZURE_VAR_SET_NAME | Repo Variable | Used to identify the Variable Set in TFC to use to authenticate with Azure | Core Repo |
| TFC_GH_TOKEN_ID | Repo Secret | Used to identify which token TFC should use to authenticate with GitHub | Core Repo |
| ARM_SUBSCRIPTION_ID | Repo Secret | Used to specify the Azure subscription to authenticate with | ARM | Core Repo |


## Rest Call Inputs

| Name | Type | Description | Specified In | Example | 
| ---- | ---- | ----------- | ------------ | ------- |
| SUBSCRIPTION_NAME | `String` | Used to specify the name of the Azure subscription to authenticate with | trigger.yml | `subscription123` |
| TEMPLATE_REPO_URL | `String` | Used to specify the URL of the GitHub repo that contains the template Terraform code for a new Landing Zone | trigger.yml | `https://github.com/jf781/lz-infra-bootstrap-repo` |
| PARENT_MGMT_GRP_NAME | `String` | Used to specify the name of the Azure Management Group that will be the parent of the new Landing Zone | trigger.yml | `mg-org-root` | 
| MGMT_GRP_POLICIES | `JSON` | Used to specify the name of the existing Azure Policies that will be assigned to the management group that will be created | trigger.yml | `{ "policy1" = "/providers/Microsoft.Authorization/policyDefinition/policyDefinitionId" }` |
| TAGS | `JSON` | Used to specify the tags that will be applied to the new Landing Zone. The values of the tags will be used to define the names of the resources created.  Please see notes below regarding Tag Inputs | trigger.yml | `{ "environment" = "value", "region" = "value", "status" = "value", "business_criticality" = "value", "data_classification" = "value", "application" = "value", "responsible_group_manager" = "value", "responsible_group_org_name" = "value", "deployed_by" = "value", "backup" = "value", "value_stream" = "value", "responsible_group_org_dl"  = "value", "deployment_date" = "1/1/2000", "shc_app_usage" = "value", "iac_repo" = "value", "cost_center" = "value" }` |
| VNET | `JSON` | Used to specify the details of the Vnet and Vnet peering of the new Landing Zone | trigger.yml | `{"addressSpace": "10.10.0.0/16", "hubVnetName": "hub-vnet", "hubVnetRgName": "hub-rg", "firewallIp": "10.0.0.1"`|
| RBAC | `JSON` | Used to specify the details of the RBAC that will be applied to the new subscription or management group | trigger.yml | `{"azureAdGroupName": "AADGroupName", "rbacRoleName": "Contributor", "rbacScopeType": "management_group", "rbacRoleType": "builtin"}` |
| BUDGET | `JSON` | Used to specify the details of the budget that will be applied to the new subscription | trigger.yml | `"{"emailContacts": "bob@bob.com", "frank@frank.com", "budgetAmount": "1000"}",` |
| SUBNETS | `JSON` | Used to specify the details of the subnets that will be created in the new Landing Zone | trigger.yml | `[{"subnet_name":"subnet-01","prefix":"10.10.10.0/24","purpose":"Test","nsg_map":[{"name":"rule01","priority":101,"direction":"Inbound","access":"Allow","protocol":"*","source_port_range":"*","destination_port_ranges":["80","443","3389","22"],"destination_port_range":"","source_address_prefix":"*","destination_address_prefix":"*"}]}]` |


### Rest Call Notes

When calling the GitHub Action via REST API, you will have to format the JSON inputs as a string. You can use the following tool to help with this: https://www.freeformatter.com/json-escape.html#before-output.  You will also need to escape the double quotes in the JSON string.  For example, if you have the following JSON:

```json
{
  "addressSpace": "10.10.0.0/16",
  "hubVnetName": "hub-vnet",
  "hubVnetRgName": "hub-rg",
  "firewallIp": "10.0.0.1"
}
```

When you pass it the GitHub Action, you will need to format it as a string and escape the double quotes:

```bash
"{\"addressSpace\": \"10.10.0.0/16\", \"hubVnetName\": \"hub-vnet\", \"hubVnetRgName\": \"hub-rg\", \"firewallIp\": \"10.0.0.1\"}",
```

The Free Formatter tool will help with this.  If you paste the formatted JSON input into the tool and select `Escape JSON` it will give you and output you can copy.  

#### Tags Input Notes

There are some examples where the use of `\r\n` are used to help with the formatting of the JSON objects as they are written to the `workspace.auto.tfvars` file (looking at the subnet example in the `example-inputs.json` file).  You cannot do this TAGS input.  There cannot be any extra spaces or extra characters.  It will prevent the tag values from being read when using it to define runtime variables.  

## Example commands

### List GitHub Actions in a repository

```bash
pat='--- Enter your GitHub Personal Access Token ---'
org='--- Enter your GitHub Org name ---'
repo='--- Enter your GitHub Repo that contains the actions ---'

curl --request GET "https://api.github.com/repos/$org/$repo/actions/workflows" \
--header "X-GitHub-Api-Version: 2022-11-28" \
--header "Accept: application/vnd.github+json" \
--header "Authorization: Bearer $pat" | jq '.workspaces'
```

### Call GitHub Action

```bash
pat='--- Enter your GitHub Personal Access Token ---'
org='--- Enter your GitHub Org name ---'
repo='--- Enter your GitHub Repo that contains the actions ---'
workflowId=`--- ID of the GitHub action you are calling ---`

curl -X POST "https://api.github.com/repos/$org/$repo/actions/workflows/$workflowId/dispatches" \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer $pat"\
  -H "X-GitHub-Api-Version: 2022-11-28" \
  -d '{"ref":"main", "inputs": { "REGION":"CentralUS", "LANDING_ZONE_NAME":"exp-lz-repo-01", "TAGS":"{\"costcenter\": \"1234\", \"businessunit\": \"Engineering\", \"dayofweek\": \"Tuesday\"}", "TEMPLATE_REPO_URL":"https://github.com/jf781/lz-infra-bootstrap-repo" }}'
 ```


