
**Date:** 2026-10-18
**Status:** In progress
**Related:** NA-001 Household segmentation assessment

## Objective
Discover and inventory every device on my home network, identify exposed ports and services, and assess risk.

## Scope & Authorization
- **Target:** my own home network only (`<YOUR_SUBNET>/24`)
- Scanning networks you don't own or have permission to test is illegal.

## Environment
- **Scanner:** MacBook Air M3 (zsh / Oh My Zsh)
- **Tool:** Nmap 7.992 (installed via Homebrew)
- **Network:** flat home network behind ISP gateway

## Step 1 – Install Nmap

brew install nmap  
nmap --version

**Issue:** `/opt/homebrew is not writable`. Homebrew's folder was owned by another user (likely root, from a past `sudo brew` command).

**Fix:**
```

sudo chown -R <YOUR_USERNAME> /opt/homebrew  
chmod -R u+w /opt/homebrew

```
**Lesson:** never run `brew` with `sudo`.

## Step 2 – Host Discovery
sudo nmap -sn <YOUR_SUBNET>/24


- `-sn` = ping scan (finds live hosts, no port scan)
- `sudo` = lets Nmap use ARP, revealing MAC addresses and vendors

**Result:** 6 hosts up

| IP | Vendor (from OUI) | Likely device |
|---|---|---|
| `<GATEWAY_IP>` | Ubee Interactive | ISP gateway (DOCSIS cable modem/router) |
| `<HOST_IP>` | Nintendo | Nintendo Switch |
| `<HOST_IP>` | Shenzhen SDMC | Streaming / TV box |
| `<HOST_IP>` | Unknown | Phone or tablet (randomized MAC) |
| `<HOST_IP>` | TPV Display | Smart TV |
| `<SCANNER_IP>` | — | My MacBook (scanner) |

MAC addresses: `<REDACTED>`

### Observations
- **OUI:** the first half of a MAC address identifies the manufacturer.
- **Randomized MAC:** a MAC starting with `x2`, `x6`, `xA` or `xE` (e.g., `BE:...`) is locally administered. Phones use these ("Private Wi-Fi Address") to prevent tracking, which makes asset inventory harder.
- **Point-in-time snapshot:** asleep or idle devices may not respond, so some known devices didn't appear.
- **IoT exposure:** 3 of 6 hosts are IoT / entertainment devices sharing a flat network with my laptop.

## Step 3 – Port Scan

### Concept
- **IP address** = building address; **port** = apartment number (0–65,535); **service** = who answers the door
- Ports are virtual (software), not physical plugs
- Common ports: 22 = SSH, 53 = DNS, 80 = HTTP, 443 = HTTPS
- Every open port is part of the device's **attack surface**

### Command
```
sudo nmap -sV -T4 -F <YOUR_SUBNET>/24
```
- **SYN scan (`-sS`, default with sudo):** "half-open" scan; starts a connection but never completes it. Faster and quieter.
- **`-sV`:** service version detection; identifies the software and version on each open port
- **`-T4`:** timing template (`-T0` slowest/stealthiest → `-T5` fastest/noisiest; default `-T3`). Faster scans are easier for defenders to detect.
- **`-F`:** fast mode; top 100 ports instead of the default top 1,000

### Scope note
First attempt with the default 1,000 ports (`sudo nmap -sV <YOUR_SUBNET>/24`) was stopped after ~15 min. Re-ran with `-T4 -F`. **This scan covered only the top 100 ports.**

### Troubleshooting: progress went backwards
- Press **spacebar** during a scan to see progress.
- Progress dropped from 61% to 57% within the same phase (SYN Stealth Scan).
- Cause: **adaptive timing**. When a device responds slowly or drops probes (sleeping phone, rate limiting), Nmap slows down and re-estimates.
- Nmap runs in **phases** (host discovery → port scan → service detection); the percentage restarts each phase.
- Nmap is already parallel; slowness comes from waiting on slow devices and timeouts.

### Port states
- **open:** a service is listening (what matters)
- **closed:** reachable, nothing listening
- **filtered:** a firewall is blocking the probe

### Results
| Host | Device | Open ports | Notes |
|---|---|---|---|
| `<GATEWAY_IP>` | ISP gateway | 53, 80, 443, 8000, 49152–49154 | SSH (22), Telnet (23), 8080 filtered |
| `<HOST_IP>` | Nintendo Switch | none | All 100 filtered |
| `<HOST_IP>` | ISP Wi-Fi extender | 21 (FTP), 22 (SSH, Dropbear 2022.83), 23 (Telnet), 49152 | Broadcom BCM96756, Linux |
| `<HOST_IP>` | Phone (randomized MAC) | none | All 100 filtered |
| `<HOST_IP>` | Huawei MatePad Pro | none | All 100 closed; broadcasts hostname |
| `<HOST_IP>` | Smart TV | 8009 (Google Cast) | tcpwrapped |
| `<SCANNER_IP>` | My MacBook | 5000 (AirPlay Receiver) | Returned 403 Forbidden |

### Analysis

**🔴 High – ISP Wi-Fi extender exposes insecure services**
- **Telnet (23) and FTP (21)** transmit credentials in cleartext; any device on the network could intercept them.
- **SSH** runs Dropbear 2022.83, an older version to check against known CVEs.
- **Risk:** on a flat network, a compromised IoT device could reach these services and use the extender to pivot (**lateral movement**).
- **Owner:** the ISP(third-party asset). 
- **Action:** document and report to vendor; no local remediation possible.
- **Note:** product names in Nmap's version output are fingerprint guesses; open ports are confirmed.

**🟡 Medium – ISP gateway exposes UPnP-like ports**
- Ports 49152–49154 are commonly **UPnP**, which lets devices automatically open ports and announce themselves.
- Admin services (80, 443, 8000) are expected for a gateway; SSH and Telnet are correctly filtered.
- **Owner:** the ISP.  
- **Action:** document only.

**🟡 Low – MacBook AirPlay Receiver (port 5000)**
- Unneeded listening service; rejected unauthenticated requests (403), but still adds attack surface.
- **Owner:** me. 
- **Action:** disable AirPlay Receiver if unused, rescan to verify (**find → fix → verify**).

**🟡 Low – Smart TV Google Cast (port 8009)**
- Expected for casting; any device on the network can discover and control the TV.
- **Action:** accept as normal; would isolate in an IoT VLAN if segmentation were possible (see NA-001).

**🟢 Good – Phone, Nintendo Switch, tablet**
- Phone and Switch silently drop all probes (filtered); tablet has nothing listening (closed).

### Key takeaways
- The riskiest device on the network is one I don't control (ISP extender), reinforcing NA-001's conclusion: the edge and ISP equipment are third-party assets.
- A flat network means every device can reach the extender's insecure services.
- Scan covered only the top 100 ports (`-F`); services on other ports were not assessed.

### Next steps
- [ ] Research CVEs for Dropbear 2022.83
- [x] Disable AirPlay Receiver on MacBook; rescan to verify port 5000 is closed
- [ ] Decide whether to report extender findings to the ISP
- [ ] Optional: full 1,000-port scan of the extender only

### What a port scan can and can't tell you
A port scan shows **attack surface**, not proven weaknesses. But versions from `-sV` can be matched against **CVEs** (Common Vulnerabilities and Exposures). An outdated version with a known CVE is a strong lead.

**The ladder:**
1. **Port scan:** which doors exist
2. **Vulnerability scan:** which doors have known weak locks (versions vs. CVEs)
3. **Penetration test:** actually trying to open them, with permission

SOC analysts work mostly with 1 and 2.

### Key Terms
- **Port / service:** numbered endpoint on a device / software listening on it
- **Attack surface:** all the points an attacker could try to enter
- **SYN scan / half-open scan:** probe that never completes the TCP handshake
- **Service version detection:** identifying software and version behind a port
- **Timing template:** Nmap speed setting (`-T0` to `-T5`)
- **Adaptive timing:** Nmap slowing down when it detects packet loss
- **Rate limiting:** a device restricting how much traffic it accepts
- **CVE:** public ID for a known vulnerability
- **Vulnerability scan / penetration test:** finding known flaws / exploiting them with permission

## Key Terms
- **Network reconnaissance:** gathering information about a network's hosts and services
- **Asset inventory:** list of all devices on a network
- **Host discovery / ping scan:** finding which IPs are live
- **ARP:** protocol that maps IP addresses to MAC addresses on a local network
- **MAC address:** hardware identifier of a network interface
- **OUI:** manufacturer prefix of a MAC address
- **Locally administered address:** MAC set by software, not the factory
- **sudo / chown / chmod:** run as admin / change owner / change permissions






