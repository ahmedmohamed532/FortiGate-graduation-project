# FortiOS 7.6 Administrator — Graduation Project

> Graduation project for the FortiOS 7.6 Administrator course — an HQ-to-Branch transit topology secured and connected using two FortiGate firewalls, built on EVE-NG.

![Status](https://img.shields.io/badge/status-in--progress-yellow)
![Platform](https://img.shields.io/badge/platform-EVE--NG-blue)
![FortiOS](https://img.shields.io/badge/FortiOS-7.6-red)

---

## 👥 Team

- Ahmed Mohamed Ali
- Ebrahim Ashraf
- Mohamed Amr

**Discussion date:** September 20, 2026
**Environment:** EVE-NG
**FortiOS version:** 7.6

---

## 🗺️ Topology Overview

The network follows a classic **HQ-to-Branch transit topology**: two separate local networks (HQ and Branch) connected over a shared WAN link, with a FortiGate firewall acting as the gatekeeper for each site.

### Physical Layout

| Element | Description |
|---|---|
| **WAN** | Represents the internet / service provider network. |
| **SW-WAN** | A Layer 2 switch acting as the transit network, connecting the WAN device to the WAN interfaces of both FortiGates. |
| **HQ Site** | `Firewall-HQ` protects the HQ LAN. It connects to `SW-HQ`, which connects to `PC-1`. |
| **Branch Site** | `Firewall-BR` protects the Branch LAN. It connects to `SW-BR`, which connects to `PC-2`. |

### Logical IP Addressing

| Network | Subnet | Notes |
|---|---|---|
| HQ LAN | 192.168.10.0/24 | Gateway: 192.168.10.1 (HQ port2) |
| Branch LAN | 192.168.20.0/24 | Gateway: 192.168.20.1 (BR port2) |
| WAN Transit Link | 10.0.0.0/24 | HQ WAN (port1): 10.0.0.1 · Branch WAN (port1): 10.0.0.2 |
| Management/Internet | 172.16.47.0/24 | Both firewalls' port3, default route via 172.16.47.2 |

---

## 🛠️ Tools Used

- EVE-NG (network emulator)
- FortiGate VM image — FortiOS 7.6
- Linux end devices (PC-1, PC-2)
- GitHub for project documentation

---

## ⚙️ Implementation

### 1. Account Configuration
<!-- TODO -->
- Changed the default admin password on both firewalls.
- Created an additional admin account with a custom access profile.


### 2. Interface Configuration
- **Firewall-HQ:** port1 = WAN (10.0.0.1/24) · port2 = LAN (192.168.10.1/24) · port3 = management (172.16.47.0/24)
- **Firewall-BR:** port1 = WAN (10.0.0.2/24) · port2 = LAN (192.168.20.1/24) · port3 = management (172.16.47.0/24)


### 3. Routing
- Static route on Firewall-HQ: `192.168.20.0/24 via 10.0.0.2 (port1)`
- Static route on Firewall-BR: `192.168.10.0/24 via 10.0.0.1 (port1)`
- Default route on both firewalls via `172.16.47.2 (port3)`


### 4. Firewall Policy
- HQ: policy `port2 → port1`, NAT disabled, action ACCEPT.
- Branch: policy `port1 → port2`, NAT disabled, action ACCEPT.
- Return traffic is handled automatically since FortiGate is stateful (a matching reverse policy is still good practice).


### 5. Authentication
<!-- TODO -->
- Local users/groups.
- (Optional) Integration with RADIUS/LDAP.


### 6. Security Profiles
<!-- TODO -->
- AntiVirus profile.
- Web Filter profile.


### 7. VPN
<!-- TODO -->
- IPSec site-to-site (route-based) between Firewall-HQ and Firewall-BR.
- (Optional) SSL-VPN for remote access.


### 8. Plus Features (optional)
<!-- TODO: remove this section if you don't implement it -->
- [ ] SD-WAN
- [ ] High Availability (HA)

---

## 🔄 Traffic Flow — How It Works

Example: **PC-1 (192.168.10.10) pings PC-2 (192.168.20.10)**

1. **PC-1 → Firewall-HQ:** PC-1 sends the packet to its default gateway (192.168.10.1), since PC-2 is on a different subnet.
2. **HQ routing:** Firewall-HQ receives the packet on port2, checks its routing table, and finds the static route to 192.168.20.0/24 via 10.0.0.2 on port1.
3. **HQ policy check:** Firewall-HQ checks its firewall policy for port2 → port1 (NAT disabled). If it matches an ACCEPT policy, the packet is forwarded out port1 to Firewall-BR.
4. **Branch routing & policy:** Firewall-BR receives the packet on port1, sees 192.168.20.0/24 is directly connected to port2, checks its policy, and forwards the packet to PC-2.
5. **Return traffic:** PC-2 replies. Since FortiGate is a stateful firewall, the return traffic is automatically permitted through the existing session (a matching reverse-direction policy is still recommended as good practice).

---

## ✅ Testing & Verification

Describe each result in words — exact command output, or a short summary of what happened.

| Test | Command / Method | Result |
|---|---|---|
| Connectivity between sites | `ping 192.168.20.10` from PC-1 | <!-- TODO: e.g. "Reply from 192.168.20.10, 0% packet loss, avg RTT 2ms" --> |
| HQ routing table | `get router info routing-table all` | <!-- TODO: e.g. "Static route to 192.168.20.0/24 via 10.0.0.2 present, distance 10" --> |
| Firewall policy hit count | `diagnose firewall statistics` | <!-- TODO: e.g. "Policy ID 1 (port2→port1) showing active sessions and increasing packet count" --> |
| VPN tunnel status | `diagnose vpn tunnel list` | <!-- TODO: e.g. "Tunnel 'vpn-to-BR' status: up, selectors matching 192.168.10.0/24 <-> 192.168.20.0/24" --> |

---

## 📂 Repository Structure

```
fortigate-graduation-project/
├── README.md
└── configs/
    ├── accounts.conf
    ├── interfaces.conf
    ├── routing.conf
    ├── firewall-policy.conf
    ├── authentication.conf
    ├── security-profiles.conf
    └── vpn.conf
```

---

## 🎯 Conclusion

This project gave us hands-on experience with a realistic HQ-to-Branch FortiGate deployment, going beyond single-device configuration into how two firewalls cooperate to route and secure traffic between sites. Working through the routing and policy logic step by step made it clear how closely interface configuration, static routing, and firewall policies depend on each other — a mistake in one layer (for example, a missing static route or a policy pointing to the wrong interface) breaks connectivity even if everything else is correct. We also came to appreciate FortiGate's stateful inspection firsthand: return traffic from the Branch site was permitted automatically once the initial session was established, which reinforced how session-based firewalls differ from simple stateless packet filtering. Overall, the project strengthened our understanding of how enterprise firewalls are used not just to block traffic, but to actively route and broker connectivity between separate, trusted networks.

---

## 👤 Authors

This project was created as part of the requirements for the **FortiOS 7.6 Administrator** course — Creativa.

- Ahmed Mohamed Ali
- Ebrahim Ashraf
- Mohamed Amr

🔗 [LinkedIn post link](#) <!-- TODO: add the link after publishing -->
