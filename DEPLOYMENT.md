# VoxOps Deployment - Next Steps

## ✅ Completed

1. ✅ Cloned Azure sample and detached from original Git history
2. ✅ Rebranded to VoxOps in `azure.yaml` (services renamed to `voxops-client` and `voxops-server`)
3. ✅ Updated Terraform defaults (name=voxops, location=eastus2, simplified RG naming to `rg-voxops`)
4. ✅ Added 4 missing conversation templates to load testing:
   - claim_filing
   - policy_update
   - billing_inquiry
   - confused_customer
5. ✅ Updated agent YAML files to use "VoxOps Demo" organization
6. ✅ Created clean git history and pushed to github.com/shmindmaster/voxops
7. ✅ Added VoxOps README with attribution to Azure sample

## 🚀 Next Steps to Deploy

### 1. Configure Azure Developer CLI Environment

From the repository root (`H:\Repos\shmindmaster\voxops`):

```powershell
# Create the prod environment
azd env new prod

# Configure environment variables
azd env set AZURE_LOCATION eastus2
azd env set NAME_PREFIX voxops
# Optional: Explicitly set resource group if needed
# azd env set AZURE_RESOURCE_GROUP rg-voxops
```

### 2. Configure Azure AI Services

Before deploying, ensure you have set up or plan to configure:

**Azure Speech Service** (East US 2):

```powershell
azd env set AZURE_SPEECH_REGION eastus2
azd env set AZURE_SPEECH_KEY <your-speech-key>
```

**Azure OpenAI** (East US 2):

```powershell
azd env set AZURE_OPENAI_ENDPOINT https://<your-openai-resource>.openai.azure.com/
azd env set AZURE_OPENAI_API_KEY <your-api-key>
```

**Azure Communication Services** (optional for PSTN):

```powershell
azd env set ACS_CONNECTION_STRING <your-acs-connection-string>
```

### 3. Deploy Infrastructure & Applications

```powershell
# Ensure you're logged in to Azure
azd auth login

# Verify your subscription is set
az account show

# Deploy everything (Terraform provision + Container Apps deploy)
azd up --no-prompt
```

This will:

- Create resource group `rg-voxops` in East US 2
- Provision all Azure resources via Terraform
- Build and deploy both container apps (voxops-client and voxops-server)

### 4. Get Container App URLs

After deployment completes:

```powershell
# Get frontend URL
az containerapp show -n voxops-client -g rg-voxops --query properties.configuration.ingress.fqdn -o tsv

# Get backend URL
az containerapp show -n voxops-server -g rg-voxops --query properties.configuration.ingress.fqdn -o tsv
```

### 5. Configure Custom Domains (voxops.shtrial.com)

#### 5a. Create DNS Zone (if needed)

```powershell
az network dns zone create -g rg-voxops -n shtrial.com
```

#### 5b. Add CNAME Records

Use the FQDNs from step 4 to create CNAMEs:

```powershell
# Replace <frontend-fqdn> and <backend-fqdn> with actual values
az network dns record-set cname set-record -g rg-voxops -z shtrial.com -n voxops -c <frontend-fqdn>
az network dns record-set cname set-record -g rg-voxops -z shtrial.com -n api -c <backend-fqdn>
```

#### 5c. Create Managed Certificates & Bind Hostnames

```powershell
# Frontend: voxops.shtrial.com
az containerapp managed-certificate create `
  -g rg-voxops -n voxops-client `
  --hostname voxops.shtrial.com `
  --certificate-name voxops-cert

az containerapp hostname bind `
  -g rg-voxops -n voxops-client `
  --hostname voxops.shtrial.com `
  --certificate-name voxops-cert

# Backend: api.voxops.shtrial.com
az containerapp managed-certificate create `
  -g rg-voxops -n voxops-server `
  --hostname api.voxops.shtrial.com `
  --certificate-name voxops-api-cert

az containerapp hostname bind `
  -g rg-voxops -n voxops-server `
  --hostname api.voxops.shtrial.com `
  --certificate-name voxops-api-cert
```

### 6. Point Frontend to Backend

Update the frontend environment to use the custom backend URL:

```powershell
azd env set VITE_BACKEND_BASE_URL https://api.voxops.shtrial.com

# Redeploy frontend with new config
azd deploy voxops-client
```

### 7. Verify Deployment

Open browser to:

- https://voxops.shtrial.com (or the Container App FQDN if custom domain not configured)

Test scenarios:

1. "I need to file a new claim" → FNOL intake
2. "What's my deductible?" → General info
3. "I want roadside assistance added" → Policy update
4. "Why is my billing higher this month?" → Billing inquiry
5. "I'm frustrated, get me a human" → Escalation

## 📋 Configuration Reference

### Container Apps Scaling (Demo/Cost Optimization)

Both apps are configured for **Consumption plan** in the Terraform defaults:

- **Min replicas**: 5 (can reduce to 0-1 for demo)
- **Max replicas**: 50
- Scale to zero when idle = cost-effective for demos

To adjust for demo (optional), edit `infra/terraform/variables.tf`:

```hcl
variable "container_app_min_replicas" {
  default = 0  # or 1 for faster cold starts
}
```

### Model Deployments

Default models in `infra/terraform/variables.tf`:

- gpt-4o (2024-11-20) - 150 capacity
- gpt-4o-mini (2024-07-18) - 150 capacity
- gpt-4.1-mini (2025-04-14) - 150 capacity
- gpt-4.1 (2025-04-14) - 150 capacity

For demo, consider using only `gpt-4o-mini` to reduce costs.

## 🔧 Troubleshooting

### If deployment fails:

1. Check Terraform state:

```powershell
cd infra/terraform
terraform plan
```

2. Check Container Apps logs:

```powershell
az containerapp logs show -n voxops-server -g rg-voxops --tail 50
az containerapp logs show -n voxops-client -g rg-voxops --tail 50
```

3. Verify environment variables:

```powershell
azd env get-values
```

### Common issues:

- **Speech Service quota**: Ensure your subscription has quota in East US 2
- **OpenAI models**: Verify model deployments exist in your Azure OpenAI resource
- **ACS**: If testing telephony, ensure you have a valid phone number provisioned

## 📚 Additional Resources

- [Azure Developer CLI Docs](https://learn.microsoft.com/azure/developer/azure-developer-cli/)
- [Container Apps Custom Domains](https://learn.microsoft.com/azure/container-apps/custom-domains-certificates)
- [Original Sample Docs](https://azure-samples.github.io/art-voice-agent-accelerator/)

---

**Repository**: https://github.com/shmindmaster/voxops
**Live Demo** (after deployment): https://voxops.shtrial.com
