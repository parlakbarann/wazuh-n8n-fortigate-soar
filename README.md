# 🛡️ Open Source SOAR: Fortigate Threat Intelligence with Wazuh & n8n

![Wazuh](https://img.shields.io/badge/Wazuh-4.x-blue?style=for-the-badge&logo=wazuh)
![n8n](https://img.shields.io/badge/n8n-Workflow-orange?style=for-the-badge&logo=n8n)
![Fortigate](https://img.shields.io/badge/Fortinet-Fortigate-red?style=for-the-badge&logo=fortinet)

## 📖 About The Project

This project provides a simple, open-source SOAR (Security Orchestration, Automation, and Response) architecture using **Fortigate Firewall**, **Wazuh SIEM**, and **n8n**.

It automatically fetches and analyzes malicious connections blocked by the Fortigate firewall. The n8n workflow periodically queries Wazuh for alerts related to Fortigate's deny actions (specifically `Rule ID: 81619`), enriches the data with threat intelligence from AbuseIPDB, and sends detailed alerts via Telegram.

### 🚀 Architecture Flow

1.  **Detection & Logging:** The Fortigate Firewall blocks a suspicious connection, and the log is forwarded to the Wazuh Manager.
2.  **Alert Generation:** Wazuh generates an alert based on the standard Fortigate rule `81619`.
3.  **Scheduled Fetch (n8n Workflow):**
    *   An n8n workflow runs on a schedule (e.g., every minute).
    *   It authenticates to the Wazuh API and queries for new alerts matching `rule.id: 81619`.
4.  **Enrichment & Alerting (n8n Workflow):**
    *   The workflow extracts the source and destination IP addresses from the Wazuh alert.
    *   It filters out private and whitelisted IP addresses.
    *   Public IPs are checked against the **AbuseIPDB** API for reputation.
    *   If an IP's abuse score is above a defined threshold (e.g., 50), a detailed intelligence report is sent to a security analyst via **Telegram**.
    *   Additionally, the workflow can detect if the log contains "nmap" and send a separate, immediate notification.

---

## ⚙️ Setup and Configuration

Follow these steps to get the SOAR workflow running.

### Step 1: Import and Configure the n8n Workflow

1.  **Import Workflow:** Import the `My workflow 2.json` file into your n8n instance.
2.  **Configure Credentials & Variables:** The workflow requires several pieces of information to be configured directly in the n8n nodes.

    *   **Wazuh Credentials:**
        *   In the **"IMPORTANT FIRST STEP!"** node, replace `YOURWAZUHUSERNAME` and `YOURWAZUHPASSWORD` with your actual Wazuh API credentials.
        *   In the **"HTTP Request"** node, update the URL `https://YOUR_WAZUH_HOST:9200` to point to your Wazuh server.

    *   **AbuseIPDB API Key:**
        *   In the **"AbuseIPDB Check"** node, go to the "Header" section to find the API key.
        > **⚠️ Security Warning:** The workflow includes a pre-filled, static API key. For security and proper functionality, you **must replace** this with your own AbuseIPDB API key. For better security practices, it is highly recommended to store the key as a credential or an environment variable in n8n instead of hardcoding it.

    *   **Telegram Configuration:**
        *   In the **"Send a text message"** and **"Send a text message1"** nodes, replace `YOUR_TELEGRAM_CHAT_ID` with the target Telegram chat ID.
        *   Create a "Telegram Bot API" credential in n8n and select it in both Telegram nodes.

### Step 2: Activate the Workflow

After configuring all the credentials and variables, **activate the workflow** in n8n. It will start running according to the schedule defined in the "Schedule Trigger" node.
