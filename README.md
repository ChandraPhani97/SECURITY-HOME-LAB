\# Security Home Lab



Hands-on home lab built from scratch to develop real security engineering skills — documented as I build it, alongside my CV.



\## Phase 0: Foundation Lab Environment



\### What I built

\- \*\*Hypervisor:\*\* VirtualBox 7.2, installed with Extension Pack

\- \*\*Kali Linux VM\*\* — attacker/tooling machine, Guest Additions installed, fully updated

\- \*\*Ubuntu Server 26.04 VM\*\* — target machine for practice

\- \*\*Isolated lab network\*\* — custom VirtualBox NAT Network (`NatNetwork1`, 10.0.2.0/24) connecting both VMs while keeping them separate from my home network

\- \*\*Docker Desktop + WSL2\*\* — for running containerized vulnerable apps and security tools

\- \*\*Git + GitHub\*\* — this repo, for tracking lab work over time



\### Network layout



```mermaid

graph LR

&#x20;   Internet((Internet))

&#x20;   Host\[Windows Host PC]

&#x20;   NAT\[NAT Network<br/>10.0.2.0/24]

&#x20;   Kali\[Kali Linux<br/>10.0.2.15<br/>attacker box]

&#x20;   Ubuntu\[Ubuntu Server<br/>10.0.2.3<br/>target box]



&#x20;   Internet --- Host

&#x20;   Host --- NAT

&#x20;   NAT --- Kali

&#x20;   NAT --- Ubuntu

```



\### Problems I hit and fixed

\- VirtualBox Network Manager UI was missing — created the NAT Network via `VBoxManage natnetwork add` CLI instead

\- Ubuntu installer hit a kernel hang during the `raid6` benchmark, caused by a VirtualBox/Hyper-V timing conflict — fixed with the `clearcpuid=avx2` boot parameter

\- A kernel Oops (Branch History Injection mitigation crash) on first real boot — fixed with `mitigations=off`, made permanent via `/etc/default/grub` + `update-grub`

\- Kali VM crashed once mid-install (resource contention) — resolved with a clean reset, did not recur



\### Verified working

\- Kali → Ubuntu ping test successful across the NAT Network

\- `docker run hello-world` successful (WSL2 + Docker Desktop confirmed working end-to-end)



\## Roadmap



This lab supports a structured, free-tier-only, Microsoft-stack-weighted learning path covering: Security Testing \& Vulnerability Management, Antivirus \& Endpoint Security, Network Scanning \& Penetration Testing, Identity \& Access Management (Entra ID), Cloud \& Infrastructure Security (Azure), Code Analysis \& Application Security, and Endpoint Detection \& Response (Microsoft Sentinel/Defender).



\*Last updated: 12 September 2026\*
## Phase 3: Active Directory & SIEM

- **Windows Server 2025 VM** — promoted to Active Directory Domain Controller, domain `securitylab.local`
- Integrated with Microsoft Sentinel via Azure Arc + AMA for centralized log collection (`SecurityEvent` table)
- Built and enabled a custom Kerberoasting detection analytics rule (MITRE ATT&CK T1558.003)

## Phase 1: Vulnerability Management

- Network reconnaissance (Nmap) + authenticated vulnerability scanning (Nessus Essentials) against the domain controller
- First written vulnerability assessment report: see `VA-Report-01-DC01-securitylab.md`
