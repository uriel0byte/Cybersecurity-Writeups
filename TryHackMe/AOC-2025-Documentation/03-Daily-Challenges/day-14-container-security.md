# Day 14: Container Security — DoorDasher's Demise

**Date:** December 14, 2025  
**Time Spent:** 2 hours  
**Difficulty:** ★★★☆  
**Category:** Container Security / Docker / Privilege Escalation  
**Room:** https://tryhackme.com/room/container-security-aoc2025-z0x3v6n9m2

---

## Overview

King Malhare seized DoorDasher and rebranded it "Hopperoo." A recovery script
existed but the security engineer was locked out. The task was to break out of
a monitoring container by exploiting Docker socket access, escalate into a
privileged deployer container, and execute the recovery script to restore the
service. Container security and Docker were new — only heard the term in
Security+ without any depth.

---

## What I Learned

### What Are Containers?

Containers solve a specific problem: applications behave differently depending
on the environment they run in. Different OS versions, conflicting library
versions, configuration drift — containers eliminate this by packaging the
application and all its dependencies into one isolated unit.

Key properties:
```
Isolated    — each app runs in its own environment
Portable    — runs consistently across different systems
Lightweight — shares the host OS kernel, no full guest OS
Fast        — starts in seconds, not minutes
```

### Containers vs. Virtual Machines

The core difference is what gets isolated.

**Virtual machine:** Full guest OS on top of a hypervisor. Heavy, slow to
start, fully isolated at the kernel level. Use case: running multiple
different operating systems on one physical host.

**Container:** Application and dependencies only. Shares the host kernel.
Lightweight, starts in seconds. Use case: deploying scalable, portable
services.

```
VM:        Hardware → Hypervisor → Guest OS → App
Container: Hardware → Host OS → Container Engine → App
```

This is why containers became the default for microservices: they scale fast
because spinning up another container takes seconds and costs almost nothing
compared to spinning up another VM.

### Docker Architecture

Docker is the most common container engine. It has five components:

| Component | What It Is |
|---|---|
| Dockerfile | Text file — instructions for building an image |
| Docker Image | Built from Dockerfile — template for containers |
| Docker Container | Running instance of an image |
| Docker Daemon | Background service managing containers |
| Docker Client | CLI tools that send commands to the daemon |

**How they connect:**
```
Docker Client (docker commands)
        ↓
Docker Socket (/var/run/docker.sock)
        ↓
Docker Daemon (manages containers)
        ↓
Containers (running applications)
```

The Docker socket (`/var/run/docker.sock`) is a Unix socket file. It's the
API endpoint the daemon listens on. The client sends every command through it.

Container engines work by leveraging two Linux kernel features:
- **Namespaces** — provide isolation (each container gets its own view of the system)
- **cgroups** — control resource limits (CPU, memory, I/O per container)

### Docker Socket — The Security Risk

Access to the Docker socket = full control of the Docker daemon = ability to
create, inspect, and execute into any container on the host.

If the socket is mounted inside a container, that container can issue Docker
commands as if it were the host. This is the container escape vector.

**Why it gets mounted anyway:** Monitoring containers need to inspect other
containers' status. Deployment tools need to manage containers. The
convenience creates the risk.

**Rule:** Never mount `/var/run/docker.sock` into a container unless there is
no alternative. In production, this is almost always a misconfiguration.

**Enhanced Container Isolation** is a Docker feature that blocks containers
from mounting the socket. It exists specifically to prevent this attack class.

### Privileged Containers

A privileged container runs with elevated permissions — access to host
devices, ability to modify kernel parameters, ability to mount file systems.
Created with the `--privileged` flag.

```bash
docker run --privileged <image>
```

Normal containers have restricted access to host resources. Privileged ones
effectively have root-level access to the host. Deployment tools sometimes
require this. In production it should be tightly controlled and monitored.

**Risk:** If an attacker escapes into a privileged container, they have
near-complete host control.

### Container Escape Attack Chain

**Starting point:** Host machine (`mrbombastic` user)

**Step 1: Enumerate containers from host**
```bash
docker ps
```
Four containers running:
```
dasherapp        port 5001  — defaced DoorDasher site
uptime-checker   port 5003  — monitoring container
wareville-times  port 5002  — news site
deployer         (no port)  — privileged deployment container
```

**Step 2: Enter monitoring container**
```bash
docker exec -it uptime-checker sh
```

**Step 3: Check socket access**
```bash
ls -la /var/run/docker.sock
```
Socket is accessible from inside the container. Escape is possible.

**Step 4: Verify Docker control**
```bash
docker ps
```
Command works inside the container. The container has full Docker daemon
access — it can see and interact with every container on the host.

**Step 5: Escape into privileged container**
```bash
docker exec -it deployer bash
```
Moved from unprivileged monitoring container to privileged deployer container.

**Step 6: Verify and execute**
```bash
whoami                     # returns: deployer
sudo /recovery_script.sh   # restores DoorDasher
cat /flag.txt              # flag in deployer container root /
```
Recovery script executed, DoorDasher restored, flag captured.

**Bonus:** The `wareville-times` news site (port 5002) contains a hidden
secret code — the password for the `deployer` Linux user account. Exposed
credentials in a running container is a separate misconfiguration from the
socket issue but compounds the risk significantly.

**Full chain:**
```
Host (mrbombastic)
      ↓
docker exec uptime-checker (monitoring)
      ↓
socket access confirmed
      ↓
docker exec deployer (privileged)
      ↓
sudo recovery_script.sh
```

### Docker Commands

```bash
# List running containers
docker ps

# Execute a command in a running container
docker exec -it <container_name> <shell>
# -i  keep STDIN open (interactive)
# -t  allocate pseudo-TTY (terminal)

# Shell access by distro
docker exec -it uptime-checker sh    # Alpine Linux uses sh
docker exec -it deployer bash        # Ubuntu/Debian uses bash

# Check Docker socket
ls -la /var/run/docker.sock

# Check current user
whoami
```

### Dockerfile Basics

```dockerfile
FROM ubuntu:20.04               # Base image to build on
RUN apt-get update              # Run commands during build
COPY app.py /app/               # Copy files into image
ADD archive.tar.gz /app/        # Like COPY but also extracts archives
CMD ["python", "/app/app.py"]   # Default command when container starts
EXPOSE 8080                     # Document which port the app uses
```

`COPY` vs `ADD`: Both copy files into the image. `ADD` additionally supports
tar extraction and remote URLs. Prefer `COPY` unless you specifically need
those features.

`EXPOSE` does not publish the port — it's documentation only. Ports are
published at runtime with `-p`.

---

## Challenges

Container security was completely new. Docker commands, socket architecture,
daemon communication, and the escape technique all came at once. The offensive
angle (container escape, privilege escalation) is less natural than Blue Team
work, but understanding the attack is what makes the defensive monitoring
rules make sense. The hardest part was visualising the socket as the attack
surface — once that clicked, the rest of the chain followed logically.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Container
vulnerabilities, privilege escalation, escape techniques, misconfigurations.

**Domain 3.0 - Security Architecture (18%):** Virtualization security,
container isolation, microservices architecture, cloud infrastructure.

**Domain 4.0 - Security Operations (28%):** Security monitoring, incident
response, vulnerability assessment, hardening procedures.

---

## Evidence

![Docker Container Listing](../07-Screenshots/Day14-1.png)
*`docker ps` output showing all 4 containers: dasherapp (5001), uptime-checker
(5003), wareville-times (5002), and deployer.*

![Docker Socket Access Check](../07-Screenshots/Day14-2.png)
*Docker socket accessible from inside uptime-checker — container escape
confirmed.*

![Privilege Escalation to Deployer](../07-Screenshots/Day14-3.png)
*Escaped to deployer container, executed recovery script with sudo, restored
DoorDasher and captured flag.*

---

## Key Takeaways

**Docker commands:**
```bash
docker ps                           # List running containers
docker exec -it <name> <shell>      # Access container shell
docker exec -it <name> sh           # Alpine (sh)
docker exec -it <name> bash         # Ubuntu/Debian (bash)
ls -la /var/run/docker.sock         # Check socket access
whoami                              # Check current user
```

**Dockerfile instructions:**
```dockerfile
FROM    base image
RUN     command during build
COPY    copy files (basic)
ADD     copy files + extract archives + support URLs
CMD     default run command
EXPOSE  document port (does NOT publish it)
```

**Container vs. VM at a glance:**

| | Container | VM |
|---|---|---|
| Kernel | Shared with host | Own guest kernel |
| Startup | Seconds | Minutes |
| Size | MB | GB |
| Isolation | Process-level | Full OS-level |
| Use case | Microservices, scaling | Multiple OS environments |

**Blue Team — what to monitor:**
```
/var/run/docker.sock mounted into container   → HIGH RISK
Container running with --privileged           → HIGH RISK
Both together                                 → CRITICAL
Unexpected docker exec from inside container  → Investigate
Docker API calls from non-admin sources       → Investigate
```

**Hardening checklist:**
```
Never mount Docker socket unless no alternative
Enable Enhanced Container Isolation
Avoid --privileged in production
Run containers as non-root users
Monitor with runtime security tool (Falco)
Scan images regularly (Trivy, Docker Bench Security)
```

**Key terms:**

| Term | Definition |
|---|---|
| Container | Isolated environment packaging app and dependencies, shares host kernel |
| Docker Image | Built from Dockerfile — template for creating containers |
| Docker Daemon | Background service that manages containers |
| Docker Socket | `/var/run/docker.sock` — API endpoint daemon listens on |
| Container Escape | Breaking out of container isolation to access host or other containers |
| Privileged Container | Container with elevated host permissions (`--privileged`) |
| Hypervisor | Software layer enabling multiple VMs on one physical host |
| Microservices | Architecture pattern splitting an app into independently scalable services |
| Enhanced Container Isolation | Docker feature blocking containers from mounting the Docker socket |
| Namespaces | Linux kernel feature providing isolation — each container gets its own system view |
| cgroups | Linux kernel feature controlling resource limits per container (CPU, memory, I/O) |
| Falco | Runtime container security monitoring tool |
