# Runtime and Model Configuration Guide

## Scope

<details>
<summary>What this guide covers</summary>

This guide defines the baseline runtime/config needed across Labs 01-11:

- Python virtual environments
- Ollama install and model setup
- Environment variable profiles (local and container)
- Docker install/verify/runbook for Linux and Windows PowerShell
- AWS and Azure CLI setup and deployment runbooks
- Pre-commit and post-commit testing configuration

</details>

---

## 1. Tooling Baseline

<details>
<summary>Required and optional tools</summary>

| Tool | Requirement |
| --- | --- |
| VS Code | Recommended |
| Python 3.11+ | Required |
| Ollama | Required |
| Docker | Required for container labs |
| AWS Cloud | Required for Lab 11 cloud deployment |
| Azure Cloud | Required for Lab 11 cloud deployment |
| Qwen (`qwen2.5:0.5b`) | Required — primary implementation model |
| Llama (`llama3.1:8b`) | Required — review model |
| Deepseek (`deepseek-r1:8b`) | Required for later labs — reasoning model |

</details>

---

## 2. Repository Setup

<details>
<summary>Create and clone your course repository</summary>

Create a **private GitHub repository** named:

```text
enrolment-app-open-ai
```

Clone the repository to your local machine.

```bash
git clone https://github.com/<your-github-username>/enrolment-app-open-ai.git
cd enrolment-app-open-ai
```

Open the repository in Visual Studio Code.

```bash
code .
```

Alternatively:

- Open **Visual Studio Code**.
- Select **File → Open Folder**.
- Open the **enrolment-app-open-ai** folder.

Verify Git is working correctly.

```bash
git status
```

Expected output:

```text
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

Your local workspace is now ready. All labs in this course assume you are working from the **enrolment-app-open-ai** repository.

</details>

<details>
<summary>Install Git Bash (Windows)</summary>

Windows users should install Git Bash so the `bash` commands shown throughout this guide can run as-is.

Download from [git-scm.com/download/win](https://git-scm.com/download/win), or install via winget:

```powershell
winget install --id Git.Git -e --source winget
```

Verify:

```bash
git --version
bash --version
```

Open **Git Bash** (not PowerShell) whenever a command in this guide is shown under a `bash` code block.

</details>

---

## 3. Python Environment

<details>
<summary>Linux/macOS</summary>

```bash
python -m venv .venv
source .venv/bin/activate
.venv/bin/python -m pip install --upgrade pip
```

Deactivate:

```bash
deactivate
```

</details>

<details>
<summary>Windows PowerShell</summary>

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
.venv\Scripts\python -m pip install --upgrade pip
```

If script execution is blocked:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Deactivate:

```powershell
deactivate
```

</details>

---

## 4. Ollama Install and Runtime

<details>
<summary>Install Ollama</summary>

Linux:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

macOS/Windows:

```text
https://ollama.com/download
```

</details>

<details>
<summary>Runtime checks (Linux/macOS)</summary>

```bash
ollama serve
ollama --version
ollama list
ollama ps
curl http://127.0.0.1:11434
```

</details>

<details>
<summary>Runtime checks (Windows PowerShell)</summary>

```powershell
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" serve
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" --version
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" list
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" ps
curl.exe http://127.0.0.1:11434
```

</details>

---

## 5. Model Setup

<details>
<summary>Pull required models</summary>

| Model | Purpose |
| --- | --- |
| `qwen2.5:0.5b` | Primary implementation model |
| `llama3.1:8b` | Review model |
| `deepseek-r1:8b` | Reasoning model (used in later labs) |

Linux/macOS:

```bash
ollama pull qwen2.5:0.5b
ollama pull llama3.1:8b
ollama pull deepseek-r1:8b
ollama run qwen2.5:0.5b   # replace the model name to run any of the above
```

Windows PowerShell:

```powershell
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" pull qwen2.5:0.5b
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" pull llama3.1:8b
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" pull deepseek-r1:8b
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" run qwen2.5:0.5b
```

</details>

<details>
<summary>Verify models via API</summary>

Linux/macOS:

```bash
curl http://localhost:11434/api/tags
```

Windows PowerShell:

```powershell
curl.exe http://localhost:11434/api/tags
```

</details>

<details>
<summary>Environment profiles</summary>

Shared by every profile:

```env
OLLAMA_MODEL=qwen2.5:0.5b
OLLAMA_REVIEW_MODEL=llama3.1:8b
```

| Profile | Use when | `OLLAMA_BASE_URL` | `DATABASE_SERVICE_URL` |
| --- | --- | --- | --- |
| Local (Labs 01-11) | Running services directly with `python` in separate terminals | `http://localhost:11434/v1` | `http://localhost:5002` |
| Container (Docker Compose) | Services run inside Docker containers | `http://host.docker.internal:11434/v1` | `http://database-service:5002` |

Reasoning-enabled (RAG and later): same as the Local profile, plus:

```env
OLLAMA_REASONING_MODEL=deepseek-r1:8b
```

</details>

<details>
<summary>Common runtime flags from labs</summary>

```env
FLASK_OPEN_CHROME=0
```

Use when launching Flask apps in environments where auto browser open is not desired.

</details>

---

## 6. Docker Install and Verification

<details>
<summary>Compose command compatibility</summary>

Try:

```bash
docker compose version
```

Fallback:

```bash
docker-compose --version
```

Use whichever command works in your environment. Docker Desktop on Windows normally provides `docker compose`. Some Linux distributions provide only the legacy `docker-compose` package.

</details>

<details>
<summary>Linux install and verify</summary>

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin || sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker

docker --version
docker compose version || docker-compose --version
docker info
docker run --rm hello-world
```

If the install falls back to `docker-compose`, use `docker-compose --version` and replace later `docker compose` commands with `docker-compose`.

</details>

<details>
<summary>Windows PowerShell install and verify</summary>

Run in Administrator PowerShell:

```powershell
winget install -e --id Docker.DockerDesktop --accept-package-agreements --accept-source-agreements
Start-Process "Docker Desktop"
```

After Docker Desktop shows Engine running:

```powershell
docker --version
docker compose version
docker info
docker run --rm hello-world
```

</details>

<details>
<summary>Windows engine-not-running fix</summary>

If `docker info` returns pipe errors, run in Administrator PowerShell and reboot:

```powershell
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism /online /enable-feature /featurename:HypervisorPlatform /all /norestart
bcdedit /set hypervisorlaunchtype auto
```

After reboot:

```powershell
wsl --update
wsl --set-default-version 2
docker info
```

</details>

<details>
<summary>Connect containers to local Ollama</summary>

Linux:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d
printf "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0:11434\"\n" | sudo tee /etc/systemd/system/ollama.service.d/override.conf >/dev/null
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Windows PowerShell:

```powershell
setx OLLAMA_HOST "0.0.0.0:11434"
```

Close and reopen Ollama after `setx`.

</details>

<details>
<summary>Container runbook: Terminal A - build and run</summary>

```bash
cd enrolment-app-open-ai
docker compose up --build
```

</details>

<details>
<summary>Container runbook: Terminal B - status and logs</summary>

```bash
docker compose ps
docker compose logs -f
```

</details>

<details>
<summary>Container runbook: Terminal C - functional checks (Linux/macOS)</summary>

```bash
curl http://localhost:5002/students
curl http://localhost:5001/
curl -X POST http://localhost:5001/ask -d "question=Say hello in one sentence"
```

</details>

<details>
<summary>Container runbook: Terminal C - functional checks (Windows PowerShell)</summary>

```powershell
Invoke-RestMethod -Method Get -Uri "http://localhost:5002/students" | ConvertTo-Json -Depth 6
Invoke-RestMethod -Method Get -Uri "http://localhost:5001/" | ConvertTo-Json -Depth 6
Invoke-RestMethod -Method Post -Uri "http://localhost:5001/ask" -Body @{ question = "Say hello in one sentence" } | ConvertTo-Json -Depth 6
```

</details>

<details>
<summary>Verify Ollama is reachable from a container</summary>

```bash
docker compose exec enrolment-service python -c "import requests; print(requests.get('http://host.docker.internal:11434/api/tags', timeout=5).status_code)"
```

Expected result: `200`

</details>

<details>
<summary>Stop and cleanup</summary>

```bash
docker compose down
docker compose rm -f
docker compose up --build --force-recreate
```

</details>

---

## 7. AWS Configuration

<details>
<summary>Sign up for AWS (free tier)</summary>

Create an account at [aws.amazon.com/free](https://aws.amazon.com/free/). Students may qualify for extra credits through [AWS Educate](https://aws.amazon.com/education/awseducate/). A credit card is required even on the free tier.

</details>

<details>
<summary>AWS CLI install</summary>

Linux:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Windows PowerShell:

```powershell
winget install -e --id Amazon.AWSCLI
aws --version
```

</details>

<details>
<summary>AWS CLI Config</summary>

AWS CLI settings live in two files: `~/.aws/config` (default region and output format) and `~/.aws/credentials` (access keys).

```bash
aws configure set region ap-southeast-2
aws configure list
```

</details>

<details>
<summary>AWS CLI login and credentials</summary>

Use one of these sign-in paths (same commands on Linux, macOS, and Windows PowerShell):

```bash
aws configure sso
```

Expect this when your class uses AWS SSO. The CLI will open a browser or print a browser URL and code, then store a local profile.

```bash
aws configure
```

Use this when your course gives you AWS access keys. The CLI will prompt for the access key ID, secret access key, default region, and output format.

</details>

<details>
<summary>AWS CLI and Terraform quick reference</summary>

Use these commands most often when working with AWS CLI and Terraform:

| Command | Meaning |
| --- | --- |
| `aws configure sso` | Sign in with AWS Single Sign-On and store a profile locally. |
| `aws configure` | Save AWS access key ID, secret access key, default region, and output format. |
| `aws sts get-caller-identity` | Confirm which AWS account and role are currently active. |
| `aws ecr create-repository --repository-name <name>` | Create an ECR repository for container images. |
| `aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com` | Authenticate Docker to push images into Amazon ECR. |
| `terraform init` | Download providers and initialize the Terraform working directory. |
| `terraform plan` | Preview the infrastructure changes Terraform will make. |
| `terraform apply` | Create or update AWS infrastructure from the Terraform configuration. |
| `terraform destroy` | Remove the AWS resources managed by the Terraform configuration. |

</details>

<details>
<summary>AWS deployment runbook</summary>

Use this when you need to deploy the Lab 11 containerized enrolment service with AWS CLI and Terraform. This is the exact script used in `scripts/lab11/deploy-aws.sh`.

```bash
#!/usr/bin/env bash
set -euo pipefail

: "${AWS_REGION:=ap-southeast-2}"
: "${AWS_ACCOUNT_ID:=}"
: "${AWS_ECR_REPOSITORY:=enrolment-service}"
: "${AWS_IMAGE_TAG:=v1}"
: "${AWS_EXECUTION_ROLE_ARN:=}"
: "${AWS_SECURITY_GROUP_ID:=}"
: "${AWS_SUBNET_IDS_JSON:=}"

if [[ -z "$AWS_ACCOUNT_ID" ]]; then
   echo "AWS_ACCOUNT_ID must be set" >&2
   exit 1
fi

if [[ -z "$AWS_EXECUTION_ROLE_ARN" || -z "$AWS_SECURITY_GROUP_ID" || -z "$AWS_SUBNET_IDS_JSON" ]]; then
   echo "AWS_EXECUTION_ROLE_ARN, AWS_SECURITY_GROUP_ID, and AWS_SUBNET_IDS_JSON must be set" >&2
   exit 1
fi

AWS_ECR_URI="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$AWS_ECR_REPOSITORY"

aws ecr create-repository --repository-name "$AWS_ECR_REPOSITORY" --region "$AWS_REGION"
aws ecr get-login-password --region "$AWS_REGION" | docker login --username AWS --password-stdin "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

docker build -t "$AWS_ECR_REPOSITORY:$AWS_IMAGE_TAG" ./enrolment-service
docker tag "$AWS_ECR_REPOSITORY:$AWS_IMAGE_TAG" "$AWS_ECR_URI:$AWS_IMAGE_TAG"
docker push "$AWS_ECR_URI:$AWS_IMAGE_TAG"

cd deployment/aws
terraform init
terraform plan \
   -var="image=$AWS_ECR_URI:$AWS_IMAGE_TAG" \
   -var="execution_role_arn=$AWS_EXECUTION_ROLE_ARN" \
   -var="security_group_id=$AWS_SECURITY_GROUP_ID" \
   -var="subnet_ids=$AWS_SUBNET_IDS_JSON"
terraform apply \
   -var="image=$AWS_ECR_URI:$AWS_IMAGE_TAG" \
   -var="execution_role_arn=$AWS_EXECUTION_ROLE_ARN" \
   -var="security_group_id=$AWS_SECURITY_GROUP_ID" \
   -var="subnet_ids=$AWS_SUBNET_IDS_JSON" \
   -auto-approve
```

</details>

---

## 8. Azure Configuration

<details>
<summary>Sign up for Azure (free tier)</summary>

Create an account at [azure.microsoft.com/free](https://azure.microsoft.com/free/), or [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students/) for eligible students (no credit card required).

</details>

<details>
<summary>Azure CLI install</summary>

Linux:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
az version
```

Windows PowerShell:

```powershell
winget install -e --id Microsoft.AzureCLI
az version
```

</details>

<details>
<summary>Azure CLI Config</summary>

Azure CLI stores settings under `~/.azure`. Check the active subscription, or switch subscriptions if your account has more than one:

```bash
az account show
az account set --subscription "<subscription-id-or-name>"
```

</details>

<details>
<summary>Azure CLI login and credentials</summary>

Use one of these sign-in paths (same commands on Linux, macOS, and Windows PowerShell):

```bash
az login
```

Expect Azure to open a browser sign-in page. No access keys are required.

```bash
az login --use-device-code
```

Use this on a headless shell, or if the browser sign-in does not open automatically. Azure prints a device code that you paste into the Microsoft sign-in page.

</details>

<details>
<summary>Azure CLI and ARM quick reference</summary>

Use these commands most often when working with Azure CLI and ARM templates:

| Command | Meaning |
| --- | --- |
| `az login` | Sign in to Azure and select the active subscription. |
| `az login --use-device-code` | Sign in without a browser by using a device code flow. |
| `az account show` | Display the current Azure subscription and account context. |
| `az group create --name <rg> --location <region>` | Create the Azure resource group that will contain the deployment. |
| `az acr create --resource-group <rg> --name <acr> --sku Basic` | Create an Azure Container Registry for Docker images. |
| `az acr login --name <acr>` | Authenticate Docker to push images into Azure Container Registry. |
| `az deployment group create --resource-group <rg> --template-file <template.json>` | Deploy an ARM template into a resource group. |
| `az containerapp env create --name <env> --resource-group <rg> --location <region>` | Create the Container Apps environment used by the app. |
| `az containerapp create --name <app> --resource-group <rg> --environment <env> --image <image> --target-port 5001 --ingress external` | Create and expose the containerized enrolment service. |
| `az containerapp show --name <app> --resource-group <rg>` | Inspect the deployed container app and its public endpoint. |

</details>

<details>
<summary>Azure deployment runbook</summary>

Use this when you need to deploy the Lab 11 containerized enrolment service with Azure CLI and an ARM template. This is the exact script used in `scripts/lab11/deploy-azure.sh`.

```bash
#!/usr/bin/env bash
set -euo pipefail

: "${AZURE_RESOURCE_GROUP:=enrolment-rg}"
: "${AZURE_LOCATION:=australiaeast}"
: "${AZURE_ACR_NAME:=enrolmentacr}"
: "${AZURE_CONTAINERAPP_ENV:=enrolment-env}"
: "${AZURE_APP_NAME:=enrolment-service}"
: "${AZURE_IMAGE_TAG:=v1}"
: "${AZURE_IMAGE_NAME:=enrolment-service}"
: "${AZURE_IMAGE_REGISTRY:=}"

if [[ -z "$AZURE_IMAGE_REGISTRY" ]]; then
   echo "AZURE_IMAGE_REGISTRY must be set to your ACR login server, for example enrolmentacr.azurecr.io" >&2
   exit 1
fi

az login
az group create --name "$AZURE_RESOURCE_GROUP" --location "$AZURE_LOCATION"
az acr create --resource-group "$AZURE_RESOURCE_GROUP" --name "$AZURE_ACR_NAME" --sku Basic
az acr login --name "$AZURE_ACR_NAME"

docker build -t "$AZURE_IMAGE_REGISTRY/$AZURE_IMAGE_NAME:$AZURE_IMAGE_TAG" ./enrolment-service
docker push "$AZURE_IMAGE_REGISTRY/$AZURE_IMAGE_NAME:$AZURE_IMAGE_TAG"

az containerapp env create \
   --name "$AZURE_CONTAINERAPP_ENV" \
   --resource-group "$AZURE_RESOURCE_GROUP" \
   --location "$AZURE_LOCATION"

az containerapp create \
   --name "$AZURE_APP_NAME" \
   --resource-group "$AZURE_RESOURCE_GROUP" \
   --environment "$AZURE_CONTAINERAPP_ENV" \
   --image "$AZURE_IMAGE_REGISTRY/$AZURE_IMAGE_NAME:$AZURE_IMAGE_TAG" \
   --target-port 5001 \
   --ingress external \
   --env-vars \
      DATABASE_SERVICE_URL=http://database-service:5002 \
      MCP_ENABLED=false \
      RAG_ENABLED=false
```

</details>

---

## 9. Pre and Post Commit Testing

<details>
<summary>Concepts</summary>

| Stage | Trigger | Tools |
| --- | --- | --- |
| Pre-commit | `git commit` (local git hook) | Ruff, detect-secrets, pip-audit |
| Post-commit | CI/CD pipeline (e.g. GitHub Actions) | pytest (API tests), PyJWT (token tests) |

</details>

<details>
<summary>What is a pre-commit YAML?</summary>

`.pre-commit-config.yaml` defines the hooks that run automatically on `git commit`, before the commit completes. Each hook runs a command — a scan or a test — and if any hook fails, the commit is blocked until the issue is fixed.

Sample snippet (not the full file):

```yaml
repos:
  - repo: local
    hooks:
      - id: ruff-scan
        name: Ruff Scan
        entry: ruff check .
        language: system
        stages: [commit]
```

</details>

<details>
<summary>Pre-commit config</summary>

```bash
cd enrolment-app-open-ai
pip install pytest pre-commit ruff detect-secrets pip-audit
pre-commit install
pre-commit run --all-files
```

</details>

<details>
<summary>How post-commit runs</summary>

Post-commit tests run in a CI/CD pipeline after code is pushed, not on the local machine. A workflow file defines the trigger (push or manual dispatch) and the steps to install dependencies, run the tests, and upload the results as evidence.

Sample CI/CD YAML snippet (not the full file):

```yaml
name: post-commit-tests
on:
  workflow_dispatch:
jobs:
  post_commit_tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: pip install pytest requests PyJWT
      - name: Run tests
        run: pytest tests/post_commit/ -v
```

</details>

<details>
<summary>Post-commit config</summary>

```bash
cd enrolment-app-open-ai
pip install pytest requests PyJWT
docker compose up -d --build
python -m pytest tests/lab11/post_commit/ -v
docker compose down -v
```

</details>

---

## 10. Course Validation Checklist

<details>
<summary>Quick checks before starting labs</summary>

| # | Check |
| --- | --- |
| 1 | Git installed; **`enrolment-app-open-ai`** repo created, cloned, and opened in VS Code |
| 2 | Windows: Git Bash installed |
| 3 | Python virtual environment activates; `pip` installs complete without errors |
| 4 | Ollama installed, running, and reachable at `http://localhost:11434` |
| 5 | `ollama list` includes `qwen2.5:0.5b`, `llama3.1:8b`, and `deepseek-r1:8b` |
| 6 | Correct `.env` profile configured for the selected runtime (local or container) |
| 7 | Docker installed, Docker Engine running, `docker compose` executes successfully |
| 8 | Container services build and start successfully via Docker Compose |
| 9 | Containers can reach the local Ollama service |
| 10 | Required ports free: `5001` (enrolment), `5002` (database), `5004` (multi-agent), `8080` (frontend) |
| 11 | AWS and/or Azure CLI installed and authenticated, if required for your labs |
| 12 | Pre-commit hooks installed (`pre-commit install`) and passing |

</details>
