# ☁️🛡️ Cloud Attack Range & Detection

A realistic cloud-security range that simulates an SSRF-to-cloud-compromise attack chain and detects it with custom Wazuh rules plus AI-assisted SOC analysis.

![Architecture](project-architecture.jpeg)

## What this project does

This project reproduces a full offensive and defensive workflow across a multi-VM lab:

- Exploits an intentional SSRF flaw in a vulnerable banking API
- Abuses metadata access to steal temporary cloud credentials
- Uses those credentials for cloud discovery and data exfiltration
- Detects each phase with custom Wazuh detections mapped to MITRE ATT&CK
- Enriches alerts with AI analysis and serves results in a SOC dashboard

## Lab architecture

| Node | Purpose | Main components |
|---|---|---|
| Kali VM | Attacker | `attack_chain.sh`, `curl`, `jq`, `aws` CLI |
| Target VM | Victim cloud stack | SecureBank API, LocalStack, mock IMDS, CloudTrail forwarder |
| SOC VM | Detection & analysis | Wazuh, AI engine, PostgreSQL, AI panel |

## Attack chain implemented

The attack script in `kali-attack/attack_chain.sh` performs a practical sequence:

1. Recon of API endpoints and health checks
2. Authentication attempts (failed + successful) to generate signal
3. SSRF probing via `/api/payments/verify-gateway`
4. IMDS access and temporary credential theft
5. Cloud enumeration (S3, DynamoDB)
6. DynamoDB data exfiltration (`Scan`)

## Detection engineering (Wazuh)

Custom rules in `wazuh-rules/cloudtrail_rules.xml` cover the chain end-to-end:

- SSRF attempts and critical IMDS targeting (`100111`, `100110`)
- Login failures and brute-force behavior (`100121`, `100122`)
- IAM abuse and privilege escalation indicators (`100130`–`100133`)
- DynamoDB exfiltration behaviors (`100140`–`100142`)
- Correlated attack-chain alerting (`100170`)

## Telemetry pipeline

`cloudtrail-forwarder/cloudtrail_forwarder.py` bridges cloud activity into SIEM:

- Polls LocalStack CloudTrail logs from S3
- Parses and writes normalized one-line JSON events
- Converts SecureBank API runtime logs into CloudTrail-style events
- Outputs to `/var/log/cloudtrail/cloudtrail.log` for Wazuh ingestion

## AI analysis and SOC panel

### AI Engine (`ai-engine/ai_engine.py`)

- Polls Wazuh indexer for new alerts (`rule.level >= 6`)
- Processes custom rule space (`100xxx`)
- Uses Gemini when available, with local fallback analysis templates
- Stores enriched insights in PostgreSQL (`ai_insights` table)

### AI Panel (`ai-panel/server.py`)

- Flask API serving insights and stats endpoints
- Timeline, severity, top-rules, top-IP, and MITRE distribution views
- Security chat assistant via OpenRouter free models with Gemini fallback

## Stack and services

- Wazuh SIEM (manager/indexer/dashboard)
- LocalStack + Terraform
- Node.js (SecureBank API)
- Python (Forwarder, AI Engine, AI Panel)
- PostgreSQL 15
- Docker / Docker Compose

## Screenshots

### Application and exploit surface

![SecureBank Login](screenshots/01-securebank-login.png)
![SSRF Endpoint](screenshots/03-ssrf-vulnerable-endpoint.png)

### Detection and investigation

![Wazuh Overview](screenshots/04-wazuh-overview.png)
![Wazuh Alerts](screenshots/05-wazuh-alerts-discover.png)
![SSRF Alert Detail](screenshots/06-wazuh-alert-detail-ssrf.png)

### AI SOC experience

![AI Dashboard](screenshots/07-ai-dashboard-alerts.png)
![AI Analytics](screenshots/09-ai-analytics.png)
![Security Assistant](screenshots/10-security-assistant.png)

### Attack execution evidence

![Attack Chain Recon](screenshots/11-attack-chain-recon.png)
![Attack Chain SSRF + Creds](screenshots/12-attack-chain-ssrf-creds.png)
![Attack Chain Exfiltration](screenshots/13-attack-chain-exfiltration.png)

## Repository structure

- `securebank-api/` vulnerable banking API with intentional SSRF endpoint
- `mock-imds/` metadata service simulator used in SSRF exploitation
- `cloudtrail-forwarder/` middleware from cloud/app logs to Wazuh input
- `wazuh-rules/` decoders and custom detection rules
- `ai-engine/` alert enrichment engine (Gemini + fallback)
- `ai-panel/` SOC dashboard and assistant
- `kali-attack/` automated attack scripts
- `terraform/` LocalStack resources (S3, IAM, DynamoDB, CloudTrail)
- `docker-compose.target.yml` target stack orchestration
- `docker-compose.ai.yml` SOC stack orchestration

## Important note

This project is for educational and defensive testing in isolated lab environments only.
