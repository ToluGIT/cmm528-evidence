# February 20, 2025 — Security Incident Analysis

## Incident Summary

| Field | Detail |
|-------|--------|
| **Date** | February 20, 2025 |
| **Attack Window** | 03:36:39 — 03:48:49 UTC (12 minutes) |
| **Total IDS Alerts** | 98 |
| **Victim IP** | 10.1.11.101 |
| **Victim OS** | Windows 7 SP1 64-bit (IE 11) |
| **DNS/Gateway** | 10.1.11.1 |
| **Attack Type** | Drive-by Download → Exploit Kit → Smoke Loader → Cryptominer |
| **Severity** | HIGH — Active malware, C2 communication, resource hijacking |

---

## Attack Chain Overview

```
User browsing → Redirect to Exploit Kit → Flash exploitation →
Smoke Loader installed → C2 beaconing → bprocess.exe downloaded →
XMRig Monero miner installed → Mining to pool.minexmr.com
```

---

## Phase 1: Exploit Kit Delivery (03:36:39 — 03:36:45, 6 seconds)

**Attack Server**: 188.227.16.131:80 (nginx/1.2.1)

The victim's browser was redirected to a combined **SunDown/RIG Exploit Kit** hosted at 188.227.16.131. The EK landing page contained heavily obfuscated JavaScript with base64-encoded URL parameters.

### Alerts (40 total)

| Signature | Count | Priority |
|-----------|-------|----------|
| ET CURRENT_EVENTS SunDown EK RIP Landing M1 B642 | 14 | 1 |
| ET CURRENT_EVENTS SunDown EK RIP Landing M4 B642 | 11 | 1 |
| ET INFO Suspicious Possible CollectGarbage in base64 1 | 11 | 3 |
| ET CURRENT_EVENTS RIG EK URI Struct Mar 13 2017 M2 | 3 | 1 |
| ET CURRENT_EVENTS RIG EK URI Struct Jun 13 2017 | 1 | 1 |
| ET POLICY Outdated Flash Version M1 | 1 | 1 |

### Key Evidence

- **SunDown EK**: Landing page with obfuscated parameters (base64: `dW5rbm93bg==` = "unknown", `bG9jYXRlZA==` = "located", `YXR0YWNrcw==` = "attacks", etc.)
- **RIG EK**: URI structures matching both March 2017 and June 2017 RIG variants
- **CollectGarbage**: JavaScript exploitation technique using garbage collection to trigger use-after-free vulnerabilities
- **Outdated Flash**: Confirms victim's Adobe Flash Player was outdated and exploitable
- **Vulnerability Exploited: CVE-2015-8651** — Adobe Flash Player integer overflow (ByteArray corruption/type confusion), confirmed by VirusTotal tags on the Flash exploit file
- **Flash exploit file (VirusTotal: 34/62 detections)**:
  - SHA256: `535d02392f19fccc890133bc044fe17ef31500e17045c72458482e07b9aa37db`
  - Threat label: **trojan.flash/exkit**, Family: flash, exkit, girdrop
  - AhnLab-V3: **SWF/RigEK.Gen** (independently confirms RIG Exploit Kit)
  - Antiy-AVL: **Trojan[Exploit]/SWF.CVE-2015-8651** (confirms CVE)
  - Code analysis: RC4 obfuscation, ByteArray corruption, shellcode injection via VirtualProtect
- **User-Agents observed**:
  - IE 11: `Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko`
  - Flash/WinHttp: `Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)` — Flash downloading additional exploit content
  - IE 10 compat mode: `Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/7.0; ...)`

---

## Phase 2: Smoke Loader C2 + Malware Download (03:37:22 — 03:38:05, 43 seconds)

**C2 Server**: 104.236.160.225 — domain: **saumottam.ru** (Russian .ru domain)
**Download Server**: 104.236.16.69 (bare IP, Apache/2.4.18 Ubuntu)

### Alerts (22 total)

| Signature | Count | Priority |
|-----------|-------|----------|
| ET TROJAN Sharik/Smoke CnC Beacon 8 | 10 | 1 |
| ET TROJAN Sharik/Smoke CnC Beacon 9 | 10 | 1 |
| ET TROJAN Possible Sharik/Smoke Loader Microsoft Connectivity check M3 | 1 | 1 |
| ET INFO Executable Download from dotted-quad Host | 1 | 1 |

### Malware Download: bprocess.exe

| Signature | Count | Priority |
|-----------|-------|----------|
| ET INFO SUSPICIOUS Dotted Quad Host MZ Response | 13 | 2 |
| ET POLICY PE EXE or DLL Windows file download HTTP | 13 | 1 |

### Key Evidence

- **C2 Beacons**: HTTP POST requests to `saumottam.ru` with 63-byte encrypted payloads
- **20 beacon alerts** across 5 connections (source ports: 49246, 49252, 49253, 49258, 49260)
- **Connectivity check**: GET to `www.microsoft.com` via 23.34.132.22 — standard Smoke Loader behaviour to verify internet access
- **Legitimate browsing**: bing.com, go.microsoft.com, msdn.microsoft.com, www.visualstudio.com — either connectivity verification or browser camouflage
- **C2 Response**: Fake HTTP 404 page from Apache/2.2.15 (CentOS), charset=windows-1251 (Cyrillic) — C2 server masquerading as a normal web server (evasion technique)
- **bprocess.exe download** at 03:38:01:
  - Source: `104.236.16.69` (bare IP — no domain name, evasion technique)
  - HTTP GET `/bprocess.exe`
  - Server: Apache/2.4.18 (Ubuntu)
  - Content-Type: `application/x-msdos-program`
  - Content-Length: **828,929 bytes** (~809 KB)
  - MZ header confirmed — Windows PE executable
  - **MD5:** `7b3491e0828d443f11080efaeb0fbec2`
  - **SHA1:** `e2efe09cb8bd67840f0a8bf02b57ade97e406a88`
  - **Zeek file ID:** `HTTP-Ft6jLvO4qLM1jaRtk.exe`
- **PE Analysis (from Zeek):**
  - Compile date: 2014-12-05 (over 10 years old)
  - Architecture: x86 (32-bit), Windows GUI subsystem
  - **NSIS installer** (identified by `.ndata` PE section) — self-extracting dropper
  - No ASLR, no DEP, no code signing — deliberately weak security features
  - Minimum OS target: Windows XP
- **VirusTotal (60/70 detections, Community Score: -84):**
  - SHA256: `f9c67313230bfc45ba8ffe5e6abeb8b7dc2eddc99c9cebc111fcd7c50d11dc80`
  - Threat label: **trojan.zbot/duyz**, Categories: trojan + miner
  - Family labels: zbot, duyz, thaaeh
  - Tags: nsis, spreader, persistence, long-sleeps, detect-debug-environment
  - ALYac: **Misc.Riskware.MoneroMiner** (confirms Monero mining)
  - AhnLab-V3: Trojan/Win32.Zbot.C2350113
  - BitDefender: Trojan.Generic.37922495
- **Flash exploit file** (from exploit kit):
  - MIME type: `application/x-shockwave-flash`
  - **MD5:** `34904aefefc53da7ae9cb581b3737639`
  - **SHA1:** `111df329c3be57e9fa7318f4f0345bbaec83ce46`
  - Delivered from `188.227.16.131` (exploit kit server)

---

## Phase 3: Cryptocurrency Mining (03:43:35 — 03:48:49, ~5 minutes)

**Mining Pool**: pool.minexmr.com (port 5555)
**Miner Software**: XMRig/2.4.2
**Cryptocurrency**: Monero (XMR)

### Alerts (8 total)

| Signature | Count | Priority |
|-----------|-------|----------|
| ET POLICY Cryptocurrency Miner Checkin | 6 | 1 |
| ET POLICY DNS request for Monero mining pool | 2 | 1 |

### Mining Pool Servers (all on port 5555)

| IP Address | Connections | Timestamp |
|------------|-------------|-----------|
| 188.165.214.95 | 1 | 03:43:35 |
| 91.121.2.76 | 1 | 03:43:57 |
| 94.23.212.204 | 1 | 03:44:04 |
| 94.130.206.79 | 2 | 03:44:23, 03:45:57 |
| 94.130.164.60 | 1 | 03:48:49 |

### Key Evidence (from decoded packet payloads)

**DNS Query**: `pool.minexmr.com` resolved via 10.1.11.1:53

**XMRig JSON-RPC Login** (extracted from packet data):
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "login",
  "params": {
    "login": "49CYQrmrqP2LyHQdMY26JGiq3M9cxFkiSU9PyfSEiXAPVR3kSWwj8Jdh2pJRPHV1ZTCQGwivo1a6494wNk7iasLpD2VrGrx",
    "pass": "x",
    "agent": "XMRig/2.4.2 (Windows NT 6.1; Win64; x64) libuv/1.14.1 gcc/7.2.0"
  }
}
```

- **Monero Wallet Address**: `49CYQrmrqP2LyHQdMY26JGiq3M9cxFkiSU9PyfSEiXAPVR3kSWwj8Jdh2pJRPHV1ZTCQGwivo1a6494wNk7iasLpD2VrGrx`
- **XMRig version**: 2.4.2, compiled with gcc/7.2.0, using libuv/1.14.1
- **Agent string confirms**: Windows NT 6.1 (Windows 7), Win64, x64 architecture

---

## Complete IP Role Summary

| IP Address | Role | Details |
|------------|------|---------|
| **10.1.11.101** | Victim | Windows 7 SP1 64-bit, IE 11 |
| **10.1.11.1** | DNS/Gateway | Internal DNS server |
| **188.227.16.131** | Exploit Kit Server | SunDown/RIG EK, nginx/1.2.1 |
| **104.236.160.225** | Smoke Loader C2 | saumottam.ru (Russian domain) |
| **104.236.16.69** | Malware Hosting | Served bprocess.exe, Apache/2.4.18 |
| **23.34.132.22** | Legitimate | www.microsoft.com (connectivity check) |
| **188.165.214.95** | Mining Pool | pool.minexmr.com:5555 |
| **91.121.2.76** | Mining Pool | pool.minexmr.com:5555 |
| **94.23.212.204** | Mining Pool | pool.minexmr.com:5555 |
| **94.130.206.79** | Mining Pool | pool.minexmr.com:5555 |
| **94.130.164.60** | Mining Pool | pool.minexmr.com:5555 |
| 204.79.197.200 | Legitimate | www.bing.com |
| 23.64.167.178 | Legitimate | go.microsoft.com |
| 157.56.148.19 | Legitimate | msdn.microsoft.com |
| 23.76.194.22 | Legitimate | www.microsoft.com |
| 104.92.3.176 | Legitimate | www.visualstudio.com |

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|--------|-----------|-----|---------|
| Initial Access | Drive-by Compromise | T1189 | SunDown/RIG EK redirect to 188.227.16.131 |
| Execution | Exploitation for Client Execution | T1203 | Outdated Flash Player exploited via CollectGarbage |
| Persistence | Boot or Logon Autostart Execution | T1547 | Smoke Loader typically adds registry persistence |
| Command & Control | Application Layer Protocol: Web | T1071.001 | HTTP POST beacons to saumottam.ru |
| Command & Control | Non-Standard Port | T1571 | Mining pool connections on port 5555 |
| Defense Evasion | Obfuscated Files or Information | T1027 | Base64-encoded EK parameters, encrypted C2 payloads |
| Defense Evasion | Masquerading | T1036 | Smoke Loader mimics legitimate browsing (bing, microsoft) |
| Ingress Tool Transfer | Ingress Tool Transfer | T1105 | bprocess.exe downloaded from bare IP 104.236.16.69 |
| Discovery | System Information Discovery | T1082 | XMRig agent string reports OS details |
| Impact | Resource Hijacking | T1496 | XMRig Monero cryptocurrency mining |

---

## Alert Distribution Summary

| Category | Alerts | Percentage |
|----------|--------|------------|
| Exploit Kit (SunDown/RIG) | 30 | 30.6% |
| Exploit Payload (CollectGarbage + Flash) | 12 | 12.2% |
| Smoke Loader C2 Beacons | 21 | 21.4% |
| Malware Download (bprocess.exe) | 27 | 27.6% |
| Cryptocurrency Mining | 8 | 8.2% |
| **Total** | **98** | **100%** |

---

## Network Connection Analysis (Zeek/Bro)

**Total connections:** 43 during attack window

| Connection State | Count | Meaning |
|-----------------|-------|---------|
| SF (Normal) | 21 | Standard request-response completed |
| RSTO/RSTR (RST aborted) | 15 | Connection reset — exploit kit probing |
| S2/S3 (Responder aborted) | 3 | Connection rejected by destination |
| SH (Half-open) | 2 | SYN sent, no response |
| S1 (Established, not terminated) | 1 | **Active mining still running at capture end** |
| RSTOS0 (SYN-RST) | 1 | Immediate rejection |

**Longest connection:** 94.130.164.60:5555 (mining pool) — 248.8 seconds (~4 min), destination geo: Germany, S1 state (not terminated)

---

## File Hash Summary (Zeek extraction + VirusTotal confirmation)

| File | MD5 | SHA256 | VT Detection | Classification |
|------|-----|--------|-------------|----------------|
| bprocess.exe | `7b3491e0828d443f11080efaeb0fbec2` | `f9c67313230bfc45ba8ffe5e6abeb8b7dc2eddc99c9cebc111fcd7c50d11dc80` | **60/70** | trojan.zbot/duyz (NSIS dropper + Monero miner) |
| Flash exploit | `34904aefefc53da7ae9cb581b3737639` | `535d02392f19fccc890133bc044fe17ef31500e17045c72458482e07b9aa37db` | **34/62** | trojan.flash/exkit (CVE-2015-8651) |

---

## Data Sources

| File | Contents |
|------|----------|
| `feb2025_all_alerts_readable.tsv` | 98 alerts with human-readable IPs |
| `feb2025_alert_counts.tsv` | Alert signature counts |
| `feb2025_http.tsv` | tshark HTTP request data with URIs and User-Agents |
| `feb2025_payloadss.tsv` | Decoded packet payloads (hex) |
| `snort.log.1740009600` | Original pcap file (Feb 20, 2025) |

