# Day 19: ICS/Modbus — Claus for Concern

**Date:** December 19, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★★ *(Official rating: Medium — rated harder; ICS/SCADA and Python scripting against live protocols were both completely new)*  
**Category:** ICS / SCADA / OT Security / Protocol Analysis  
**Room:** https://tryhackme.com/room/ICS-modbus-aoc2025-g3m6n9b1v4

---

## Overview

TBFC's drone delivery system is shipping Easter eggs instead of Christmas presents.
The conveyor belts and robotic arms are fully operational — the system isn't broken,
it's been manipulated. King Malhare bypassed the web interface entirely and hit the
PLC directly over Modbus TCP, changing the package type register. The web dashboard
shows everything as normal because it reads from the same manipulated values.

The task was to run a reconnaissance script against the PLC, read the register map,
identify the compromised values, and remediate — in the correct order, because the
attacker left a trap.

First time touching ICS, SCADA, PLCs, or any industrial protocol.

---

## What I Learned

### ICS, SCADA, and PLCs

**ICS (Industrial Control Systems):** Systems that control industrial physical
processes — factories, power plants, water treatment, pipelines. Security priority
is safety and uptime first. Confidentiality is secondary. An outage or physical
accident is worse than a data breach.

**SCADA (Supervisory Control and Data Acquisition):** The command center. Provides
the human interface — dashboards, alarms, trending graphs. SCADA monitors and sends
commands to the devices doing the actual work. TBFC uses SCADA to monitor its entire
drone delivery operation.

**PLC (Programmable Logic Controller):** A rugged industrial computer that runs
automation logic 24/7 and directly controls physical equipment. Reads sensors and
drives actuators (motors, valves, robotic arms). In TBFC's case, the PLC decides
what type of package to load and which delivery zone to send it to. This is the
device King Malhare actually compromised.

**IT vs. OT security:**

| | IT Security | OT/ICS Security |
|---|---|---|
| Priority | Confidentiality | Safety and uptime |
| Acceptable downtime | Patching windows | Near zero |
| Failure consequence | Data loss | Physical damage / safety incident |
| Update cycle | Frequent | Rare (years) |

### Modbus Protocol

One of the oldest industrial protocols, originally designed for serial communication
(Modbus RTU), now commonly used over TCP/IP (Modbus/TCP) on **port 502**.

**Client/Server model:** The client (SCADA system, script, or attacker) sends
requests. The server (PLC) responds.

**The security problem:** Modbus has no authentication, no encryption, and no
integrity checking. If you can reach port 502, you can read and write values without
proving your identity. There is no login prompt — the PLC just responds to requests.

This was not a design oversight. Modbus was designed for isolated, trusted industrial
networks. The problem is those networks are no longer isolated.

**FrostyGoop:** Real-world ICS malware discovered in early 2024 that directly
interfaces with industrial control systems via Modbus TCP port 502, enabling
arbitrary reads and writes to device registers. King Malhare used the same approach.

### Modbus Data Types

Modbus organizes data into four types. The writable vs. read-only distinction
matters for understanding what an attacker can and cannot change.

| Type | Abbreviation | Data | Access |
|---|---|---|---|
| Coils | C | Boolean (True/False) | Read/Write |
| Discrete Inputs | DI | Boolean (True/False) | Read-only |
| Holding Registers | HR | 16-bit integer | Read/Write |
| Input Registers | IR | 16-bit integer | Read-only |

Coils and Holding Registers are writable — an attacker with Modbus access can
change them to manipulate the physical process. Discrete Inputs and Input Registers
reflect sensor readings that can only be observed.

**Critical detail:** Modbus addresses start at 0, not 1. HR0 is address 0,
C10 is address 10.

### Reconnaissance

**Nmap scan:**
```bash
nmap -sV -p 22,80,502 MACHINE_IP
```

**Open ports:**
```
22/tcp   SSH
80/tcp   HTTP (web interface + CCTV feed)
502/tcp  Modbus TCP
```

Checking `http://MACHINE_IP` shows the warehouse CCTV: robotic arms working,
conveyor belts running — delivering Easter eggs. The system is functioning
perfectly. It's been told to ship the wrong thing.

**Run the reconnaissance script:**
```bash
python3 reconnaissance.py
```

**Output:**
```
HOLDING REGISTERS:
HR0 (Package Type):     1    0=Christmas, 1=Eggs, 2=Baskets
HR1 (Delivery Zone):    5    1-9=Normal zones, 10=Ocean dump
HR4 (System Signature): 666  WARNING: Eggsploit signature detected

COILS (Boolean Flags):
C10 (Inventory Verification): False  [Should be True]
C11 (Protection/Override):    True   ACTIVE — system monitoring for changes
C12 (Emergency Dump):         False
C13 (Audit Logging):          False  [Should be True]
C14 (Christmas Restored):     False  [Auto-set when system is fixed]
C15 (Self-Destruct Armed):    False
```

HR0=1 confirms it — the PLC is set to ship eggs. HR4=666 is the Eggsploit attack
signature. C10 and C13 are disabled — inventory verification and audit logging
both silenced. C11 is the one to watch.

### The Trap: C11

**From the crumpled note found near the PLC terminal:**
```
CRITICAL: Never change HR0 while C11 = True — will trigger countdown!
```

C11 is an override flag left active by the attacker. If you correct HR0 (package
type) while C11 is still True, it triggers a countdown that locks the system. The
attacker anticipated incident response and built a counter-measure into the compromise.

**Correct remediation order:**
```
1. Disable C11 first  ← disarm the trap
2. Fix HR0            ← restore Christmas packages
3. Restore C10        ← re-enable inventory verification
4. Restore C13        ← re-enable audit logging
5. C14 auto-sets to True when the system recognizes the fix
```

Skipping step 1 and going straight to HR0 triggers the trap. This is a deliberate
attacker TTP — anticipating blue team response and building resistance into the
compromise.

### Python with pymodbus

**Library:** pymodbus 3.6.8  
**Install (if not on AttackBox):** `pip3 install pymodbus==3.6.8`

```python
from pymodbus.client import ModbusTcpClient

# Connect
client = ModbusTcpClient('MACHINE_IP', port=502)
client.connect()

# Read holding registers (HR0 through HR4)
result = client.read_holding_registers(address=0, count=5, slave=1)
print(result.registers)
# Output: [1, 5, 0, 0, 666]

# Read coils (C10 through C15)
result = client.read_coils(address=10, count=6, slave=1)
print(result.bits)
# Output: [False, True, False, False, False, False]

# Step 1: Disable C11 (disarm trap)
client.write_coil(address=11, value=False, slave=1)

# Step 2: Fix package type
client.write_register(address=0, value=0, slave=1)

# Steps 3-4: Restore monitoring
client.write_coil(address=10, value=True, slave=1)
client.write_coil(address=13, value=True, slave=1)

client.close()
```

**pymodbus 3.x note:** The parameter is `slave=1`, not `unit=1`. Using `unit=1`
is pymodbus 2.x syntax and will throw a deprecation error on 3.6.8.

---

## Challenges

Entirely new domain. The conceptual jump from IT security (protecting data) to OT
security (protecting physical processes) required rebuilding the mental model from
scratch. A compromised register in a PLC is not a data leak — it is machinery
doing the wrong thing in the physical world.

Python was the other wall. Prior experience was running scripts, not modifying
them. Understanding the pymodbus client object, reading register output arrays,
and knowing which function applies to coils vs. holding registers took time.
The `slave=1` vs. `unit=1` distinction was a practical trip-up.

The C11 trap mechanic was the most interesting part — the attacker anticipated
incident response and built a counter-measure into the compromise. That level of
operational thinking is worth internalizing.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** ICS/OT
vulnerabilities, unauthenticated protocol access, physical consequences of cyber
attacks.

**Domain 3.0 - Security Architecture (18%):** IT/OT network segmentation, attack
surface of industrial environments, legacy protocol risks.

---

## Evidence

![Reconnaissance Output](../07-Screenshots/Day19-1.png)
*reconnaissance.py output: HR0=1 (Eggs), HR4=666 (Eggsploit signature), C11=True
(trap active), C10 and C13 disabled — full compromise picture confirmed.*

![Remediation Sequence](../07-Screenshots/Day19-2.png)
*Modbus write sequence executed in correct order: C11 disabled first, HR0 corrected
to 0, C10 and C13 restored — C14 auto-set to True confirming successful remediation.*

---

## Key Takeaways

**Modbus data types:**

| Type | Access | Use |
|---|---|---|
| Coils (C) | Read/Write | Boolean control flags |
| Discrete Inputs (DI) | Read-only | Boolean sensor readings |
| Holding Registers (HR) | Read/Write | Integer process setpoints |
| Input Registers (IR) | Read-only | Integer sensor readings |

**TBFC register map:**

| Address | Name | Values | Compromised State |
|---|---|---|---|
| HR0 | Package Type | 0=Christmas, 1=Eggs, 2=Baskets | 1 (Eggs) |
| HR1 | Delivery Zone | 1-9=Normal, 10=Ocean dump | 5 |
| HR4 | System Signature | 666=Eggsploit | 666 |
| C10 | Inventory Verification | True=Active | False |
| C11 | Protection/Override | True=Trap armed | True |
| C12 | Emergency Dump | False=Safe | False |
| C13 | Audit Logging | True=Active | False |
| C14 | Christmas Restored | Auto-set on fix | False |
| C15 | Self-Destruct Armed | False=Safe | False |

**Remediation order:**
```
1. write_coil(11, False)    ← disarm C11 trap FIRST
2. write_register(0, 0)     ← restore Christmas packages
3. write_coil(10, True)     ← re-enable inventory verification
4. write_coil(13, True)     ← re-enable audit logging
```

**pymodbus quick reference:**
```python
from pymodbus.client import ModbusTcpClient

client = ModbusTcpClient('MACHINE_IP', port=502)
client.connect()

# Read
client.read_holding_registers(address=0, count=5, slave=1).registers
client.read_coils(address=10, count=6, slave=1).bits

# Write
client.write_register(address=0, value=0, slave=1)
client.write_coil(address=11, value=False, slave=1)

client.close()
```

**Key terms:**

| Term | Definition |
|---|---|
| ICS | Industrial Control Systems — systems managing physical industrial processes |
| SCADA | Supervisory Control and Data Acquisition — human operator interface for ICS |
| PLC | Programmable Logic Controller — rugged computer directly controlling physical equipment |
| Modbus TCP | Industrial protocol over TCP/IP; no authentication, no encryption; default port 502 |
| Holding Register | Writable 16-bit Modbus value storing process setpoints or configuration |
| Coil | Writable boolean Modbus flag for on/off control |
| FrostyGoop | 2024 ICS malware using Modbus TCP to manipulate industrial device registers |
| IT/OT convergence | Connecting OT networks to corporate/internet — defeats assumed isolation |
| Trap logic | Attacker-placed counter-measure that triggers if remediation is attempted in the wrong order |
| slave=1 | pymodbus 3.x parameter identifying the target device unit (replaces unit=1 from 2.x) |
