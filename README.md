# 🛒 Enterprise Retail Store Network Infrastructure

A production-ready retail store network simulation built and validated using **Cisco Packet Tracer**. The project mirrors a modern supermarket / department store infrastructure that strictly segments transactional Point-of-Sale (POS) devices, self-checkout kiosks, administrative offices, and guest Wi-Fi across isolated broadcast domains. It incorporates Layer 3 Inter-VLAN routing, dynamic DHCP relay, NAT/PAT translation, Access Control Lists (ACLs), and Layer 2 switchport hardening (Port Security).

---

## 📌 Network Topology & Visual Layout

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d0e5b20c-7038-4d25-8a12-8154a2a21926" />



---

## 🎯 Business Problem & Architectural Goal

Retail environments operate under high-compliance standards (like PCI-DSS for card payment security). Allowing guest Wi-Fi users, administrative workstations, and payment terminals to sit on the same broadcast domain introduces severe vulnerabilities. 

This project solves this by:
1. **Isolating Payment Systems:** Restricting unauthorized access to POS and Self-Checkout systems.
2. **Preventing Rogue Hardware:** Shutting down physical ports instantly if an unknown laptop is connected to an active register.
3. **Restricting Guest Network:** Allowing guest customers to access external internet services while completely dropping any access towards internal company subnets.
4. **Providing Redundant Layer 3 Core:** Performing inter-VLAN routing and centralizing DHCP services directly at the Core Switch.

---

## 📊 IP Addressing & VLAN Scheme

| VLAN ID | Subnet Name | Network Subnet | Default Gateway | Function / Connected Devices |
| :---: | :---: | :---: | :---: | :--- |
| **10** | POS | `192.168.10.0/24` | `192.168.10.1` | Traditional Checkout Cash Registers (POS-1 to POS-4) |
| **20** | Self-Checkout | `192.168.20.0/24` | `192.168.20.1` | Customer Self-Checkout Terminals (SCO-1 to SCO-3) |
| **30** | Back Office | `192.168.30.0/24` | `192.168.30.1` | Store Manager PC & Inventory Tracking PC[cite: 5] |
| **40** | Guest Wi-Fi | `192.168.40.0/24` | `192.168.40.1` | Isolated Wireless Access Point for Staff / Guests[cite: 5] |
| **50** | Printers | `192.168.50.0/24` | `192.168.50.1` | Networked Shared Office Printer (Static IP: 192.168.50.10)[cite: 5] |
| **99** | Management | `192.168.99.0/24` | `192.168.99.1` | SVI Management for 2960 and 3560 Switches (SSH Access)[cite: 5] |
| **999** | Blackhole | — | — | Parking VLAN for all unused/disabled access ports[cite: 5] |
| **WAN** | Point-to-Point | `10.0.0.0/30` | — | Uplink between Layer 3 Core Switch & Edge Router[cite: 5] |
| **ISP** | Public Link | `203.0.113.0/30` | `203.0.113.1` | Simulated Internet Connection to Outside Payment Server[cite: 5] |

---

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3252e089-1edd-41a8-826c-bae886685f87" />


## ⚙️ Key Technical Implementations

### 1. Layer 2 Trunking & Port Optimization
- Access switches (`Front Store` & `Back Store`) connect to the Multilayer Switch via **802.1Q trunks**[cite: 5].
- Trunk links use **Explicit Allowed VLAN lists** (`switchport trunk allowed vlan ...`) rather than carrying arbitrary broadcast traffic[cite: 5].
- Enabled **PortFast** and **BPDU Guard** on all edge access ports to bypass standard Spanning Tree convergence delays while protecting against rogue switch loops[cite: 5].

- <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/87b1a99b-bcc9-4a92-aae1-677d532d7bf1" />


### 2. Core Inter-VLAN Routing & Dynamic DHCP
- Enabled IP routing (`ip routing`) on the Cisco Catalyst 3560 Core Switch[cite: 5].
- SVI addresses serve as default gateways across each segment[cite: 5].
- Configured DHCP server pools directly on the switch with excluded address ranges (`192.168.x.1` to `192.168.x.20`) reserved for gateways, management, and fixed network devices[cite: 5].

- <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/09e48567-2177-4a3e-a21d-be7967d839d5" />


### 3. Edge Routing, Default Routes & PAT (NAT Overload)
- Core Switch routes all non-local internet traffic via a default static route pointing to `10.0.0.1` (Edge-R1)[cite: 5].
- Edge Router summarizes all internal subnets (`192.168.0.0/16`) back towards the Multilayer Switch[cite: 5].
- Dynamic **Port Address Translation (PAT)** translates private internal IPs over the public WAN interface (`Gig0/0`) using a standard Access List (`access-list 1 permit 192.168.0.0 0.0.255.255`)[cite: 5].

- <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5c3aede1-d3a4-4c4f-8386-521873ffbfe2" />


### 4. Advanced Layer 2 & Layer 3 Security Hardening
- **Port Security:** Enforced on all cash register ports[cite: 5]:
  ```cisco
  switchport mode access
  switchport port-security
  switchport port-security maximum 1
  switchport port-security mac-address sticky
  switchport port-security violation shutdown


  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a8cae4a9-ee75-40e8-a8ea-ade14bcd5cfb" />



  Verification & Proof of Work
End-to-End Connectivity Test: POS-2 to Payment Server
Action: Executed end-to-end ping from Point-of-Sale terminal POS-2 (192.168.10.22) towards simulated Payment Server (198.51.100.2).

Result: Successful reply packets verified. The traffic successfully routes from the Access switch, through Layer 3 SVI routing on the Multilayer Switch, across the point-to-point link to Edge-R1 (with NAT/PAT translation), traversing the ISP to the destination server.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5b9bcc5f-4c86-4056-932d-6244cf875822" />




---

## 🔑 Device Credentials & Wi-Fi Access

To inspect configurations or test devices in the CLI, use the following credentials:

| Resource / Parameter | Access Type | Value / Credential |
| :--- | :--- | :--- |
| **All Switches & Routers** | Privilege Exec Mode (`enable`) | `Retail123!` |
| **Console / VTY Access** | Line Password | `Retail123!` |
| **Guest Wi-Fi (SSID: Retail-Guest)** | WPA2-PSK Authentication | `RetailGuest123` |

---

## 🗂️ How to Run & Verify This Project

Follow these steps to run, inspect, and validate the network lab on your local machine:

### 1. Prerequisites
- Install **Cisco Packet Tracer** (v8.0 or newer recommended).

### 2. Download the Repository
Clone this repository to your local computer:
```bash
git clone [https://github.com/Asmeer12/Retail-Store-Network-Packet-Tracer.git](https://github.com/Asmeer12/Retail-Store-Network-Packet-Tracer.git)
