# Intro
This template provides a complete solution for regularly exporting an inventory of all Entra ID Enterprise Applications to Log Analytics. This inventory can then be used in Sentinel, especially in Workbooks and, if necessary, in Analytics Rules.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FJLangthaler%2FSentinel-App-Registratons-Inventory%2Frefs%2Fheads%2Fmain%2FTemplate.json)

# Components
## Data Collection Rule (DCR)
The Data Collection Rule is necessary to send data via API to Log Analytics. This is a requirement from Microsoft.

## Data Collection Endpoint (DCE)
Just like with the DCR, a Data Collection Endpoint is required for the DCR. With this, you get a dedicated endpoint to send data to.

## Custom Log Analytics Table
Since the data does not have a structure that default tables allow, a custom table must be created. The template contains a schema that matches the output of the queried data.

## Logic App
The Logic App is responsible for sending the Enterprise App data from Entra ID to the Log Ingestion API. The other components are necessary for ingesting the data.

## Deployment Script
The deployment script is only used to introduce a delay so that the Managed Identity of the Logic App can be authorized. This is because it is only created after the Logic App, which can cause problems if you do not wait.

## Role Assignment
In the final step, the Managed Identity of the Logic App is given the `Monitoring Metrics Publisher` role on the created DCR. This is necessary so that the Logic Apps can send the data to the Log Analytics API.

# Prerequisites
1. Azure Subscription
2. Resource Group
3. Log Analytics Workspace

# Important Notes
This template **MUST** be deployed to the resource group where the Log Analytics Workspace to be used is located.
This is because the template, among other things, creates a new custom table in the Log Analytics Workspace.
However, when deploying templates, the RG specified in the wizard is always used for all paths. Therefore, there is a reference problem here. The LAW must therefore be in the same RG.
The Logic Apps, DCE & DCR can be moved to another RG afterwards without any problems.

# Deployment
1. Azure Portal --> `Deploy a custom template` --> `Build your own template in the editor`
2. Upload or copy-paste Template.json
3. Fill in parameters, paying attention to the following:
    1. LAW must be in the specified RG
    2. Custom Table may only contain letters
    3. All resource names should be distinguishable from the second, very similar solution for App Registrations. This is especially important for the Custom Table Name
    4. The parameter `UTC Value` must not be changed!
4. After successful deployment, switch to the `Outputs` tab and copy the `logicAppManagedIdentityObjectId` displayed here; this will be needed in the next step.

# Post-Deployment
After deployment, only the system-assigned Managed Identity of the created Logic App needs to be authorized and the Logic App activated.

## Authorize Managed Identity
1. Use [`Add-ManagedIdentityGraphPermissions.ps1`](https://github.com/JLangthaler/Add-Managed-Identity-Graph-Permissions-PowerShell)
3. The required Graph Permission is `Application.Read.All`, which must be entered in the script
4. Run the script in Azure CLI

It may take about 10 minutes for the permission to take effect. If the Logic App is activated and started immediately after granting, a 400 (Forbidden) error may occur in the last step. In that case, just wait.

## Activate Logic App
Finally, the Logic App only needs to be activated.
