# MyMvcApp-Contact-Databse-Application

# Deploying MyMvcApp to Azure with ARM Template and GitHub Actions

This guide explains how to deploy the ASP.NET Core MVC application (`MyMvcApp`) to Azure App Service using an ARM template and automate the deployment with a GitHub Actions workflow.

## Prerequisites
- An Azure subscription
- Azure CLI installed and logged in
- Resource group created in Azure
- The following files in your project root:
  - `deploy.json` (ARM template)
  - `deploy.parameters.json` (parameters for the ARM template)

## 1. ARM Template Deployment

### Files
- **deploy.json**: Defines Azure resources (App Service, App Service Plan).
- **deploy.parameters.json**: Specifies parameter values (site name, plan name, location, etc).

### Deploy via Azure CLI
Replace `<resource-group>` with your Azure resource group name:

```powershell
az deployment group create --resource-group <resource-group> --template-file deploy.json --parameters @deploy.parameters.json
```

This will provision the App Service and App Service Plan as defined in the templates.

## 2. GitHub Actions Workflow

You can automate build and deployment using GitHub Actions. Below is a sample workflow file (`.github/workflows/azure-deploy.yml`).

### Sample Workflow: `.github/workflows/azure-deploy.yml`

```yaml
name: Build and Deploy ASP.NET Core app to Azure Web App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Build
        run: dotnet build MyMvcApp.csproj --configuration Release

      - name: Publish
        run: dotnet publish MyMvcApp.csproj --configuration Release --output ./publish

      - name: Azure Login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy ARM Template
        uses: azure/arm-deploy@v1
        with:
          subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          resourceGroupName: <resource-group>
          template: ./deploy.json
          parameters: ./deploy.parameters.json

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v3
        with:
          app-name: <app-service-name>
          package: ./publish
```

### Notes
- Replace `<resource-group>` and `<app-service-name>` with your actual Azure resource group and App Service name.
- Set the following GitHub repository secrets:
  - `AZURE_CREDENTIALS`: Output of `az ad sp create-for-rbac --sdk-auth`
  - `AZURE_SUBSCRIPTION_ID`: Your Azure subscription ID

## References
- [Azure ARM Templates Documentation](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview)
- [GitHub Actions for Azure](https://github.com/Azure/actions)

---

This setup will provision Azure resources and deploy your ASP.NET Core MVC app automatically on every push to the `main` branch.
