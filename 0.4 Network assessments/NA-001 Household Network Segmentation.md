
**Date:** 2026-10-18 
**Status:** Assessment complete, implementation blocked 
**Related:** Fortinet NSE1, Lab 01 (Wazuh brute-force)

## Objective

Segment my household network into zones (trusted, IoT, guest) to apply network security concepts from the Fortinet certification.

## Environment

- **ISP:** residential cable provider
- **Gateway:** ISP-provided modem + router combo (ISP-owned)
- **Default gateway:** `<GATEWAY_IP>`
- **My MacBook IP:** `<SCANNER_IP>` (assigned by DHCP)
- **Devices:** laptops, phones, TVs, tablets
- **Current state:** flat network (all devices in one zone)

## Steps & Findings

1. **Asset discovery:** identified gateway model and IP.
    - Initially confused my laptop's IP with the gateway's. Verified with:
        - `route -n get default | grep gateway` → gateway IP
        - `ipconfig getifaddr en0` → my laptop's IP on Wi-Fi
2. **Accessed `<GATEWAY_IP>`:** browser showed a certificate warning.
    - Cause: router uses a **self-signed certificate** (not issued by a trusted CA).
    - Safe to proceed on my own local device; never on public websites (MitM risk).
3. **Redirected to the ISP's web portal:** the gateway has no local admin page. It is **cloud-managed** by the ISP.
    - Verified the domain before signing in (phishing check).
4. **Port configuration page:** "This section is unavailable" for the account used.
    - Likely cause: account lacks admin rights (**RBAC**) or the gateway is app-managed only.

## Conclusion

The network edge is controlled by a third party (the ISP). The gateway does not support VLANs or inter-zone firewall rules. Real segmentation would require my own router/firewall behind the gateway.

**Real-world parallel:** organizations often don't control ISP equipment, so they place their own firewall (e.g., FortiGate) behind it and enforce segmentation there.

## How segmentation would work (design)

|VLAN|Zone|Subnet|Devices|
|---|---|---|---|
|10|Trusted|192.168.10.0/24|Laptops, phones|
|20|IoT|192.168.20.0/24|TVs, tablets|
|30|Guest|192.168.30.0/24|Visitors|

**Firewall policies (default deny):**

- Trusted → IoT: allow
- IoT → Trusted: deny
- Guest → internal: deny
- All zones → internet: allow

**Known gotcha:** casting (AirPlay/Chromecast) uses **mDNS**, which doesn't cross VLANs without an mDNS reflector.

## Options Evaluated

|Option|Cost|Decision|
|---|---|---|
|Own router with VLAN support|$|Not now (no budget)|
|Used FortiGate|$$|Not now (no budget)|
|pfSense/OPNsense mini PC|$|Not now (no budget)|
|FortiGate-VM (free permanent trial, ARM64 on UTM)|Free|Declined for now|
|ISP guest network|Free|To ask ISP|
|macOS host-based firewall|Free|Enabled|

## Next Steps

- [x] Enable macOS firewall (System Settings → Network → Firewall)
- [ ] Contact ISP: ask about guest network and bridge mode on the gateway

## Key Terms

- **Network segmentation:** splitting a network into zones to limit lateral movement
- **Flat network:** all devices share one zone
- **Default gateway:** device that routes traffic out of the local network
- **DHCP:** automatically assigns IP addresses to devices
- **VLAN / VLAN ID:** virtual network on shared hardware, identified by a number
- **802.1Q:** standard for VLAN tagging
- **Trunk port / access port:** carries multiple VLANs / carries one VLAN
- **Subnet / CIDR (/24):** IP address range / notation for network size
- **SSID:** Wi-Fi network name
- **Self-signed certificate / CA:** certificate the device vouches for itself / trusted issuer
- **MitM:** attacker intercepting traffic between two parties
- **Cloud-managed device:** configured through a vendor portal, not locally
- **RBAC:** permissions based on user role
- **Bridge mode:** gateway stops routing and passes traffic to your own router
- **Double NAT:** two routers both translating addresses
- **Default deny:** block everything, allow only what's needed
- **Host-based firewall:** firewall running on the device itself
- **Defense in depth:** layered security controls