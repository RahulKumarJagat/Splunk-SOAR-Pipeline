## 🛡️ Splunk SOAR Pipeline: Advanced Threat Enrichment Lab (L3)

This project demonstrates an **Advanced (Level 3) Security Orchestration, Automation, and Response (SOAR)** solution. It integrates essential security and ticketing tools within a private Docker network, focusing on automated decision-making and real-time threat enrichment.

-----

## 💻 Project Architecture & Components

| Component | Role in Project | Access URL |
| :--- | :--- | :--- |
| **Splunk Enterprise** | SIEM / Log Source (Generates Alerts & Data) | http://localhost:8000 |
| **n8n** | SOAR Engine / Automation Orchestrator | http://localhost:5678 |
| **VirusTotal** | External Threat Intelligence (Real-time IP Scanning) | API Calls |
| **Jira Software** | Ticketing / Incident Triage (Final Action) | External Link |

-----

## ⚙️ Deployment & Installation

### Prerequisites

  * **Docker & Docker Compose** (Installed on your Linux Host).
  * A **VirusTotal API Key** and a **Jira API Token** (You will configure these in n8n later).

### Step 1: Create Files & Directory

1.  Create a fresh directory for your project and navigate into it:
    ```bash
    mkdir ~/soc-automation-lab
    cd ~/soc-automation-lab
    ```
2.  Create and save the `docker-compose.yml` file (content provided in the original prompt).
3.  Create two empty placeholder files for your exported n8n workflows:
    ```bash
    touch incident_response_workflow.json
    touch daily_report_workflow.json
    ```

### Step 2: Launch the Environment

Run this command in your project directory to create the private `soc-net` network and start the services:

```bash
docker-compose up -d
```

### Step 3: Initial Setup & Configuration

1.  **Access Splunk:** Go to http://localhost:8000. Log in as `admin` with the password set in the `.yml` file.
2.  **Access n8n:** Go to http://localhost:5678.
3.  **Import Workflows:** Import the two JSON files (`incident_response_workflow.json` and `daily_report_workflow.json`) into n8n.
4.  **Set Credentials:** Configure the required credentials for **Jira** and **VirusTotal** in n8n's Credentials panel.

-----

## 🌐 Workflow Deep Dive

### 1\. Incident Response Pipeline (Level 3 Enrichment)

This workflow runs in **real-time**, pulling in an alert from Splunk and using an external API (VirusTotal) to determine risk before creating a ticket.

| Step | Technical Action | Resume Value |
| :--- | :--- | :--- |
| **Webhook** | Receives JSON payload (`src_ip`) pushed by Splunk. | Real-time Ingestion |
| **VirusTotal** | Queries the Threat Intelligence API using the dynamically mapped IP (`{{ $json.src_ip }}`). | Data Enrichment |
| **IF Node** | Decision Logic: Checks if the Malicious Flag count is $> 5$. | Noise Reduction / SOAR |
| **Jira** | Creates a Critical Bug with the VT score and report link. | Automated Triage |

### 2\. Scheduled Reporting Pipeline

This is an automated **ETL (Extract, Transform, Load)** process for compliance and reporting data extraction.

| Step | Technical Action | Resume Value |
| :--- | :--- | :--- |
| **Schedule** | Triggers the workflow daily (e.g., 9:00 AM). | Time-based Automation |
| **Splunk Query** | Connects to the HTTPS Management API (8089) in Blocking Mode to ensure data integrity. | Secure API Extraction |
| **Convert to CSV** | Takes the structured JSON results and converts the output to a standard CSV file. | Data Transformation |
| **Write to Disk** | Saves the file to a temporary, writable location (`/tmp/`) inside the container. | Data Delivery |

-----

## 📸 Project Proof & Evidence

Please replace the following placeholders with actual screenshots from your executed lab.

### A. Workflow Diagrams ()

  * **incident response.png** Shows the full chain including the IF node.
  * **daily report.png** Shows the sequential ETL process.

### B. Splunk Alert Configuration

  * **splunk.png** Proof that the Splunk alert is correctly configured to use `makeresults` with the `src_ip` field and points to the n8n Webhook URL.

### C. Final Results

  * **jira.png** Proof that the automation successfully created a ticket and enriched it with external data.

  * **report.csv** The output of the daily reporting pipeline.

-----

## 🐳 Docker Compose File (`docker-compose.yml`)

Copy the following content into a file named `docker-compose.yml`. **Remember to replace YOUR\_PASSWORD\_HERE.**

```yaml
version: '3.7'
networks:
  soc-net:
    driver: bridge # Custom network for internal communication

services:
  # ----------------------------------------------------
  # 1. SPLUNK ENTERPRISE (SIEM)
  # ----------------------------------------------------
  splunk:
    image: splunk/splunk:latest
    container_name: splunk
    hostname: splunk
    environment:
      # Required new license flags for latest image
      - SPLUNK_GENERAL_TERMS=--accept-sgt-current-at-splunk-com
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=YOUR_PASSWORD_HERE # <-- REPLACE THIS
      - SPLUNK_USER=admin
    ports:
      - "8000:8000" # Web UI
      - "8089:8089" # Management API (Used by n8n)
    networks:
      - soc-net
    volumes:
      - splunk_data:/opt/splunk/var # Persistence
    
  # ----------------------------------------------------
  # 2. N8N AUTOMATION ENGINE (SOAR)
  # ----------------------------------------------------
  n8n:
    image: n8nio/n8n
    container_name: n8n
    hostname: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    networks:
      - soc-net
    environment:
      - N8N_HOST=n8n
      - N8N_PORT=5678
      # Enables user management and helps with volume permissions
      - N8N_USER_MANAGEMENT_ENABLED=true
    volumes:
      - n8n_data:/home/node/.n8n # Persistence

volumes:
  splunk_data:
  n8n_data:
```
