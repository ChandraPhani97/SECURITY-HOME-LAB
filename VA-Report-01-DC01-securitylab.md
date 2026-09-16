# Vulnerability Assessment Report — Domain Controller (DC01)

**Report #1 of 5 — Security Skills Home Lab Portfolio**
**Author:** Chandra Phani
**Date:** 16 September 2026
**Target:** `10.0.2.10` (`WIN-AU2M9H9G3D7`), domain `securitylab.local`
**Classification:** Internal lab — non-production, isolated network

---

## 1. Executive Summary

This report documents a vulnerability assessment performed against a Windows Server 2025 domain controller in a self-built home lab, as hands-on practice for the vulnerability & exposure management lifecycle described in my current role. Testing combined authenticated vulnerability scanning (Nessus Essentials) with manual network reconnaissance (Nmap) to build a complete picture of the host's exposed attack surface, identify potential weaknesses, and record clear, actionable remediation guidance — the same structure I would use to report findings to a service owner who is not a security specialist.

*[Once Nessus results are added below: 1–2 sentences here summarizing the overall risk posture — e.g. "N critical, N high, N medium findings were identified, primarily relating to X" — fill in once section 4 is complete.]*

## 2. Scope

| Item | Detail |
|---|---|
| Target host | `10.0.2.10` / `WIN-AU2M9H9G3D7` |
| Role | Active Directory Domain Controller, domain `securitylab.local` |
| Network | Isolated lab NAT network (VirtualBox), no external exposure |
| Testing type | Unauthenticated network reconnaissance + authenticated vulnerability scan |
| Out of scope | Exploitation / active attack simulation (covered separately in detection-engineering work, not this assessment) |

## 3. Methodology

1. **Host discovery** — `nmap -sn 10.0.2.0/24` to enumerate live hosts on the lab network and confirm the target's presence before deeper testing.
2. **Port & service enumeration** — full TCP port scan with service/version detection and default NSE scripts: `nmap -sV -sC -p- 10.0.2.10`.
3. **Authenticated vulnerability scan** — Nessus Essentials, credentialed scan against the same host, to identify missing patches, misconfigurations, and known CVEs beyond what network-level scanning alone can reveal.

This two-tool approach mirrors real assessment practice: Nmap establishes *what* is exposed and how it's configured at the network level, while an authenticated scanner like Nessus checks *what's actually vulnerable* underneath — patch levels, weak configurations, and known CVEs a port scan alone can't see.

## 4. Findings

### 4.1 Network reconnaissance findings (Nmap)

| # | Finding | Severity | Evidence |
|---|---|---|---|
| N-1 | Host fingerprinted as a Domain Controller from an unauthenticated scan | Informational | Port combination unique to DCs: 88 (Kerberos), 389/3268 (LDAP/GC), 464 (kpasswd), 445 (SMB), 53 (DNS), 9389 (AD Web Services) — see full port list in Appendix A |
| N-2 | SMB message signing enabled and required | Positive control (no action needed) | `smb2-security-mode` script: `3.1.1: Message signing enabled and required` — mitigates SMB relay attacks |
| N-3 | WinRM (port 5985) exposed | Low–Medium (context-dependent) | Legitimate remote-management service, but a valid credential (e.g. from a phishing compromise or credential-stuffing) would allow remote code execution via this port. Recommend restricting WinRM access to a dedicated management subnet/jump host rather than leaving it reachable from the general network. |
| N-4 | Anonymous LDAP bind status not yet confirmed | To be tested | Follow-up action: test whether unauthenticated LDAP binds are permitted on port 389, which — if allowed — would let an unauthenticated attacker enumerate usernames, groups, and password policy. Common finding in real AD environments due to legacy configuration. |
| N-5 | Wide range of dynamic RPC ports exposed (49664–49710) | Informational | Standard Windows RPC endpoint mapper behavior, not a finding in isolation, but increases the surface available to enumeration tools (`rpcclient`, `enum4linux-ng`) if combined with other weaknesses |

### 4.2 Authenticated vulnerability scan findings (Nessus)

*[Add your Nessus scan results here. For each finding, use this structure so it's consistent and CV/portfolio-ready:]*

| # | Finding | Severity | CVE/Plugin ID | Remediation |
|---|---|---|---|---|
| *(fill in)* | | | | |

## 5. Overall Risk Assessment

*[Complete once section 4.2 is filled in — one short paragraph: does the combination of network exposure + Nessus findings represent a realistic attack path? E.g. "No critical remote-code-execution vulnerabilities were identified. The primary residual risk is credential-based lateral movement via WinRM, which is not itself a vulnerability but a configuration risk worth restricting."]*

## 6. Recommendations

1. Confirm and, if enabled, disable anonymous LDAP binds (see N-4).
2. Restrict WinRM (5985) access to a dedicated administrative subnet rather than general network reachability (see N-3).
3. *[Add recommendations arising from Nessus findings once added.]*
4. Re-scan after remediation to confirm findings are resolved (standard vulnerability-management lifecycle close-out step).

## Appendix A — Full Nmap Port List

```
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Global Catalog)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (WinRM)
9389/tcp  open  mc-nmf        .NET Message Framing (AD Web Services)
49664-49710/tcp open msrpc    Microsoft Windows RPC (dynamic range)
```

---
*Assessment performed in an isolated, self-owned home lab environment for skills-development purposes. No production systems were tested.*
