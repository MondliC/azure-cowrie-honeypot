
# Azure Cowrie Honeypot Step‑by‑Step Deployment Guide

This repository provides a complete, practical guide for deploying a **Cowrie SSH Honeypot on Microsoft Azure**, using **Ubuntu Server 24.04 LTS**, a custom SSH listener on **Port 2222**, and secure network rules.

Cowrie is a medium‑interaction honeypot designed to record brute‑force attempts, attacker behavior, and malicious commands. This deployment demonstrates cloud security engineering, threat intelligence collection, and network monitoring in a real Azure environment.

**There’s still more coming  including collecting real‑time data from lured attackers and connecting it to Microsoft Sentinel for automated threat intelligence, detection rules, analytics, and dashboards.**

---

## 📚 Project Contents

This repository includes:

- `deployment-guide.pdf`  
  A full step‑by‑step guide for deploying the Azure Cowrie Honeypot.
---

## 🚀 Deployment Overview

### 1️⃣ Create the Azure VM
- Open **Azure Portal → Virtual Machines → Create VM**  
- Select:
  - **Image:** Ubuntu Server 24.04 LTS  
  - **Size:** B1s  
  - **Authentication:** Password  
- Under **Networking → NSG inbound rules**, allow:
  - **Port 22** (optional for admin SSH access)
  - **Port 2222 (TCP)** — required for Cowrie honeypot listener  

### 2️⃣ SSH into Your VM
```bash ssh azureuser@<YOUR-PUBLIC-IP>

### 3️⃣ Update System + Install Dependencies
Update and upgrade your Ubuntu VM:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv libssl-dev libffi-dev build-essential


### 4️⃣ Clone the Cowrie Repository
git clone https://github.com/cowrie/cowrie

### 5️⃣ Add Inbound Rule for Port 2222
In Azure Portal:
Go to Virtual Machine → Networking
Under Inbound Security Rules, select Add
Configure:
Source: Any
Destination Port: 2222
Protocol: TCP
Action: Allow
Priority: (lowest safe number)
Save the rule.
Port 2222 is the SSH honeypot listener port that attackers will target.







