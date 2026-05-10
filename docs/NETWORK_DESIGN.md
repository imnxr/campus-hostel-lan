# Network Design Document
## Campus Hostel LAN — Technical Reference

---

## 1. Design Goals

| Goal | Implementation |
|---|---|
| Isolate user groups | 5 VLANs — Admin, Security, Students, IoT, Emergency |
| Centralized services | Single server handles DHCP, DNS, and IoT for all VLANs |
| Internet access for all | NAT/PAT on router using one public IP |
| Block students from admin | Extended ACL `block-Students` on VLAN 30 subinterface |
| Remote IoT control | TabletPC accesses IoT server via browser at 192.168.40.2 |
| Wireless per room | Access points with individual SSIDs (room1–room8) |

---

## 2. Physical Topology

```
                        [Internet / Cloud-PT]
                               |
                        203.0.113.2 (Internet PC)
                               |
                        203.0.113.1 fa0/1
                           [Router0]
                        fa0/0 (trunk to core)
                               |
                        [Core Switch]
                       /       |        \
                      /        |         \
              [Basement SW] [1F Switch] [2F Switch]
                  |               |           |
          Admin/Security     4 rooms      4 rooms
          IoT Gateway        (VLAN 30)    (VLAN 30)
          (VLAN 10/20/40/50)
```

---

## 3. VLAN Table

| VLAN | Name | Subnet | Gateway | DHCP Range | Notes |
|---|---|---|---|---|---|
| 10 | Admin | 192.168.10.0/24 | 192.168.10.1 | Static only | Server + Admin PC |
| 20 | Security | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.100+ | CCTV PCs |
| 30 | Students | 192.168.30.0/24 | 192.168.30.1 | 192.168.30.100+ | Laptops + phones |
| 40 | IoT | 192.168.40.0/24 | 192.168.40.1 | 192.168.40.100+ | Smart devices |
| 50 | Emergency | 192.168.50.0/24 | 192.168.50.1 | 192.168.50.100+ | Alarms |

---

## 4. Device Inventory

### Basement
| Device | Type | IP | VLAN |
|---|---|---|---|
| Server0 | Server-PT | 192.168.10.2 (static) | 10 |
| Admin PC | PC-PT | 192.168.10.10 (static) | 10 |
| TabletPC | Tablet-PT | DHCP (192.168.10.x) | 10 |
| IP Phone0 | 7960 IP Phone | DHCP | 10 |
| Security PC 1 | PC-PT | DHCP (192.168.20.x) | 20 |
| Security PC 2 | PC-PT | DHCP (192.168.20.x) | 20 |
| HostelGateway | Home Gateway | 192.168.40.2 (static) | 40 |
| CCTV-B | Webcam (IoT) | DHCP (192.168.40.x) | 40 |
| E-B (Siren) | Siren (IoT) | DHCP (192.168.40.x) | 40 |
| Main-Gate | Smart Door | DHCP (192.168.40.x) | 40 |

### 1st Floor (Rooms 5–8)
| Device | Type | IP | VLAN |
|---|---|---|---|
| Room-5 to Room-8 | AccessPoint-PT | N/A (bridge) | 30 |
| Laptop3–Laptop6 | Laptop-PT | DHCP (192.168.30.x) | 30 |
| Smartphone23–26 | Smartphone-PT | DHCP (192.168.30.x) | 30 |
| CCTV-1F | Webcam (IoT) | DHCP (192.168.40.x) | 40 |
| E-F1 (Siren) | Siren (IoT) | DHCP (192.168.40.x) | 40 |

### 2nd Floor (Rooms 9–12 / Rooms B)
| Device | Type | IP | VLAN |
|---|---|---|---|
| Room-9 to Room-12 | AccessPoint-PT | N/A (bridge) | 30 |
| Laptop7–Laptop10 | Laptop-PT | DHCP (192.168.30.x) | 30 |
| Smartphone27–30 | Smartphone-PT | DHCP (192.168.30.x) | 30 |
| CCTV-2F | Webcam (IoT) | DHCP (192.168.40.x) | 40 |
| E-F2 (Siren) | Siren (IoT) | DHCP (192.168.40.x) | 40 |

---

## 5. Router Subinterface Summary

| Subinterface | VLAN | IP Address | Purpose |
|---|---|---|---|
| fa0/0.10 | 10 | 192.168.10.1/24 | Admin gateway |
| fa0/0.20 | 20 | 192.168.20.1/24 | Security gateway |
| fa0/0.30 | 30 | 192.168.30.1/24 | Students gateway |
| fa0/0.40 | 40 | 192.168.40.1/24 | IoT gateway |
| fa0/0.50 | 50 | 192.168.50.1/24 | Emergency gateway |
| fa0/1 | — | 203.0.113.1/24 | Public IP (NAT Outside) |

---

## 6. ACL Policy Table

| ACL Name | Blocks | Permits | Applied On |
|---|---|---|---|
| block-Students | VLAN 30 → VLAN 10 | Everything else | fa0/0.30 inbound |
| server-only-Admin | Any → 192.168.10.2 (except VLAN 10) | VLAN 10 → server | fa0/0.20, .30, .40 inbound |
| block-IoT | VLAN 40 → any 192.168.x.x | Everything else | fa0/0.40 inbound |

---

## 7. IoT Device Control Flow

```
TabletPC (VLAN 10)
   |
   | HTTP GET http://192.168.40.2/home.html
   |
HostelGateway IoT Server (192.168.40.2)
   |
   |── CCTV-1F  (webcam, 1st floor)
   |── CCTV-2F  (webcam, 2nd floor)
   |── CCTV-B   (webcam, basement/entrance)
   |── E-F1     (siren, 1st floor)
   |── E-F2     (siren, 2nd floor)
   |── E-B      (siren, basement)
   └── Main-Gate (smart door lock)
```

The TabletPC opens a web browser to the IoT server's dashboard. From there, security/admin staff can:
- Toggle each siren ON/OFF
- Lock/unlock the main gate door
- View live webcam feeds from each floor

---

## 8. Known Limitations & Design Decisions

1. **Wireless security disabled** — Access points have no WPA2 passphrase. In a real network this would be a critical vulnerability. Set WPA2-PSK on each AP for production use.

2. **Single router / no redundancy** — Router0 is a single point of failure. A real deployment would add a secondary router with HSRP.

3. **Security and Emergency VLANs consolidated to TabletPC** — Originally the Security PCs (VLAN 20) and Emergency PC (VLAN 50) had separate controllers. These were simplified to one TabletPC for ease of IoT management in the simulation.

4. **Static routing only** — The simulation uses static routes implied by connected routes. EIGRP or OSPF would be more scalable in a multi-router setup.

5. **CCTV via IoT webcam** — Packet Tracer simulates CCTV using IoT webcam devices connected through the HostelGateway. In reality, CCTV runs on a dedicated NVR system.
