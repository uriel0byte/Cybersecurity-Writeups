# Incident Response Report: PoisonedCredentials (CyberDefenders)

## Scenario
Your organization's security team has detected a surge in suspicious network activity. There are concerns that LLMNR (Link-Local Multicast Name Resolution) and NBT-NS (NetBIOS Name Service) poisoning attacks may be occurring within your network. These attacks are known for exploiting these protocols to intercept network traffic and potentially compromise user credentials. Your task is to investigate the network logs and examine captured network traffic.

Analyze network traffic for LLMNR/NBT-NS poisoning attacks using Wireshark to identify the rogue machine, compromised accounts, and affected systems.

### 1. Executive Summary
*An attacker successfully used poisoning attacks using a rogue machine to intercept network traffic `LLMNR` and compromise user credentials. Network traffic (PCAP) was analyzed to identify the attacker's IP address and the rogue machine, the mistyped queries, the affected machines and the compromised accounts.*

#### 2. Incident Details
* **Lab/Challenge:** PoisonedCredentials (CyberDefenders)
* **Category:** Network Forensics / SOC Analyst Tier 1
* **Tools Used:** Wireshark
* **Date of Investigation:** 5-3-2026

### 3. Investigation Methodology & Findings
* **Step 1: Identifying the Attacker and the Affected Machines**
  * *Method:* The PCAP file was analyzed to determine the IP address of the rogue machine. `llmnr` filter was used in WireShark to isolate LLMNR traffic. Followed the initial query, focusing on response packets. Network analysis revealed the victim machine `192.168.232.162` sent a multicast LLMNR query to `224.0.0.252` looking for the mistyped share `fileshaare`. The response to the LLMNR query for `192.168.232.162` was `192.168.232.215`, which identified the rogue machine that initiated the poisoning attack. The attacker initiated their actions by taking advantage of benign network traffic from legitimate machines. The mistyped query `fileshaare` was identified in packet details in the `Queries` field. Sent from the `192.168.232.162` LLMNR queries. Also another affected machine `192.168.232.176` was identified during the same process with mistyped query `prinetr`. To understand the extent of the attacker's activities, the hostname of the machine that the attacker accessed via SMB was identified. `ntlmssp.challenge.target_info` filter was used to locate relevant packets. SMB2 header was examined, searching for the field `NetBIOS Computer Name` to identify the hostname `AccountingPC`.
   * *Finding:* Rogue Machine: `192.168.232.215`. Affected Machines: `192.168.232.162` (Hostname: `AccountingPC`) and `192.168.232.176` Mistyped Queries: `fileshaare` and `prinetr`

* **Step 2: Credential Theft**
  * *Method:* The username associated with the compromised account was identified by applying the `ntlmssp.auth.username` filter to locate packets containing NTLM authentication data. The username `janesmith` was found in the NTLM Secure Service Provider in the SMB Header of the SMB2 packet.
  * *Finding:* The compromised account: `janesmith`

### 4. Indicators of Compromise (IoCs)
* **Attacker IP Address(es):** `192.168.232.215`


### 5. Mitigation & Recommendations
* Disable LLMNR.
* Disable NetBIOS Name Service (NBT-NS).
* Enforce SMB Signing.
* Initiate a mandatory password reset for the compromised account (janesmith).
* Ensure Multi-Factor Authentication is turned on for all corporate accounts so that even if the attacker tries to use the stolen password, they are blocked by the MFA prompt (Enforce MFA).