# Home Lab: Virtual SOC, SIEM Deployment & Attack Detection

## Project Overview
I built this virtual lab to simulate a Security Operations Center (SOC) environment. My main goal was to put foundational cybersecurity theory into practice by engineering a functional logging pipeline, executing a simulated attack, and analyzing the resulting telemetry. This project provided hands-on experience connecting a Linux SIEM (Wazuh) to a target endpoint and troubleshooting network visibility gaps.

## Skills Demonstrated
* **Security Orchestration, Automation, and Response (SOAR):** Transitioning a SIEM from passive monitoring to automated active defense.
* **Incident Mitigation:** Dynamically deploying endpoint firewall (`iptables`/UFW) rules to isolate network threats.
* **Advanced Log Analysis:** Parsing complex JSON telemetry and SIEM execution logs (`wazuh-execd`) to verify automated system actions.
* **Endpoint Telemetry Troubleshooting:** Engineering custom logging pipelines (`rsyslog`) to bridge visibility gaps between endpoints and the SIEM.

## Tools & Environment
* **Hypervisor:** Proxmox VE (Running on bare metal)
* **SIEM / Manager:** Ubuntu Server 22.04 LTS (hosting Wazuh)
* **Attacker / Agent:** Kali Linux
* **Attack Tool:** THC-Hydra
* **Automation:** Wazuh Active Response module

## Technical Steps Executed
1. **The Infrastructure:** Set up two isolated virtual machines in Proxmox. Configured the networking so the Ubuntu server could monitor the Kali machine while keeping the lab isolated from my primary home network.
2. **The Agent Installation:** Utilized the Linux CLI to add the Wazuh repository to Kali's source list, installed the agent via `apt`, and reloaded the system daemon to successfully initiate the `wazuh-agent` service.
3. **Attack Simulation:** Executed a dictionary-based SSH brute-force attack from Kali Linux against the Ubuntu server using Hydra and the `rockyou.txt` wordlist.
4. **Automated Defense (SOAR):** Configured Wazuh's Active Response module to automatically deploy a firewall drop rule against the attacker's IP when the SSH brute-force alert threshold was met.

## Challenges & Troubleshooting (The "Struggle")
The most valuable part of this project was diagnosing and fixing the technical roadblocks that occurred during the attack simulation:

* **Agent Communication Failure:** After moving the server to a new network, DHCP assigned a new IP address to the Manager. The Kali agent immediately went offline. I had to use `nano` to edit the `/var/ossec/etc/ossec.conf` file on the endpoint to point to the new IP and restore the connection.
* **Service Rate Limiting:** During the initial SSH brute-force attack, Kali's built-in security dropped the connections (`[ERROR] all children were disabled`). I had to throttle the Hydra attack speed (`-t 1`) and use verbose mode (`-V`) to bypass the rate limit and successfully send the payloads.
* **The Log Visibility Gap:** The attack was successful, but the SIEM remained blind. By isolating the attack manually, I realized the Proxmox Ubuntu image was using `journald` but was missing the traditional `rsyslog` pipeline. The logs were never reaching `/var/log/auth.log` for Wazuh to read. I installed and configured the `rsyslog` service on the Ubuntu Manager, restarted the SIEM, and successfully bridged the visibility gap.

## Verification & Telemetry
By bypassing the cached dashboard widgets and utilizing the OpenSearch Discover tab, I was able to analyze the raw JSON logs. I successfully filtered the noise to identify the high-severity alerts (Rule Level 10), confirming the MITRE ATT&CK framework categorization (T1110 - Brute Force), the targeted service (`sshd`), and the originating attacker IP address.

![Wazuh_dash](https://github.com/user-attachments/assets/94329de3-fa1b-4ea7-90ad-49e88c2b5335)
<img width="696" height="324" alt="CLI_SOC" src="https://github.com/user-attachments/assets/5b0bd272-e484-488c-8c74-2ec033e567d6" />
<img width="1517" height="893" alt="Screenshot 2026-05-23 191121" src="https://github.com/user-attachments/assets/c7133c32-49a4-4e82-88c0-28b790db829a" />

---

## 🛡️ Automated Threat Mitigation (SOAR)
To elevate this environment from passive monitoring to active defense, I configured Wazuh's Active Response module to function as a basic SOAR system. 

* **The Trigger:** Configured `ossec.conf` to monitor for Rule ID 5710 (SSH Brute Force).
* **The Response:** Upon reaching the attack threshold, the Wazuh Manager dynamically injected a `firewall-drop` rule into the Ubuntu endpoint, blackholing the attacker's IP for 180 seconds.
* **Self-Healing:** After the 180-second timeout, the system automatically lifted the ban, restoring normal network routing.

**Verification:**
The automated network block was successful, and the response was verified within the SIEM via the `wazuh-execd` daemon logs.

<img width="1039" height="376" alt="Kali" src="https://github.com/user-attachments/assets/9f1294e9-7103-4d8d-89f4-efb37154c904" />

<img width="1511" height="809" alt="SIEM logs" src="https://github.com/user-attachments/assets/d02e7cc5-0c1f-4d56-b978-b71bb2bbf32f" />
