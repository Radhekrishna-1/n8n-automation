# Container Engineering Automation Architecture

This document outlines the architecture, data flow, and required files for automating container engineering tasks using an AI-powered n8n workflow (`n8n-02`).

## 1. Flow Diagram

The following diagram illustrates the lifecycle of a request interacting with the AI Agent in the n8n environment:

```mermaid
graph TD
    A[Trigger Event] -->|User Request / Webhook| B(AI Agent Node)
    
    subgraph n8n Orchestrator
        B -->|System Persona Rules| C{Deployment Engineer Agent}
        C -->|1. Analyze Task| C
        C -->|2. Request File Output| D[Read File Tool]
        D -.->|Returns File Content| C
        
        C -->|3. Generate/Optimize| C
        
        C -->|4. Push Changes| E[Write File Tool]
        E -.->|Status (Success/Fail)| C
        
        C -->|5. Validate Deploy| F[Execute Command Tool]
        F -.->|Build/Test Log Output| C
    end
    
    C -->|6. Final Report / Output| G[End User Notification]
    
    classDef agent fill:#f5d0f5,stroke:#333,stroke-width:2px;
    class C agent;
```

## 2. Architecture Overview

- **n8n Orchestrator (`n8n-02`)**: Acts as the central hub. It manages the tools and node connections, routes the data, and connects the chat triggers to the agent.
- **AI Agent (The Brain)**: Built on an LLM (e.g., Claude 3.5 Sonnet / OpenAI). It executes the logic to modify container and orchestration files based on the `deployment-engineer.md` instructions.
- **The System Persona**: The specialized prompt (`deployment-engineer.md`) that forces the LLM to behave strictly as a Senior DevOps Engineer. It enforces rules like "Automation First," environment parity, and robust CI/CD practices.
- **Tools Nodes**: The "hands" of the agent. By attaching the "Read/Write Files from Disk" node as a tool, n8n grants the agent the capability to securely access your container config files on the host system.

## 3. Required Files for Automation

Below are the key files we are using to achieve this container automation within the `n8n-02` server.

### **A. Core Automation Definitions**
These are the files in this directory used to structure the AI execution workflow:

1. `/opt/ai/n8n/n8n-02/container-engineering/architecture.md`
   *(This reference documentation).*
2. `/opt/ai/n8n/n8n-02/container-engineering/deployment-engineer.md`
   *(The System Prompt/Persona. Its complete text must be pasted into the "System Message" box of the n8n AI Agent node.)*

### **B. Target Management Files (For Current Automation)**
These are the files your agent interacts with to containerize and manage your `n8n-02` environment. You must configure the **Read/Write Files** tools in n8n to give the agent access to these paths:

1. `/opt/ai/n8n/n8n-02/docker-compose.yml`
   *(The main orchestration file. The AI can read it, add services like Redis, and optimize it.)*
2. `/opt/ai/n8n/n8n-02/.env`
   *(Required environment variables. Crucial for adding `NODE_FUNCTION_ALLOW_EXTERNAL=*` or `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=false` to power the tools.)*

### **C. Optional / Future Target Files**
Once comfortable with the current setup, the automation can be expanded to create and manage:
- `/opt/ai/n8n/n8n-02/Dockerfile` (For optimized multi-stage custom image builds)
- `.github/workflows/deploy.yml` (For CI/CD GitHub action pipelines to automate the n8n application testing)

## 4. Setup Implementation Steps

1. Enable necessary execution permissions in `/opt/ai/n8n/n8n-02/.env`.
2. Open the `n8n-02` web interface and create a new workflow.
3. Bring in a **Chat Trigger** or **Webhook** node.
4. Add the **AI Agent** node.
5. In the **AI Agent** settings, set the Agent type to "Tools Agent".
6. Copy the contents of `deployment-engineer.md` and paste it into the "System Message" property of the AI Agent node.
7. Connect an LLM node (like Anthropic Claude) below the agent.
8. Add a `"Read/Write Files from disk"` Tool node to the right side of the Agent. This turns the persona into an actual deployment engineer capable of maintaining your `.yml` and `.env` files.
