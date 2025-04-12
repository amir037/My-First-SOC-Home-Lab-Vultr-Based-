# 🏠 My First SOC Home Lab (Vultr-Based)

This is my first go at building a home SOC (Security Operations Center) lab — hosted entirely on Vultr. It’s a hybrid blue/red team environment where I can mess around with logging, detection, C2 frameworks, and incident tracking. All in the name of learning and leveling up 🔧💻🔐

---

## 🧠 Lab Purpose

The goal of this lab is to:

- Simulate a basic enterprise environment
- Monitor real-time logs using Elastic & Kibana
- Practice threat detection with Elastic Agent + Fleet
- Simulate attacks using Kali + Mythic C2
- Track and manage alerts via osTicket
- Train like a SOC analyst in a safe cloud lab

---

## 🏗️ Lab Infrastructure

### 🌐 Network

- **Private Network:** `172.31.0.0/24`
- **Subnet Mask:** `255.255.255.0`
- **Available IPs:** `172.31.0.1 - 172.31.0.254`

### 🔹 Components

| Role                | OS / App            | Notes                            |
|---------------------|---------------------|----------------------------------|
| Elastic Stack       | Elastic + Kibana    | Main log storage + dashboards    |
| Fleet Server        | Elastic Fleet       | Manages agents across endpoints  |
| Windows Server      | Windows (RDP)       | Simulated endpoint w/ monitoring |
| Ubuntu Server       | Ubuntu (SSH)        | Another endpoint (Linux)         |
| osTicket Server     | osTicket            | For handling alerts/tickets      |
| C2 Server           | Mythic              | Command & control server         |
| Attack Laptop       | Kali Linux          | Red team / offensive side        |
| Analyst Laptop      | Windows             | Used for SOC-level monitoring    |

---

## 🌐 Hosting

All machines are hosted on **[Vultr](https://www.vultr.com/)** using cloud instances.

---

## 🔁 Log Flow

- Logs from endpoints are forwarded via **Elastic Agent**
- Collected by the **Fleet Server**
- Shipped to **Elastic Stack** for analysis and visualization
- Alerts and tickets created in **osTicket**

---

## 🖼️ Lab Architecture Diagram

![SOC Lab Diagram](./amir.png)

---

## 🔓 Attack Simulation Flow

This lab includes a simulated attack chain that follows the attacker lifecycle from initial access to exfiltration:

1. **Initial Access** – RDP brute force from Kali to Windows Server  
2. **Discovery** – Using built-in commands like `whoami`, `net user`, etc.  
3. **Defense Evasion** – Disabling Windows Defender  
4. **Execution** – PowerShell IEX to load Mythic agent  
5. **Command & Control** – Persistent access established via Mythic C2  
6. **Exfiltration** – Fake `passwords.txt` file created and "stolen"

![Attack Simulation Diagram](./attack.png)

---

## 🛠️ Deployment Phase

Below is the deployment mapping with public and private IPs for each component.

| Component           | Public IP        | Private IP          |
|---------------------|------------------|---------------------|
| Elastic & Kibana    | `x.x.x.x`        | `172.31.x.x`        |
| Fleet Server        | `x.x.x.x`        | `172.31.x.x`        |
| Windows Server      | `x.x.x.x`        | `172.31.x.x`        |
| Ubuntu Server       | `x.x.x.x`        | `172.31.x.x`        |
| osTicket Server     | `x.x.x.x`        | `172.31.x.x`        |
| Mythic C2 Server    | `x.x.x.x`        | `172.31.x.x`        |

### 🔧 Step 1: Create the Virtual Private Cloud (VPC)

The first step is to create a **VPC (Virtual Private Cloud)** that connects all lab components within a private address space.

- **VPC CIDR Block:** `172.31.0.0/24`
- **Subnet Mask:** `255.255.255.0`
- **Purpose:** Provides internal routing between lab components while still allowing controlled external access (e.g. SSH, RDP, web GUI).
- This VPC is created within Vultr’s platform and serves as the backbone of the lab environment.

  ![image](https://github.com/user-attachments/assets/520c30fe-7d50-444a-8e5e-b1a611e44fb6)


### 🖥️ Step 2: Deploy Lab Instances

Each virtual machine (VM) or server was deployed on Vultr with the required OS and specifications. All tools were installed by following their official documentation to ensure correct setup and updates.

| Component           | OS / Base Image     | Setup Guide |
|---------------------|---------------------|-------------|
| **Elastic Stack**   | Ubuntu 22.04        | [Elastic Docs](https://www.elastic.co/guide/en/elastic-stack/current/index.html) |
| **Fleet Server**    | Ubuntu 22.04        | [Fleet Server Setup](https://www.elastic.co/guide/en/fleet/current/fleet-server.html) |
| **Windows Server**  | Windows Server 2019 | Manual install + Elastic Agent |
| **Ubuntu Server**   | Ubuntu 22.04        | Manual install + Elastic Agent |
| **osTicket Server** | Ubuntu 22.04        | [osTicket Docs](https://docs.osticket.com/en/latest/) |
| **Mythic C2**       | Ubuntu 22.04        | [Mythic Docs](https://docs.mythic-c2.net/) |
| **Kali Linux**      | Kali Cloud Image    | [Kali Linux](https://www.kali.org/get-kali/) |
| **Analyst Laptop**  | Any OS              | No deployment needed (used locally) |

> ⚙️ All installations followed their respective official guides with minimal modifications to ensure compatibility and security.


