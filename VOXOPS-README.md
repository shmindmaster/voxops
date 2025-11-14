# VoxOps - Real-Time Voice Agent Demo

**VoxOps** is a demonstration deployment derived from the [Azure Real-Time Voice Agent Accelerator](https://github.com/Azure-Samples/art-voice-agent-accelerator) (MIT License).

## Overview

This repository showcases a working implementation of Azure Communication Services, Azure Speech, and Azure OpenAI for building production-grade voice agents.

**Live Demo**: https://voxops.shtrial.com

## What VoxOps Demonstrates

- **Omnichannel voice capabilities** including web-based and telephony (PSTN/SIP via Azure Communication Services)
- **Low-latency voice pipeline** with Speech-to-Text → LLM → Text-to-Speech orchestration
- **Multi-agent handoffs** for:
  - Authentication
  - Claims intake (First Notice of Loss / FNOL)
  - General insurance inquiries
  - Policy updates
  - Billing inquiries
- **Load testing framework** with realistic conversation templates:
  - Insurance inquiry
  - Quick questions
  - Claim filing
  - Policy updates
  - Billing inquiries
  - Confused customer escalation

## Architecture

- **Frontend**: React + Vite (Container Apps)
- **Backend**: FastAPI + Python 3.11 (Container Apps)
- **Infrastructure**: Azure Container Apps (Consumption plan for demo cost optimization)
- **Region**: East US 2
- **Resource Group**: `rg-voxops`

## Key Changes from Original Sample

1. **Branding**: Renamed from `art-voice-agent-accelerator` to `voxops`
2. **Services**:
   - `rtaudio-client` → `voxops-client`
   - `rtaudio-server` → `voxops-server`
3. **Organization**: Changed from "GBB AI" to "VoxOps Demo" in agent configurations
4. **Load Testing**: Added 4 additional conversation scenario templates:
   - Claim filing
   - Policy updates
   - Billing inquiries
   - Confused customer handling
5. **Infrastructure**: Simplified resource group naming to `rg-voxops`
6. **Defaults**: Set default location to `eastus2` and default name prefix to `voxops`

## Quick Start

### Prerequisites

- Azure CLI
- Azure Developer CLI (`azd`)
- Node.js 20+
- Python 3.11+
- Docker Desktop
- Terraform 1.7+

### Deploy to Azure

```bash
# Login to Azure
azd auth login

# Create environment
azd env new prod

# Set environment variables
azd env set AZURE_LOCATION eastus2
azd env set NAME_PREFIX voxops

# Provision infrastructure and deploy
azd up --no-prompt
```

### Configure Custom Domain (Optional)

After deployment, to use custom domains like `voxops.shtrial.com`:

```bash
# Get Container App FQDNs
az containerapp show -n voxops-client -g rg-voxops --query properties.configuration.ingress.fqdn -o tsv
az containerapp show -n voxops-server -g rg-voxops --query properties.configuration.ingress.fqdn -o tsv

# Create DNS CNAME records in your zone
# Then bind custom domains with managed certificates
az containerapp managed-certificate create \
  -g rg-voxops -n voxops-client \
  --hostname voxops.shtrial.com \
  --certificate-name voxops-cert

az containerapp hostname bind \
  -g rg-voxops -n voxops-client \
  --hostname voxops.shtrial.com \
  --certificate-name voxops-cert
```

## Attribution

This project is derived from the [Azure-Samples/art-voice-agent-accelerator](https://github.com/Azure-Samples/art-voice-agent-accelerator) repository, which is licensed under the MIT License.

**Original Authors**: Azure Global Black Belt (GBB) AI Team
**Original License**: MIT (see LICENSE file)

All foundational voice agent architecture, ACS integration, STT/TTS pipelines, and orchestration patterns are from the original Azure sample. VoxOps is a demonstration deployment with branding and configuration changes for the VoxOps demo environment.

## License

MIT License - see [LICENSE](LICENSE) file for details.

This derivative work maintains the same MIT license as the original Azure sample.

## Documentation

For comprehensive documentation on the underlying architecture, API reference, and deployment guides, see the original project documentation:

https://azure-samples.github.io/art-voice-agent-accelerator/
