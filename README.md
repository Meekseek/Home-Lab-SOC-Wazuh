# Home Lab: Virtual SOC, SIEM Deployment & Attack Detection

## Project Overview
I built this virtual lab to simulate a Security Operations Center (SOC) environment. My main goal was to put foundational cybersecurity theory into practice by engineering a functional logging pipeline, executing a simulated attack, and analyzing the resulting telemetry. This project provided hands-on experience connecting a Linux SIEM (Wazuh) to a target endpoint and troubleshooting network visibility gaps.

## Tools & Environment
* **Hypervisor:** Proxmox VE (Running on bare metal)
* **SIEM / Manager:** Ubuntu Server 22.04 LTS (hosting Wazuh)
* **Attacker / Agent:** Kali Linux
* **Attack Tool:** THC-Hydra

## Technical Steps Executed
1. **The Infrastructure:** Set up two isolated virtual machines in Proxmox. Configured the networking so the Ubuntu server could monitor the Kali machine while keeping the lab isolated from my primary home network.
2. **The Agent Installation:** Utilized the Linux CLI to add the Wazuh repository to Kali's source list, installed the agent via `apt`, and reloaded the system daemon to successfully initiate the `wazuh-agent` service.
3. **Attack Simulation:** Executed a dictionary-based SSH brute-force attack from Kali Linux against the Ubuntu server using Hydra and the `rockyou.txt` wordlist.

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
