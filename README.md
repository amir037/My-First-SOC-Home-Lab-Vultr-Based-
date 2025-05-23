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

<img width="1000" alt="image" src="./amir.png">

---

## 🔓 Attack Simulation Flow

This lab includes a simulated attack chain that follows the attacker lifecycle from initial access to exfiltration:

1. **Initial Access** – RDP brute force from Kali to Windows Server  
2. **Discovery** – Using built-in commands like `whoami`, `net user`, etc.  
3. **Defense Evasion** – Disabling Windows Defender  
4. **Execution** – PowerShell IEX to load Mythic agent  
5. **Command & Control** – Persistent access established via Mythic C2  
6. **Exfiltration** – Fake `secret-password.txt` file created and "stolen"

<img width="1000" alt="image" src="./attack.png">

---

## 🛠️ Deployment Phase

Below is the deployment mapping with public and private IPs for each component.

| Component           | Public IP        | Private IP          |
|---------------------|------------------|---------------------|
| Elastic & Kibana    | `95.179.252.168` | `172.31.0.3`        |
| Fleet Server        | `140.82.35.156`  | `172.31.0.4`        |
| Windows Server      | `45.32.154.93`   | `N/A`               |
| Ubuntu Server       | `80.240.23.65`   | `N/A`               |
| osTicket Server     | `95.179.165.50`  | `172.31.0.5`        |
| Mythic C2 Server    | `199.247.23.210` | `N/A`               |

### 🔧 Step 1: Create the Virtual Private Cloud (VPC)

The first step is to create a **VPC (Virtual Private Cloud)** that connects all lab components within a private address space.

- **VPC CIDR Block:** `172.31.0.0/24`
- **Subnet Mask:** `255.255.255.0`
- **Purpose:** Provides internal routing between lab components while still allowing controlled external access (e.g. SSH, RDP, web GUI).
- This VPC is created within Vultr’s platform and serves as the backbone of the lab environment.

  <img width="1000" alt="image" src="https://github.com/user-attachments/assets/520c30fe-7d50-444a-8e5e-b1a611e44fb6">


### 🖥️ Step 2: Deploy Lab Instances

Each virtual machine (VM) or server was deployed on Vultr with the required OS and specifications. All tools were installed by following their official documentation to ensure correct setup and updates.

| Component           | OS / Base Image     | Setup Guide |
|---------------------|---------------------|-------------|
| **Elastic Stack**   | Ubuntu 22.04        | [Elastic Docs](https://www.elastic.co/guide/en/elastic-stack/current/index.html) |
| **Fleet Server**    | Ubuntu 22.04        | [Fleet Server Setup](https://www.elastic.co/guide/en/fleet/current/fleet-server.html) |
| **Windows Server**  | Windows Server 2022 | Manual install + Elastic Agent |
| **Ubuntu Server**   | Ubuntu 22.04        | Manual install + Elastic Agent |
| **osTicket Server** | Windows Server 2022 | [osTicket Docs](https://docs.osticket.com/en/latest/) |
| **Mythic C2**       | Ubuntu 22.04        | [Mythic Docs](https://docs.mythic-c2.net/) |
| **Kali Linux**      | Kali Local VM       | [Kali Linux](https://www.kali.org/get-kali/) |
| **Analyst Laptop**  | Windows 11 (Local)  | No deployment needed (used locally) |

> ⚙️ All installations followed their respective official guides with minimal modifications to ensure compatibility and security.

---

### 📊 Step 3: Create Authentication Dashboards

After deploying and enrolling our Windows and Ubuntu servers into Fleet, we created Kibana dashboards to monitor authentication activity in real time.

#### 👀 What We're Tracking

- **Failed vs successful login attempts**
- **Source IP addresses**
- **Target usernames**
- **Login geolocation** 

#### 🪟 Windows Server (RDP)

- Data source: `winlogbeat` via Elastic Agent
- Visualizing:
  - `event.code: 4624` (Successful login)
  - `event.code: 4625` (Failed login)
  - `user.name` (username that attempted login)
  - `source.ip` (origin of the attempt)
  - Timestamps of login attempts

  ##### 🔍 RDP Failure Detection Query (KQL)

  ```kql
  event.code: 4625 AND agent.name: "My-SOC-Windows-Host" AND message:*mstsc*
  ```
  This filters failed login attempts triggered by the Microsoft RDP client (mstsc.exe) on the host named My-SOC-Windows-Host.

  🧠 Why not filter on LogonType: 10?
  In this lab setup, all failed RDP logins appear as LogonType: 3 instead of 10. This is likely due to how Windows logs pre-authentication RDP handshake failures — especially when using tools like xfreerdp. So     instead, we match on message:*mstsc* to detect RDP login attempts initiated via the Remote Desktop client.

  

  ##### ✅ RDP Success Detection Query (KQL)

```kql
event.code: 4624 AND agent.name: "My-SOC-Windows-Host" AND winlog.event_data.LogonType: 10
```
##### 🗺️ RDP Source IP Map Visualization

We also use  **Kibana Maps**  to display the geographic origin of RDP login attempts.



This map helps visualize suspicious login origins and detect possible brute force attempts from unexpected regions.


<img width="1000" alt="image" src="https://github.com/user-attachments/assets/681fc7df-442c-4ec1-8e76-1372feb4811a">

##### 📋 RDP Login Table Visualization

A **Data Table** visualization is used to display detailed RDP login activity.

This table gives a clear timeline of who attempted to connect, from where, and whether they were successful.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/69d38c0a-9c12-4f40-b386-c7a356858cfc">


#### 🐧 Ubuntu Server (SSH)

- Data source: `syslog` via Elastic Agent
- Visualizing:
  - `event.outcome` for success/failure
  - `user.name` (username that attempted login)
  - `source.ip` (origin of the attempt)
  - Timestamps of login attempts

##### 🔍 SSH Failure Detection Query (KQL)

```kql
agent.name :"My-SOC-Ubuntu" and event.category : "authentication"  and event.outcome : "failure" "
```

This query filters failed SSH login attempts on the host named `My-SOC-Ubuntu-Host`.

##### ✅ SSH Success Detection Query (KQL)

```kql
agent.name :"My-SOC-Ubuntu" and event.category : "authentication"  and event.outcome : "success" "
```

This query filters successful SSH login events on the same host.

##### 🗺️ SSH Source IP Map Visualization

We use a **Kibana Maps** visualization to plot geographic origins of SSH login attempts.

This visualization helps identify where login attempts are coming from and spot anomalies.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/116829df-9fc7-4b18-8f8a-631f09344a67">


##### 📋 SSH Login Table Visualization

A **Data Table** visualization gives a clear view into SSH login activity.

The table is useful for quick investigations and spotting brute force patterns or unusual user behavior.

<img width="1000" alt="image" src="https://github.com/user-attachments/assets/ee221def-1a7d-4424-ba26-0c033da5a3ad">



> 🔐 These dashboards provide visibility into brute force attempts, suspicious remote access, and login patterns across the environment.

---
## 🧨 Step 4: Simulate an Attack

This phase demonstrates a red team simulation using Kali Linux and Mythic C2. The attack follows a standard adversary kill chain and is designed to simulate common TTPs (Tactics, Techniques, and Procedures) observed in real-world incidents.

---

### 🔓 Phase 1: Initial Access

- The attacker (Kali) initiates a brute-force attack against the Windows Server over RDP.
- Tools used: `hydra` (for brute-force login) and `xfreerdp` (for interactive session testing).
- A weak password `Winter2024!` was intentionally configured to simulate poor password hygiene.
- After multiple failed attempts, the attacker successfully logs in using valid credentials.

  <img width="1000" alt="image" src="https://github.com/user-attachments/assets/4bc565a1-edbd-4922-bc11-4c38c9a9345d">


---

### 🔍 Phase 2: Discovery

- After gaining access via RDP, the attacker runs built-in commands for enumeration:
  - `whoami`
  - `ipconfig`
  - `net user`, `net group`
- These commands help the attacker gather information about users, network settings, and potential privilege escalation paths.

  <img width="1000" alt="image" src="https://github.com/user-attachments/assets/b00f309b-52e5-4a49-b963-e369db0c3044">


---

### 🛡️ Phase 3: Defense Evasion

- The attacker disables Windows Defender using PowerShell.
- This allows for unrestricted payload execution and simulates real-world bypass techniques.
- Disabling built-in defenses increases persistence and stealth.

  <img width="1000" alt="image" src="https://github.com/user-attachments/assets/dbef702d-1e44-4baf-8223-930a054d13f3">


---

### ⚙️ Phase 4: Execution

- A PowerShell command using `Invoke-Expression (IEX)` is used to execute a remote payload in memory.
- This payload loads a Mythic agent (Apollo) without touching disk, simulating a fileless malware technique.
- The script is obfuscated to avoid signature-based detection.

   <img width="1000" alt="image" src="https://github.com/user-attachments/assets/49e7ef89-071f-4930-9d66-990c4abe8959">

  <br><br>
   <img width="1000" alt="image" src="https://github.com/user-attachments/assets/eebfbf1b-07e5-4de6-b259-2c9d2b5aaa16">


---

### 🛰️ Phase 5: Command & Control (C2)

- The Mythic agent establishes a secure connection to the Mythic C2 server.
- The attacker gains full command-and-control over the compromised Windows Server.
- Actions include issuing tasks, retrieving host info, and preparing for lateral movement.

  <img width="1000" alt="image" src="https://github.com/user-attachments/assets/225e01cc-440f-4905-8559-07920d2827a4">



---

### 📤 Phase 6: Exfiltration

- A fake file (`secret-password.txt`) is created on the victim machine to simulate sensitive data.
- This file is exfiltrated through the Mythic agent using its built-in file transfer capabilities.
- Exfiltration happens over the encrypted C2 channel, simulating stealthy data theft.

   <img width="1000" alt="image" src="https://github.com/user-attachments/assets/27907c51-b4e7-4669-87b2-78d8b3cd105a">


---

## 🕵️ Step 5: Threat Hunting

This phase simulates a real-world blue team scenario where the SOC receives a report from a user — not a detection alert. The hunt begins with minimal context and evolves through investigation and correlation across logs.

---

### 📣 Incident Context

The SOC received a report from a user stating that a suspicious file named `secret-passwords.txt` appeared on their Desktop. No alerts were triggered. The analyst team was tasked with determining what happened, how, and when.

---

### 🗺️ Step 1: Initial Recon — Investigating Login Origins

- We started by reviewing **Kibana Maps** to visualize geographic origins of login attempts.
- RDP login attempts from unusual countries stood out immediately.
- Combined with the RDP login table, we were able to confirm that one of these attempts eventually succeeded — coming from an unfamiliar location.

  <img width="1000" alt="image" src="https://github.com/user-attachments/assets/3eef41ea-b960-4abf-a9b6-6d86f0c61de7">

  


---

### 🐚 Step 2: Investigate Remote Command Activity

- We pivoted to **Sysmon Event ID 11 (FileCreate)** to investigate file drops on the host, since PowerShell command-line logs did not show any results.
- We identified the creation of a suspicious file named `services.exe` in a user-accessible directory:
  -  path: `C:\Users\Administrator\Downloads\services.exe`
- This filename is often used to masquerade as a legitimate system binary, increasing the likelihood of user or system trust.
- The file creation timestamp aligned closely with the time of the user's report and RDP session.
- This strongly suggested that the attacker downloaded a payload to disk (likely via PowerShell) and prepared it for execution.

![image](https://github.com/user-attachments/assets/073096aa-ad56-482f-8823-a15dbd411b63)

---

### 🧱 Step 3: Check for AV/EDR Evasion

- We reviewed **Windows Defender Operational Logs**, specifically **Event ID 5001**, which tracks configuration changes to Defender settings.
- The logs showed that several protections were disabled, including:
  - Real-time monitoring
  - Behavior monitoring
  - Script scanning
- These changes occurred shortly after the attacker logged in via RDP and before the payload was executed.
- The use of `Set-MpPreference` was inferred based on the Defender configuration changes, though the actual command was not captured.
- This confirmed that the attacker took active steps to weaken endpoint defenses before launching their payload.

![image](https://github.com/user-attachments/assets/b2fbf7fb-e61a-4439-9788-6add6675a774)

---
### 📤 Step 4: Confirm File Creation (Exfiltration Simulation)

- We identified the creation of a suspicious file named `secret-passwords.txt` on the user’s Desktop.
- This file was not part of any known system or user activity and matched the filename reported by the client.
- The file was created shortly after the Defender protections were disabled and `services.exe` was dropped.
- This strongly suggested that the file was either created manually by the attacker or generated by a malicious process as part of the payload chain.
- While we do not have logs showing the file being exfiltrated via C2, the presence and timing of this file point to data staging and exfiltration intent.

![image](https://github.com/user-attachments/assets/43cf8de5-8838-451e-aa0b-8bc0661fa9be)

---

> 🧠 This threat hunt was conducted manually using Elastic dashboards, Maps, and Discover views. It mirrors how real SOCs triage user-reported incidents with limited initial context and piece together the full story through log correlation.
---
## 🚨 Step 6: Detection & Response

The final phase involves turning the threat hunt findings into actionable detections. Using Elastic Defend, we created a rule to catch similar activity in the future and tested automated response workflows to contain the threat and notify the analyst team.

---

### ✅ Detection Rule

We created a custom detection rule in Kibana to identify execution of the Mythic agent payload by monitoring process creation logs for a known malicious binary.

**Detection Query**
```kql
event.code :1 and ( winlog.event_data.OriginalFileName :"Apollo.exe" or winlog.event_data.Hashes:*SHA1=D61869B40EA81C8DCBABA4C80F960E140AA47098* )
```

This rule is highly specific and is designed to detect execution of a known Mythic agent on the host by matching both filename and SHA1 hash.

> 🔒 This minimizes false positives while ensuring that known attacker tools are flagged immediately if reused.

---

### 🛡️ Automated Response with Elastic Defend

Once the detection rule triggers:

- Elastic Defend automatically **kills the associated process** using Elastic's endpoint protection capabilities.
- The malicious process (`services.exe`) is terminated before execution or shortly after it begins.

> 🧪 We simulated this by manually triggering the rule and verifying that the endpoint responded in real-time.

![image](https://github.com/user-attachments/assets/8d5f394c-128b-4cef-b1aa-2ad7df10d8c3)

---

### 📬 Alerting via osTicket

To complete the response workflow:

- The alert generated by the detection rule is forwarded to our **osTicket server** using a webhook integration.
- A ticket is automatically created and assigned to the SOC queue, containing:
  - Hostname
  - Username
  - Timestamp
  - File path
  - Detection rule name

This ensures incidents are logged and tracked in a formal workflow system.


![image](https://github.com/user-attachments/assets/738430f4-06b2-49f3-807c-049e26b121fe)



---

> 📌 This phase ties detection to response — combining real-time prevention with alert triage and case tracking, just like a real SOC workflow.




