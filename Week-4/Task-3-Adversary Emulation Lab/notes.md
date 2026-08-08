Adversary Emulation Lab — APT29 Simulation

Adversary Emulation & Blue Team Detection

Objective

Emulate APT29 (Cozy Bear) tactics, techniques, and procedures (TTPs) using MITRE Caldera and Evilginx2 in an isolated lab environment, then analyze Wazuh SIEM logs to evaluate blue team detection coverage.

Lab Environment
Component	Details
Attacker Host	Kali Linux 2026.1 (VirtualBox)
Target Host	Windows 10 (VirtualBox), isolated host-only network
C2 / Emulation Framework	MITRE Caldera v5.3.0
Phishing Framework	Evilginx2 Community Edition v3.3.0
SIEM / Detection	Wazuh Manager, Indexer, Dashboard v4.7.5

All activity was conducted strictly within an isolated VirtualBox lab network — no external or production systems were targeted.

Part 1: Caldera — APT29 Emulation

Since a pre-built APT29 adversary profile was not available in the local Caldera stockpile, a custom adversary profile (APT29-Custom-Emulation) was built by mapping individual abilities to known APT29 TTPs (per MITRE ATT&CK).

Setup Summary
Deployed Sandcat agent to Windows 10 target via PowerShell one-liner (required temporarily disabling Windows Defender real-time protection in the isolated lab VM)
Agent connected successfully with elevated privileges (group: red)
Built a 4-step operation chain: Discovery → Discovery → Execution → Persistence
Ran operation with autonomous execution enabled
Emulation Log
Phase	TTP	Tool Used	Notes
Discovery	T1082	Caldera (Find OS Version)	Enumerated target Windows OS version — Success
Discovery	T1033	Caldera (Current User)	Identified active logged-in user context — Success
Execution	T1059.001	Caldera (Emulate Administrator Tasks)	Simulated admin task execution via PowerShell — Success
Persistence	T1547.001	Caldera (Winlogon HKCU Shell Key Agent Execution)	Created registry-based logon persistence (technique adapted from a Dark Pink-derived ability in the local stockpile, functionally equivalent to APT29's Registry Run Key persistence) — Success
Phishing	T1566.001	Evilginx2	Credential harvesting infrastructure configured (see Part 2)

All 4 Caldera abilities executed successfully against the Windows 10 target agent.

Part 2: Evilginx2 — Phishing Simulation

Evilginx2 was installed and configured to simulate a T1566.001 (Spearphishing Link) credential-harvesting scenario using a reverse-proxy phishing framework.

Setup Completed
Built Evilginx2 from source (Go)
Configured local lab domain (lab.local) with target VM hosts file entries for DNS resolution
Enabled the example phishlet with hostname mapping
Generated a phishing lure URL
Known Limitation

TLS certificate provisioning encountered a persistent issue in this build (v3.3.0): neither Let's Encrypt autocert nor manually-generated self-signed certificates were picked up correctly by Evilginx2's internal certificate store (cert_db) for the phishing subdomain, resulting in ERR_SSL_PROTOCOL_ERROR on the target browser. Root-cause troubleshooting traced the issue to file-naming/format conflicts within ~/.evilginx/crt/sites/. Due to time constraints, full end-to-end credential capture was not demonstrated, though the phishing infrastructure (domain, phishlet, lure generation) was fully configured and functional up to the TLS layer.

This reflects a realistic infrastructure troubleshooting challenge encountered when standing up phishing simulation tooling.

Part 3: Blue Team Detection — Wazuh
Setup Summary
Wazuh Manager, Indexer, and Filebeat (v4.7.5) were already present on the Kali host; Wazuh Dashboard was installed and configured
Resolved dashboard startup failures caused by missing TLS certificates (dashboard-key.pem, root-ca.pem) by generating a self-signed certificate and linking the Indexer's existing root CA
Confirmed Manager–Dashboard connectivity (API status: Online)
Began Windows agent deployment via the Dashboard's "Deploy new agent" wizard; encountered an agent/manager version compatibility error during enrollment (Agent version must be lower or equal to manager version), requiring a version-pinned reinstall to resolve
Detection Summary (50 words)

Wazuh's Wazuh Manager, Indexer, and Dashboard were deployed and connected successfully, confirming SIEM readiness for log ingestion. Windows agent enrollment was in progress but blocked by an agent/manager version mismatch. Once resolved, expected detections include PowerShell execution (Sysmon Event ID 1) and registry persistence via File Integrity Monitoring alerts.

Key Learnings
Custom adversary profile construction — mapped real-world APT29 TTPs to available Caldera abilities when a pre-built profile wasn't available, reinforcing understanding of the MITRE ATT&CK framework beyond point-and-click tooling.
Infrastructure troubleshooting is a core red-team skill — diagnosed and resolved AV blocking, network binding, and multi-layered TLS certificate chain issues across three separate tools (Caldera, Evilginx2, Wazuh).
SIEM deployment dependencies — Wazuh's Indexer/Dashboard/Manager stack has strict version and certificate-chain requirements that must match exactly across components and agents.
Tools Reference
MITRE Caldera
Evilginx2
Wazuh
MITRE ATT&CK — APT29
