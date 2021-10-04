# jrap-AzureVerticalScaling
This project contains a variety of runbooks to autoscale various resources VERTICALLY. _It is not recommended to install this in a production environment. The primary targets for this repository are non-prod environments._ For example, if you have your Staging or Dev infrastructure set up as a mirror to your production infrastructure, you're paying a **premium price** for an environment that does not have the same 100% up-time requirement. By turning off resources during non-work hours you can realistically cut your Azure costs in half+.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjoanvianadev%2Fjrap-AzureVerticalScaling%2Fmaster%2Fazuredeploy.json)

## Getting Started
Use the ***Deploy to Azure*** button above to quickly deploy these scripts to your Azure Subscription. The deployment will take a few minutes. The longest piece is the import of dependent Azure Az modules in the runbook gallery. The deployment will also automatically add several "Tutorial" runbooks, even though they are excluded from the ARM template. Feel free to delete them post-deployment (AzureAutomationTutorial, AzureAutomationTutorialPython2, AzureAutomationTutorialScript).

Once deployed, you will see the following resources:
- \<Your Resource Group\>/\<Your Automation Account\>/ScaleAppServicePlansDown
- \<Your Resource Group\>/\<Your Automation Account\>/ScaleAppServicePlansUp

Next, add a Run As account...

**Configure the Run As Account**

(Note: It is not possible to create the Run As Account with an ARM template: https://feedback.azure.com/forums/246290-automation/suggestions/18864241-creation-of-run-as-accounts-using-arm-template)

1. Select your \<Your Automation Account\> resource
2. Select Account Settings > Run as accounts from the left hand navigation
3. Select "Create" on the "Azure Run As Account"
4. Create
5. The name will be "AzureRunAsConnection". (This value is defaulted in the scripts.)

More information regarding Run As accounts: https://docs.microsoft.com/en-us/azure/automation/manage-runas-account

Next, add a schedule to your runbooks to reduce your expenses when you're not using them.

**Adding a Schedule**

1. Select your \<Your Automation Account\> resource
2. Select Shared Resources > Schedules from the left hand navigation
3. Select "+ Add a schedule"
    1. Enter name, e.g. "Scale Down at night"
	2. Enter a description (optional)
	3. Select a Start time (must be at least 5 minutes from now)
	    1. The _time_ is the most important. Choose a time such as 5:00 PM or whenever you are confident the environments are not needed
	4. Choose Recurrence > Recurring
	    1. Recur every 1 Week
		2. On these days: Monday, Tuesday, Wednesday, Thursday, Friday
	5. Set expiration: No
	6. Create
4. Once the deployment of your schedule is complete, select your runbook(s)
    1. \<Your Resource Group\>/\<Your Automation Account\>/ScaleAppServicePlansDown, for example
5. Select Link to schedule
    1. In the new blade that opens, Link a schedule to your runbook
	    1. Select your "Scale Down at night" schedule, for example
	2. Choose your parameters (see the Parameters section below)
	3. OK
	
_If you set the ScaleAppServicePlansDown to run every M-F at 5pm, you should set the ScaleAppServicePlansUp to run every M-F at 8am, for example._


	
## Parameters

### Scale App Service Plans Down/Up

Parameter  | Usage
------------- | -------------
resourceGroupName  | If blank, all available Resource Groups will be iterated. If supplied, only App Service Plans within the specified Resource Group will be iterated.
appServicePlanName  | If blank, all available App Service Plans in the Resource Group(s). If supplied, the runbook will only target this specific App Service Plan and ignore all others.

## Manually Executing Runbooks

You may wish to manually execute a runbook to ensure it performs as expected.

1. Select your <Your Automation Account> resource
2. Select Process Automation > Runbooks in the left hand navigation
3. Select the runbook you wish to execute
4. Select Start
5. Optionally, enter the parameters: RESOURCEGROUPNAME, APPSERVICEPLANNAME

## Edit/Test Runbooks

1. Select your <Your Automation Account> resource
2. Select Process Automation > Runbooks in the left hand navigation
3. Select the runbook
4. Select Edit
5. Make modifications
6. Select Test pane
7. Select Start
8. Optionally, enter the parameters: RESOURCEGROUPNAME, APPSERVICEPLANNAME 
9. When satisfied, Select "Edit PowerShell Runbook" from the breadcrumbs
10. Select Save

***Do not forget to publish your runbook or any linked schedules will not execute your most recent code!***

# How it works

## Scaling App Service Plans Up/Down

The vertical scaling of App Service Plans mirrors any manual steps that would be required to scale a resource up or down. During the scaling down process, all resources have their current Tier and Size saved to Automation Variables in the format: RESOURCEGROUPNAME.RESOURCENAME.\<PROPERTY\>. This allows the scaling up runbook to know what to set the resources back to once executed.

**Tier-Specific Settings for Individual App Services**

If you are converting from a Production or Standard Tier to a free tier, some settings need to be saved and modified before converting. These runbooks account for these unique settings by iterating over all App Services within an App Service Plan and storing the previous value. When the sites are re-scaled back up, they are properly set back to their previously defined values. 

The following settings must be set as defined below in order to convert to the Free tier:

***AlwaysOn***

"Always on" must be set to `False`.

***Use32BitWorkerProcess***

"Use32BitWorkerProcess" must be set to `True`.

***ClientCertEnabled***

"ClientCertEnabled" must be set to `False`.

_Note: There may be additional settings that do not descend properly to a Free tier. If so, the same strategy may be used to store the value, then on upscaling, retrieve the value and set it back to its original value. If any of these settings cannot be applied confidently to your solution, you may need to modify the scripts._

# What is deployed

The one-click deployment will install the following resources. You can view the details in azuredeploy.json in the root of the repo.

- Automation Account
   - Run books
      - Scale App Service Plans Down
	  - Scale App Service Plans Up
	  - Utility-DeleteAllVariables (included to assist with testing, it removes all variables in the Automation Account, **use with care**)
   - Modules (dependencies on scripts)
      - Az.Accounts 
      - Az.Resources 
      - Az.Websites 
      - Az.Automation 
      - Az.Sql 
      - Az.Monitor 


