# Bicep Deployment Workflow
The following workflow has been created to deploy bicep stacks to the Azure Platform. Form of authentication used for connection is Service Pricipal, this will be addressed in the [Authenticating To Azure](#authenticating-to-azure) section. File can be found in the following directory: `.github/workflows/bicep-deploy.yml`.

## Assumptions
Knowledge in the following:
- Github Actions Workflows
- Azure CLI
- Bicep

### Calling this Workflow
Sample code to call this workflow is the following.
```YAML
  Deploy:
    uses: scankin/bicep-workflows/.github/workflows/bicep-deploy.yml@main
    with:
      location: uksouth
      deployment_file: /main.bicep
      resource_group_name: rg-sample-dev-uks-01
      deployment_name: sample-stack
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
```

### Inputs
The following inputs are designed to be as unperscriptive as possible. Allowing the user to choose their own naming convention for resource group, and decide their own naming convention for the deployment stack name.

| Name | Type | Required | Description | 
| ---- | ---- | ---------| ----------- |
| location | `string` | Y | The azure region you  wish to deploy to i.e. `uksouth` |
| deployment_file | `string` | Y |The relative path of the `.bicep` deployment file in your repository. |
| resource_group_name | `string` | Y | The name of the resource group to deploy your resources to |
| deployment_name | `string` | Y | The deployment name for the deployment stack |

### Secrets
| Name | Required | Description | 
| ----  | ---------| ----------- |
| AZURE_CREDENTIALS | Y | The Credentials of your Service Principal. [See Authenticating To Azure](#authenticating-to-azure) for more information.|


### Jobs
#### Deploy
Job which is composed of all steps to deploy `.bicep` deployment file.
##### Checkout Repository
Checks out the working in repository.
##### Install AzCLI
Installs AzCLI using curl [see here](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?view=azure-cli-latest&pivots=apt#option-1-install-with-one-command).
##### Authenticate
Uses the `azure/login@v2` step to authenticate to Azure using a Service Principal. [See Authenticating To Azure](#authenticating-to-azure)
##### Check Resource Group
Checks to see if the resource group supplied exists within the allowed scope of the Service Principal. If it does not exits, an error is thrown.
##### Deploy Bicep
This task deploys the specified `.bicep` deployment file to the supplied `resource_group`. Action on unmanage is set to `detatchAll` to reduce unwanted changes. Deny settings mode is set to None.

### Authenticating To Azure
The pipeline authenticates using the `azure/login@v2` step using `SERVICE_PRINCIPAL` authentication method. This requires a repository secret to hold the authentication details for the service principal named `AZURE_CREDENTIALS` which is defined in the following format.
```JSON
{
    "clientId": "<Client ID>",
    "clientSecret": "<Client Secret>",
    "subscriptionId": "<Subscription ID>",
    "tenantId": "<Tenant ID>"
}
```
For further information, please see the [Microsoft Guidance](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-secret) around configuration.

## Resources
- [Authenticating Workflows to Azure Using a Service Principal](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-secret)