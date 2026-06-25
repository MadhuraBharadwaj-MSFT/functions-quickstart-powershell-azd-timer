<!--
---
name: Azure Functions PowerShell Timer Trigger using Azure Developer CLI
description: This repository contains an Azure Functions timer trigger quickstart written in PowerShell and deployed to Azure Functions Flex Consumption using the Azure Developer CLI (azd). The sample uses managed identity and a virtual network to make sure deployment is secure by default.
page_type: sample
products:
- azure-functions
- azure
- entra-id
urlFragment: starter-timer-trigger-powershell
languages:
- powershell
- bicep
- azdeveloper
---
-->

# Azure Functions PowerShell Timer Trigger using Azure Developer CLI

This template repository contains a timer trigger reference sample for functions written in PowerShell and deployed to Azure using the Azure Developer CLI (`azd`). The sample uses managed identity and a virtual network to make sure deployment is secure by default. You can opt out of a VNet being used in the sample by setting VNET_ENABLED to false in the parameters.

This project is designed to run on your local computer. You can also use GitHub Codespaces:

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/azure-samples/functions-quickstart-powershell-azd-timer)

This codespace is already configured with the required tools to complete this tutorial using either `azd` or Visual Studio Code. If you're working in a codespace, skip down to [Run your app section](#run-your-app-from-the-terminal).

## Common Use Cases for Timer Triggers

- **Regular data processing**: Schedule batch processing jobs to run at specific intervals
- **Maintenance tasks**: Perform periodic cleanup or maintenance operations on your data
- **Scheduled notifications**: Send automated reports or alerts on a fixed schedule
- **Integration polling**: Regularly check for updates in external systems that don't support push notifications

## Prerequisites

- [Azure Storage Emulator (Azurite)](https://learn.microsoft.com/azure/storage/common/storage-use-azurite) - Required for local development with Azure Functions. Install via the VS Code [Azurite extension](https://marketplace.visualstudio.com/items?itemName=Azurite.azurite), `npm install -g azurite`, or run the [Docker image](https://hub.docker.com/_/microsoft-azure-storage-azurite)
- [PowerShell 7.4](https://learn.microsoft.com/powershell/scripting/install/installing-powershell)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local?pivots=programming-language-powershell#install-the-azure-functions-core-tools)
- [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)
- To use Visual Studio Code to run and debug locally:
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
  - [PowerShell extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.powershell)

## Initialize the local project

You can initialize a project from this `azd` template in one of these ways:

- Use this `azd init` command from an empty local (root) folder:

    ```shell
    azd init --template functions-quickstart-powershell-azd-timer
    ```

    Supply an environment name, such as `flexquickstart` when prompted. In `azd`, the environment is used to maintain a unique deployment context for your app.

- Clone the GitHub template repository locally using the `git clone` command:

    ```shell
    git clone https://github.com/Azure-Samples/functions-quickstart-powershell-azd-timer.git
    cd functions-quickstart-powershell-azd-timer
    ```

    You can also clone the repository from your own fork in GitHub.

## Prepare your local environment

Add a file named `local.settings.json` in the `src` folder with the following contents:

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "powershell",
        "TIMER_SCHEDULE": "*/30 * * * * *"
    }
}
```

The `TIMER_SCHEDULE` setting defines when your timer function runs using NCRONTAB format. The example above runs every 30 seconds. For more information on NCRONTAB expressions, see [Configuration](#configuration).

## Run your app from the terminal

1. Start the Azurite storage emulator in a separate terminal window. Use whichever install option you set up in the prerequisites — for example, the `npm` package:

   ```shell
   azurite
   ```

   Or, if you installed Docker, run the Azurite container:

   ```shell
   docker run -p 10000:10000 -p 10001:10001 -p 10002:10002 mcr.microsoft.com/azure-storage/azurite
   ```

2. From the `src` folder, run this command to start the Functions host locally:

    ```shell
    cd src
    func start
    ```

3. Wait for the timer schedule to execute the timer trigger.

4. When you're done, press Ctrl+C in the terminal window to stop the `func.exe` host process.

## Run your app using Visual Studio Code

1. Open the `src` app folder in a new terminal.
2. Run the `code .` command to open the project in Visual Studio Code.
3. In the command palette (F1), type `Azurite: Start`, which enables debugging without warnings.
4. Press **Run/Debug (F5)** to run in the debugger. Select **Debug anyway** if prompted about local emulator not running.
5. Wait for the timer schedule to trigger your timer function.

## Deploy to Azure

Run this command to provision the function app, with any required Azure resources, and deploy your code:

```shell
azd up
```

Alternatively, you can opt-out of a VNet being used in the sample. To do so, use `azd env` to configure `VNET_ENABLED` to `false` before running `azd up`:

```bash
azd env set VNET_ENABLED false
azd up
```

## Redeploy your code

You can run the `azd up` command as many times as you need to both provision your Azure resources and deploy code updates to your function app.

> [!NOTE]
> Deployed code files are always overwritten by the latest deployment package.

## Clean up resources

When you're done working with your function app and related resources, you can use this command to delete the function app and its related resources from Azure and avoid incurring any further costs:

```shell
azd down
```

## Source Code

The function code for the timer trigger is defined in [`src/timerFunction/run.ps1`](./src/timerFunction/run.ps1) with its binding configuration in [`src/timerFunction/function.json`](./src/timerFunction/function.json).

This code shows the timer function implementation:

**function.json** (binding configuration):
```json
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "%TIMER_SCHEDULE%",
      "runOnStartup": true,
      "useMonitor": true
    }
  ]
}
```

**run.ps1** (function code):
```powershell
# Input bindings are passed in via param block.
param($myTimer)

# Get the current universal time in the default string format.
$currentUTCtime = (Get-Date).ToUniversalTime()

# The 'IsPastDue' property is 'true' when the current function invocation is later than scheduled.
if ($myTimer.IsPastDue) {
    Write-Host "PowerShell timer is running late!"
}

# Write an information log with the current time.
Write-Host "PowerShell timer trigger function executed at: $currentUTCtime"
```

PowerShell Azure Functions use a [function.json](https://learn.microsoft.com/azure/azure-functions/functions-reference-powershell#folder-structure) file to define input and output bindings, with the function logic in a corresponding `run.ps1` script.

### Key Features

1. **Parameterized Schedule**: The function uses the `%TIMER_SCHEDULE%` environment variable to determine the execution schedule, making it configurable without code changes.

2. **RunOnStartup Parameter**: Setting `runOnStartup: true` makes the function execute immediately when the app starts, in addition to running on the defined schedule. This is useful for testing but should be set to `false` in production.

3. **Past Due Detection**: The function checks if the timer is past due using the `$myTimer.IsPastDue` property, allowing for appropriate handling of delayed executions.

4. **PowerShell Function Model**: This function uses the PowerShell function model with `function.json` bindings and a `run.ps1` entry point, providing a familiar scripting experience for PowerShell developers.

5. **Built-in Logging**: The function uses `Write-Host` for logging, which integrates seamlessly with Azure Functions and Application Insights.

### Configuration

The timer schedule is configured through the `TIMER_SCHEDULE` application setting, which follows the NCRONTAB expression format. For example:

- `0 */5 * * * *` - Run once every 5 minutes
- `0 0 */1 * * *` - Run once every hour
- `0 0 0 * * *` - Run once every day at midnight

For more information on NCRONTAB expressions, see [Timer trigger for Azure Functions](https://learn.microsoft.com/azure/azure-functions/functions-bindings-timer).
