<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:8E2DE2,100:FF0066&height=220&section=header&text=Red%20Team%20Labs&fontSize=60&fontColor=ffffff&animation=fadeIn&fontAlignY=35&desc=Offensive%20Security%20%7C%20Exploit%20Development%20%7C%20Adversary%20Emulation&descAlignY=55&descSize=18" width="100%"/>

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=22&duration=3000&pause=800&color=FF0055&center=true&vCenter=true&width=700&lines=Break+it.+Understand+it.+Fix+it.;Hands-on+CVE+exploitation+labs;Every+vuln%2C+self-hosted+%26+reproducible;RCE+%7C+Privilege+Escalation+%7C+AD+Attacks" alt="Typing SVG" />

<br/>

![Labs](https://img.shields.io/badge/Labs-Growing-critical?style=for-the-badge&logo=hackthebox&logoColor=white)
![Focus](https://img.shields.io/badge/Focus-Offensive%20Security-red?style=for-the-badge&logo=metasploit&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge&logo=statuspage&logoColor=white)
![License](https://img.shields.io/badge/Use-Educational%20Only-yellow?style=for-the-badge&logo=readthedocs&logoColor=white)

</div>

---

## What This Is

A collection of **self-hosted, fully reproducible exploitation labs** — each one takes a real CVE
or attack technique from zero to confirmed impact, documented end-to-end: recon, exploit chain,
every bug hit along the way, and remediation. No copy-pasted exploit scripts without understanding
them — every lab here was built, broken, debugged, and fixed by hand.

<div align="center">
<img src="https://skillicons.dev/icons?i=linux,docker,bash,python,java,git&theme=dark" />
</div>

---

## Lab Index

| # | Lab | CVE / Technique | Status | Impact |
|---|---|---|:---:|---|
| 01 | [Log4Shell — Apache Solr RCE](./01-Log4Shell-CVE-2021-44228) | CVE-2021-44228 | ✅ Complete | Remote Code Execution (root) |
| 02 | [Spring4Shell — Tomcat/Spring MVC RCE](./02-Spring4Shell-CVE-2022-22965) | CVE-2022-22965 | ✅ Complete | Remote Code Execution (root) |
| 03 | [Fastjson — JSON Deserialization RCE](./03-Fastjson-CVE-2017-18349) | CVE-2017-18349 | ✅ Complete | Remote Code Execution (root) |
| 04 | [Struts2 — S2-045 RCE](./04-Struts2-S2-045-CVE-2017-5638) | CVE-2017-5638 | ✅ Complete | Remote Code Execution (root) |
| 05 | [Drupalgeddon2 — Drupal RCE](./05-Drupalgeddon2-CVE-2018-7600) | CVE-2018-7600 | ✅ Complete | Remote Code Execution (root) |

> Each lab folder is self-contained: `README.md` (full writeup), `screenshots/` (visual evidence),
> `raw-output/` (unedited terminal logs), and any PoC code used.

---

## Attack Surface Covered So Far

```mermaid
mindmap
  root((Red Team Labs))
    Web/App Layer
      Log4Shell RCE
      Deserialization
    Active Directory
      Kerberoasting
      AS-REP Roasting
      BloodHound Enumeration
    Post-Exploitation
      Privilege Escalation
      Lateral Movement
    Infrastructure
      Container Escapes
      Network Pivoting
```

---

## How a Lab Is Built

```mermaid
flowchart LR
    A[Pick a CVE / Technique] --> B[Stand up vulnerable target<br/>Docker / VM]
    B --> C[Recon & confirm<br/>vulnerable version]
    C --> D[Build exploit chain]
    D --> E{Works?}
    E -- No, debug it --> D
    E -- Yes --> F[Capture evidence<br/>screenshots + raw output]
    F --> G[Document methodology<br/>+ remediation]
    G --> H[Push to repo]
```

Every lab documents the **real debugging path** — tooling quirks, networking gotchas, version
mismatches — not just the clean happy-path exploit. That's usually where the actual learning is.

---

## Methodology

Each lab follows the same structure for consistency:

1. **Recon** — identify the vulnerable component and confirm exploitability
2. **Isolation Check** — verify blast radius before touching anything (host mounts, network scope)
3. **Injection / Exploitation** — build and fire the actual exploit chain
4. **Evidence** — screenshots + raw terminal output, unedited
5. **Remediation** — how the vulnerability is actually fixed in production

---

## Environment

<div align="center">

![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![KVM](https://img.shields.io/badge/KVM%2Flibvirt-FF6600?style=flat-square&logo=linux&logoColor=white)
![Kali](https://img.shields.io/badge/Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![Vulhub](https://img.shields.io/badge/Vulhub-Targets-black?style=flat-square)

</div>

All labs run **fully self-hosted** — no cloud infrastructure, no shared targets. Vulnerable
services are containerized (Vulhub) or run as isolated KVM/libvirt VMs, with network isolation
verified before any exploitation begins.

---

## Disclaimer

> Every lab in this repository is performed against **deliberately vulnerable, self-hosted
> targets** on an isolated local network, for educational purposes only. No production systems,
> third-party infrastructure, or external targets are involved in any exercise documented here.

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:FF0066,100:8E2DE2&height=100&section=footer" width="100%"/>

**⭐ More labs added as they're completed — check the index above for progress.**

</div>
