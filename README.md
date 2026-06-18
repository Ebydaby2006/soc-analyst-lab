# 🔵 soc-analyst-lab

A personal blue team lab documenting hands-on SOC analyst exercises, PCAP analysis, network forensics investigations, and incident report writeups.

Built by a cybersecurity student at the **Federal University of Technology Owerri (FUTO)** actively working towards a SOC Analyst role.

---

## 📁 Repository Structure

```
soc-analyst-lab/
│
├── 01-network-forensics/
│   └── 2026-02-28-netsupport-rat/
│       ├── README.md        ← Incident report (this exercise)
│       └── screenshots/     ← Evidence screenshots
│
├── 02-letsdefend-labs/      ← Coming soon
├── 03-cyberdefenders/       ← Coming soon
└── 04-tryhackme/            ← Coming soon

```

---

## 🧪 Exercises Completed

| # | Date | Title | Source | Difficulty | Status |
|---|------|-------|--------|------------|--------|
| 01 | 2026-02-28 | NetSupport Manager RAT — Traffic Analysis | malware-traffic-analysis.net | Beginner | ✅ Complete |

---

## 📄 Exercise 01 — NetSupport Manager RAT

### 🗂 Source
[malware-traffic-analysis.net — 2026-02-28 Traffic Analysis Exercise: Easy As 123](https://www.malware-traffic-analysis.net/2026/02/28/index.html)

---

### 🏢 Environment
| Field | Value |
|---|---|
| LAN Segment Range | `10.2.28.0/24` |
| Domain | `easyas123.tech` |
| AD Environment | `EASYAS123` |
| Domain Controller | `10.2.28.2` — EASYAS123-DC |
| Gateway | `10.2.28.1` |
| Broadcast | `10.2.28.255` |

---

### 🚨 Scenario
As a SOC analyst, the SIEM flagged several signature hits for **NetSupport Manager RAT** communicating with external IP `45.131.214.85` over **TCP port 443**. Activity began on **2026-02-28 at 19:55 UTC**.

A PCAP was retrieved from the internal host triggering the alerts. The goal: identify the infected machine and produce an incident report.

---

### 🔍 Investigation Methodology

#### Step 1 — Identify the infected host
![C2 traffic filtered by attacker IP](screenshots/01-infected-host-handshake-beacon.png)
**Filter used:**
```
ip.addr == 45.131.214.85
```
**Finding:** Internal IP `10.2.28.88` was the only host communicating with the attacker's C2 server. The TCP handshake (SYN → SYN-ACK → ACK) was followed immediately by repeated HTTP POST requests to `http://45.131.214.85/fakeurl.htm` — a classic RAT beacon pattern, firing approximately every 60 seconds.

---

#### Step 2 — Retrieve MAC address
**Method:** Clicked the first SYN packet and expanded the Ethernet II layer in the packet detail pane.

**Finding:**
```
Source: Intel_b2:4d:ad (00:19:d1:b2:4d:ad)
```

---

#### Step 3 — Retrieve hostname
![NBNS registration showing DESKTOP-TEYQ2NR](screenshots/02-nbns-hostname.png)
**Filter used:**
```
nbns
```
**Finding:** The infected machine was broadcasting its NetBIOS name to the subnet (`10.2.28.255`):
```
Registration NB DESKTOP-TEYQ2NR
```

---

#### Step 4 — Retrieve Windows user account name
![kerberos CNameString Filter showing brolf](screenshots/03-kerberos-username.png)
**Filter used:**
```
kerberos.CNameString
```
**Finding:** Expanded `cname → cname-string` in the Kerberos AS-REQ packet detail pane:
```
CNameString: brolf
```

---

#### Step 5 — Retrieve full name of user
![SAMR QueryUserInfo response showing Becka Rolf](screenshots/04-fullname.png)
**Method:** Edit → Find Packet → **Packet details**, String, **Case sensitive**, searched for `Rolf`

**Finding:** Located in a SAMR `QueryUserInfo` response (packet 339) from the domain controller:
```
Full Name: Becka Rolf
```

---

### 📋 Incident Report

| Field | Value |
|---|---|
| **Infected IP** | `10.2.28.88` |
| **MAC Address** | `00:19:d1:b2:4d:ad` |
| **Hostname** | `DESKTOP-TEYQ2NR` |
| **User Account** | `brolf` |
| **Full Name** | `Becka Rolf` |
| **Malware** | NetSupport Manager RAT |
| **C2 Server** | `45.131.214.85:443` |
| **C2 Endpoint** | `http://45.131.214.85/fakeurl.htm` |
| **Protocol** | HTTP over TCP port 443 |
| **Beacon Interval** | ~60 seconds |
| **Activity Start** | 2026-02-28 at 19:55 UTC |

---

### 🧠 Key Lessons Learned

- **Beacon detection:** Regular, clock-like POST requests to the same external IP at ~60 second intervals is a primary IOC for RAT activity. No human behaves this mechanically — SIEMs flag this pattern automatically.
- **`fakeurl.htm`:** The attacker named their C2 endpoint literally `fakeurl.htm`. Always check the URI in HTTP POST traffic — legitimate software rarely POSTs to suspicious-looking endpoints.
- **Protocol mismatch:** The RAT communicated over TCP port 443 but used plain HTTP — not HTTPS. Port 443 is expected to carry TLS-encrypted traffic. Unencrypted traffic on 443 is itself a red flag.
- **Find Packet modes:** Wireshark's Find Packet has three modes — Packet bytes, Packet details, and Display filter. The full name was only discoverable via **Packet details** search, not packet bytes.
- **SAMR protocol:** Full user account details (including display names) are transmitted via the Security Account Manager Remote (SAMR) protocol when Windows queries Active Directory for user info.

---

### 🛠 Tools Used
- Wireshark (with TCP SYN profile)
- malware-traffic-analysis.net PCAP

---

## 🎯 Learning Tracks

This lab supports my active learning across:
- **TCM Security** — SOC 101
- **LetsDefend** — SOC Analyst Learning Path
- **malware-traffic-analysis.net** — Traffic Analysis Exercises

---

## 📬 Connect
- **LinkedIn:** [Sharon's LinkedIn](#)
- **GitHub:** [@Ebydaby2006](https://github.com/Ebydaby2006)

---

> *"You can't defend what you can't see. Learn the packets."*
