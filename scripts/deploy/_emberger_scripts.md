# emberger deployment steps/notes

## Windows local machine prep

### Software requirements

Install with:

```powershell
   cd D:\Git\GitHub\gopher194_personal\gopher194\chat-copilot-fork
   cd scripts
   .\Install.ps1
```

   > NOTE: This script will install `Chocolatey`, `dotnet-7.0-sdk`, `nodejs`, and `yarn`.

## Azure deployment

### Deploy Azure Infrastructure

#### Application Landing Zone

Done with terraform.

#### deploy-azure.ps1

Creates all the application related resources.

```powershell
cd '\chat-copilot-fork\scripts\deploy'

az login

az account set -s $SUBSCRIPTION_ID

./deploy-azure.ps1 -Subscription $SUBSCRIPTION_ID -DeploymentName $DEPLOYMENT_NAME -AIService $AI_SERVICE -BackendClientId $BACKEND_APPLICATION_ID -FrontendClientId $FRONTEND_APPLICATION_ID -TenantId $TENANT_ID -ResourceGroup $RESOURCE_GROUP_NAME -Region $REGION -AIApiKey $AI_KEY -AIEndpoint $AZURE_OPENAI_ENDPOINT
```

### Deploy Application

#### package-webapi

Build and package as Zip the `webapi/CopilotChatWebApi.csproj`

```powershell
./package-webapi.ps1 -Version "0.0.1" -InformationalVersion "Initial Zip package"
```

#### deploy-webapi

Deploy the Zip to the web application in Azure.

```powershell
./deploy-webapi.ps1 -Subscription $SUBSCRIPTION_ID -DeploymentName $DEPLOYMENT_NAME -ResourceGroupName $RESOURCE_GROUP_NAME
```

- Error:
```
az webapp deployment source config-zip --resource-group $RESOURCE_GROUP_NAME --name $webApiName --src $PackageFilePath
Getting scm site credentials for zip deployment
Starting zip deployment. This operation can take a while to complete ...
HTTPSConnectionPool(host='app-copichat-4styicvu54uz4-webapi.scm.azurewebsites.net', port=443): Max retries exceeded with url: /api/zipdeploy?isAsync=true (Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:2396)')))
Certificate verification failed. This typically happens when using Azure CLI behind a proxy that intercepts traffic with a self-signed certificate. Please add this certificate to the trusted CA bundle. More info: <https://docs.microsoft.com/cli/azure/use-cli-effectively#work-behind-a-proxy>.
```

- Fix:
  - Change `Settings` / `Configuration` / `General settings` / `Basic Auth Publishing Credential`
  - From default `Off` to `On` / `Save`

### Deploy MemoryPipeline (optional)

```powershell
.\package-memorypipeline.ps1

.\deploy-memorypipeline.ps1 -Subscription $SUBSCRIPTION_ID -DeploymentName $DEPLOYMENT_NAME -ResourceGroupName $RESOURCE_GROUP_NAME
```

## Local deployment

### Configure Chat Copilot

#### without AAD

   ```powershell
   .\Configure.ps1 -AIService $AI_SERVICE -APIKey $AI_KEY -Endpoint $AZURE_OPENAI_ENDPOINT
   ```

#### with AAD

   ```powershell
   .\Configure.ps1 -AiService $AI_SERVICE -APIKey $AI_KEY -Endpoint $AZURE_OPENAI_ENDPOINT -BackendClientId $BACKEND_APPLICATION_ID -FrontendClientId $FRONTEND_APPLICATION_ID -TenantId $TENANT_ID -Instance $AZURE_AD_INSTANCE
  ```

### Run Chat Copilot locally

   ```powershell
   cd D:\Git\GitHub\gopher194_personal\gopher194\chat-copilot-fork
   cd scripts

  # This script starts the backend (dotnet core app in ../webapi on https://localhost:40443) first,
  # then the frontend (yarn in ../webapp on http://localhost:3000/)
   .\Start.ps1
   ```

- (Optional) To start ONLY the backend:

     ```powershell
     .\Start-Backend.ps1
     ```

- (Optional) To start ONLY the frontend:

     ```powershell
     .\Start-Frontend.ps1
     ```
