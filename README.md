# Home Lab SOC with Wazuh

## Project Overview
I built this virtual lab to test out the basics of a Security Operations Centers (SOC). My main goal was to put theory into practice and get some hands-on experience connecting a Linux security manager (Wazuh) to a target endpoint.

## What I Built
* **Hypervisor** * Proxmox VE (Running on bare metal)
* **Manager** * Ubuntu Server (hosting Wazuh)
* **Agent** * Kali Linux (the target machine)

## Key Steps & Challenges
### 1. The Infrastructure
I set up two virtual machines in Proxmox. I had to configure the networking so the Ubuntu server could "talk" to the Kali machine while keeping them isolated from my main home network.

### 2. The Agent Installation
This was the hardest part for me. I had to work through how to use the Linux command line (CLI) to:
* Add the Wazuh repository to Kali's source list
*  Install the agent using 'apt'
*  Troubleshoot the 'wazuh-agent' service when it failed to start (required reloading the system daemon)

### 3. Verification
I verified the setup by checking the agent status in the Wazuh dashboard.
* **Status** Active
* **OS** Kali GNU/Linux
* **Manager IP** Verified connection to my Ubuntu server
  
<img width="696" height="324" alt="CLI_SOC" src="https://github.com/user-attachments/assets/abed3b68-d282-4715-84e3-5e3182f72000" />

