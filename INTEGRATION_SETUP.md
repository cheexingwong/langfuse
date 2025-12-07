# Langfuse Integration Setup Guide

> **Note:** This is a custom setup guide for integrating Langfuse with the TD.AIBIPrototype project. For the official Langfuse documentation, see [README.md](./README.md).

This guide explains how to set up Langfuse for LLM observability in the TD.AIBIPrototype project. Langfuse provides tracking of traces, costs, and performance metrics for your LangChain API.

## Prerequisites

- Docker and Docker Compose installed
- PostgreSQL (if running locally, you'll need to change the port mapping)
- Access to the `TD.AIBIPrototype.Langfuse` folder (this is a Git submodule)

## Setup Steps

### 1. Navigate to Langfuse Directory

```bash
cd TD.AIBIPrototype.Langfuse
```

### 2. Update PostgreSQL Port in `docker-compose.yml` ⚠️ IMPORTANT

**This step is critical if you have a local PostgreSQL running on port 5432.**

If you have a local PostgreSQL running on port 5432, you **MUST** change the port mapping to avoid conflicts:

1. Open `docker-compose.yml` in the `TD.AIBIPrototype.Langfuse` directory
2. Find the PostgreSQL service (around line 160)
3. Change the port mapping from:
   ```yaml
   ports:
     - 5432:5432
   ```
   
   To:
   ```yaml
   ports:
     - 127.0.0.1:5433:5432  # Changed from 5432:5432
   ```

This maps Langfuse's PostgreSQL to port 5433 on your host machine, leaving port 5432 free for your local PostgreSQL.

**Note:** If you don't have a local PostgreSQL, you can skip this step, but it's recommended to use port 5433 to avoid future conflicts.

### 3. Create `.env` File in Langfuse Directory ⚠️ REQUIRED

**You must create a `.env` file** in the `TD.AIBIPrototype.Langfuse` directory before starting Docker Compose.

1. Navigate to the `TD.AIBIPrototype.Langfuse` directory
2. Create a new file named `.env` (no extension)
3. Add the following content with initial user, organization, and project setup:

```env
LANGFUSE_INIT_ORG_ID=a7f3c9e2-8b1d-4a6f-9c2e-5d8b3a1f7e4c
LANGFUSE_INIT_ORG_NAME=Tech Development
LANGFUSE_INIT_PROJECT_ID=project-aibi-2024
LANGFUSE_INIT_PROJECT_NAME=AIBIPrototype
LANGFUSE_INIT_USER_EMAIL=admin@example.com
LANGFUSE_INIT_USER_NAME=Admin
LANGFUSE_INIT_USER_PASSWORD=adminadmin
```

**Important Notes:** 
- **File Location:** The `.env` file must be in `TD.AIBIPrototype.Langfuse/` directory (same level as `docker-compose.yml`)
- **Initial User:** This creates an initial user so you don't need to sign up manually when you first access Langfuse UI
- **Security:** Change the credentials (especially `LANGFUSE_INIT_USER_PASSWORD`) to secure values in production
- **Git Ignored:** The `.env` file is typically gitignored, so it won't be committed to the repository (this is intentional for security)
- **Required:** Without this file, Langfuse may not initialize properly with your custom organization and project settings

### 4. Start Langfuse with Docker Compose

```bash
docker compose up -d
```

Wait for all services to be healthy. You can check the status with:

```bash
docker compose ps
```

All services should show as "healthy" or "running" before proceeding.

### 5. Access Langfuse UI and Create API Keys

1. **Open Langfuse UI:**
   - Navigate to `http://localhost:3000` in your browser

2. **Log in:**
   - Use the credentials from step 3:
     - Email: `admin@example.com` (or the email you set)
     - Password: `adminadmin` (or the password you set)

3. **Create API Keys:**
   - Navigate to **Settings → API Keys**
   - Create a new API key (or use existing one)
   - Copy both the **Public Key** (starts with `pk-lf-...`) and **Secret Key** (starts with `sk-lf-...`)

### 6. Configure LangChain API

Add Langfuse credentials to your LangChain project's `.env` file:

Navigate to `TD.AIBIPrototype.LangChainApi` and add to your `.env` file:

```env
LANGFUSE_PUBLIC_KEY="pk-lf-..."
LANGFUSE_SECRET_KEY="sk-lf-..."
LANGFUSE_HOST="http://localhost:3000"
```

Replace the placeholder values with the actual keys you copied in step 5.

## Viewing Traces

Once configured, you can view your LLM traces:

1. **Access Langfuse Dashboard:**
   - Open `http://localhost:3000` in your browser
   - Log in with your credentials

2. **View Traces:**
   - Navigate to **Observability → Tracing** to see your LLM traces
   - View model costs, token usage, and performance metrics on the **Home** dashboard

3. **Monitor Usage:**
   - Track token consumption per model
   - Monitor costs in real-time
   - Analyze performance metrics

## Important Notes

- **API Keys Persist:** API keys are stored in the PostgreSQL volume, so they persist across Docker container restarts
- **One-Time Setup:** You only need to get the API keys once and save them in your LangChain project's `.env` file
- **Automatic Tracking:** The callback handler is already integrated in the LangChain code and will automatically capture all LangChain agent interactions
- **Cost Calculation:** Langfuse automatically calculates model costs based on token usage
- **Submodule Updates:** If you update the Langfuse submodule, your `.env` file and custom configurations will remain intact

## Troubleshooting

### Port Conflict Error

**Problem:** Docker fails to start because port 5432 is already in use.

**Solution:** 
1. Make sure you completed **Step 2** above and changed the PostgreSQL port mapping in `docker-compose.yml`
2. Change from `5432:5432` to `127.0.0.1:5433:5432`
3. If you already started Docker, stop it first:
   ```bash
   docker compose down
   ```
4. Then start again:
   ```bash
   docker compose up -d
   ```

### Import Errors in LangChain API

**Problem:** `Import "langfuse" could not be resolved` in your IDE.

**Solution:**
1. Ensure `langfuse` is installed in your virtual environment:
   ```bash
   cd TD.AIBIPrototype.LangChainApi
   python -m pip install langfuse
   ```
2. Verify VS Code is using the correct Python interpreter:
   - Command Palette (`Ctrl+Shift+P`) → **Python: Select Interpreter**
   - Choose the interpreter that matches your virtual environment

### No Traces Showing

**Problem:** Traces are not appearing in the Langfuse dashboard.

**Solution:**
1. Verify your API keys are correct in the LangChain project's `.env` file
2. Check that the Langfuse container is running:
   ```bash
   cd TD.AIBIPrototype.Langfuse
   docker compose ps
   ```
3. Ensure `LANGFUSE_HOST` is set to `http://localhost:3000` in your LangChain `.env`
4. Check the LangChain API logs for any connection errors

### Docker Container Issues

**Problem:** Containers are not starting or showing as unhealthy.

**Solution:**
1. Check container logs:
   ```bash
   docker compose logs
   ```
2. Restart containers:
   ```bash
   docker compose down
   docker compose up -d
   ```
3. If issues persist, try rebuilding:
   ```bash
   docker compose down -v  # Removes volumes (⚠️ This will delete data)
   docker compose up -d --build
   ```

## Stopping Langfuse

To stop Langfuse services:

```bash
cd TD.AIBIPrototype.Langfuse
docker compose down
```

To stop and remove volumes (⚠️ This will delete all data):

```bash
docker compose down -v
```

## Updating Langfuse

When you want to update Langfuse to the latest version, see the main [README.md](../README.md) for instructions on updating Git submodules.

