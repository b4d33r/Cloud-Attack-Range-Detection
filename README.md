# Cloud Attack Range & Detection

A practical cloud-security lab to simulate SSRF-based cloud attacks and validate detections with Wazuh SIEM and AI-assisted analysis.

![Architecture](project-architecture.jpeg)

## Overview

This project demonstrates an end-to-end attack and defense workflow:

- Attack-chain simulation in a controlled environment
- Cloud telemetry collection and SIEM-based detection
- AI-assisted alert triage for SOC workflows

## Lab Topology

- Kali VM: attack execution
- Target VM: vulnerable API + LocalStack + mock IMDS
- SOC VM: Wazuh + AI services

## Quick Start

# Target VM
cd ~/cloud-target
docker compose up -d
cd terraform && terraform init && terraform apply -auto-approve

# SOC VM
cd ~/wazuh-rules && bash deploy-rules.sh
cd ~/ai-engine && docker compose up -d --build

# Kali VM
cd ~/kali-attack && bash setup_kali.sh
bash attack_chain.sh

## Selected Screenshots

![SSRF Endpoint](screenshots/03-ssrf-vulnerable-endpoint.png)
![Wazuh Alerts](screenshots/05-wazuh-alerts-discover.png)
![AI Dashboard](screenshots/07-ai-dashboard-alerts.png)

## Repository Structure

- `kali-attack/` attack automation scripts
- `securebank-api/` vulnerable API service
- `mock-imds/` metadata service simulator
- `cloudtrail-forwarder/` event forwarding service
- `wazuh-rules/` detection rules and decoders
- `ai-engine/` AI analysis backend
- `ai-panel/` SOC dashboard
- `terraform/` LocalStack infrastructure

## Note

Educational and defensive use in isolated lab environments only.
