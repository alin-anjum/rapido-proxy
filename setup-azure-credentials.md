# Azure Service Principal Setup for GitHub Actions

## Prerequisites
- Azure CLI installed and logged in
- Azure subscription with Contributor permissions
- GitHub repository with Actions enabled

## Step 1: Login to Azure CLI
```bash
az login
```

## Step 2: Get Your Subscription ID
```bash
az account show --query id --output tsv
```
**Save this value - you'll need it for AZURE_SUBSCRIPTION_ID**

## Step 3: Create Service Principal
Replace `YOUR_SUBSCRIPTION_ID` with the ID from Step 2:

```bash
az ad sp create-for-rbac \
  --name "rapido-github-actions" \
  --role contributor \
  --scopes /subscriptions/YOUR_SUBSCRIPTION_ID \
  --json-auth
```

## Step 4: Copy the Output
The command will output JSON like this:
```json
{
  "clientId": "12345678-1234-1234-1234-123456789012",
  "clientSecret": "your-client-secret",
  "subscriptionId": "your-subscription-id",
  "tenantId": "87654321-4321-4321-4321-210987654321",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

## Step 5: Set GitHub Secrets
Go to your GitHub repository → Settings → Secrets and variables → Actions

Add these Repository Secrets:
- `AZURE_CLIENT_ID` = `clientId` from the JSON
- `AZURE_TENANT_ID` = `tenantId` from the JSON  
- `AZURE_SUBSCRIPTION_ID` = `subscriptionId` from the JSON

## Step 6: Update Workflow File
In `.github/workflows/main_streaming-api-test.yml`, update the app names:
- Line 78: Change `'rapido-api-proxy'` to your actual Azure App Service name
- Line 85: Change `'rapido-api-proxy'` to your actual Azure App Service name

## Alternative: Azure Portal Method

### Using Azure Portal:
1. Go to **Azure Portal** → **Azure Active Directory**
2. Click **App registrations** → **New registration**
3. Name: `rapido-github-actions`
4. Click **Register**
5. Copy the **Application (client) ID** → This is your `AZURE_CLIENT_ID`
6. Copy the **Directory (tenant) ID** → This is your `AZURE_TENANT_ID`
7. Go to **Certificates & secrets** → **New client secret**
8. Copy the secret value → This is your `AZURE_CLIENT_SECRET` (not needed for our workflow)
9. Go to **Subscriptions** → Select your subscription → **Access control (IAM)**
10. Click **Add role assignment** → **Contributor** → Assign to your app registration

## Troubleshooting

### Common Issues:
- **Permission Denied**: Ensure you have Owner/Contributor role on the subscription
- **App Already Exists**: Use a different name or delete the existing one
- **Invalid Subscription**: Verify you're using the correct subscription ID

### Verify Setup:
```bash
# Test the service principal
az login --service-principal \
  --username YOUR_CLIENT_ID \
  --password YOUR_CLIENT_SECRET \
  --tenant YOUR_TENANT_ID
```

## Security Best Practices
- ✅ Use least privilege (Contributor role only)
- ✅ Store secrets in GitHub Secrets (never in code)
- ✅ Rotate client secrets regularly
- ✅ Monitor service principal usage in Azure logs
