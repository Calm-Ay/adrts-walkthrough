# ADRTS Lab Walkthrough — telecore.ad

**Author:** Rasaq Ayomide (Calm Ay)
**Lab:** ADRTS — Active Directory Red Team Specialist
**Result:** ✅ Full domain compromise — 24/24 flags
**Live page:** [calm-ay.github.io/adrts-walkthrough](https://calm-ay.github.io/adrts-walkthrough/)
**Profile:** [calm-ay.github.io](https://calm-ay.github.io) · [LinkedIn](https://www.linkedin.com/in/rasaq-ayomide-sec) · [GitHub](https://github.com/Calm-Ay)

---

A full Active Directory red team lab against `telecore.ad`, walked end to end along two separate adversary paths. Starting position: VPN access only, zero credentials.

## Path 01 — Unauthenticated Adversary (VPN → Domain Admin → ESXi)

1. **Recon** — `nmap` service sweep of `10.5.2.0/24` maps six hosts (DC, PKI, Exchange, SQL, ESXi, IIS web).
2. **AS-REP Roasting** — `tel_support01` has Kerberos pre-auth disabled; `GetNPUsers` pulls the hash, cracked offline.
3. **MSSQL → xp_cmdshell** — recovered password logs into SQL-SRV; `xp_cmdshell` gives OS command execution.
4. **GodPotato + SeImpersonate** — token abuse escalates the SQL service account to `NT AUTHORITY\SYSTEM`.
5. **LSASS dump → pypykatz** — SYSTEM-level LSASS dump exfiltrated over SMB, credentials extracted offline.
6. **ADCS ESC1** — vulnerable certificate template; request with spoofed DA SID mints a certificate as Domain Admin.
7. **Certipy → hash → john** — authenticate with the certificate, recover the DA NT hash, crack to cleartext.
8. **ESXi via pyVmomi → VaultVM RCE** — DA access to ESXi exposes VM-annotation creds; `mkfifo` shell into VaultVM captures the flag.

## Path 02 — Authenticated Adversary (IIS foothold → DC filesystem → Exchange)

1. **IIS 10 / ASP.NET recon** — map the Web-Srv application.
2. **LFI → web.config extraction** — leaks machine keys and machine name.
3. **ysoserial.net → ViewState injection** — forged `__VIEWSTATE` deserializes into an IIS worker shell.
4. **Registry → DPAPI blob** — decrypt a DPAPI-protected blob to unlock stored secrets.
5. **DC filesystem via SQL pivot** — a scheduled-task script widens access into the domain core.
6. **Exchange EWS → mailbox dump** — the authenticated path's objective and the last of the 24 flags.

## Full report

📄 [`ADRTS_telecore_Walkthrough_CalmAy.pdf`](./ADRTS_telecore_Walkthrough_CalmAy.pdf) — every scan, payload, credential, and the reasoning that connects each technique to the next, with the full 24-flag summary.

---

*For educational and authorized penetration-testing purposes only.*
