# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — Final Submission

### Added
- Complete 3-floor hostel network topology in Cisco Packet Tracer
- 5 VLANs: Admin (10), Security (20), Students (30), IoT (40), Emergency (50)
- Router-on-a-Stick inter-VLAN routing with 802.1Q encapsulation on all 5 VLANs
- Centralized DHCP server (Server0) with per-VLAN address pools
- NAT/PAT configuration allowing all internal VLANs to share one public IP
- Local DNS with A records: server.local, admin-pc.local, lock1.local, emergency.local
- Extended ACL `block-Students` — prevents VLAN 30 from reaching Admin VLAN 10
- Extended ACL `server-only-Admin` — restricts server access to Admin VLAN only
- Extended ACL `block-IoT` — isolates IoT devices from other internal VLANs
- Per-room wireless access points (SSIDs: room1–room8) on 1st and 2nd floors
- IoT integration: smart door lock (main gate), floor sirens (E-F1, E-F2, E-B)
- CCTV webcams on each floor and main entrance (CCTV-1F, CCTV-2F, CCTV-B)
- TabletPC-based IoT dashboard via browser (http://192.168.40.2/home.html)
- IP Phone (7960) in basement admin area

### Changed
- Consolidated Security PC (VLAN 20) and Emergency PC (VLAN 50) control into single TabletPC for simplified IoT management

### Fixed
- ACL syntax error on `block-Students` (`intfa0/0.30` corrected to `int fa0/0.30`)
- VLAN 30 subinterface DHCP helper address added to ensure students receive IPs

---

## [0.3.0] — Week 3: Smart Features & Testing

### Added
- IoT device configuration on HostelGateway
- ACL rules implemented and tested (before/after screenshots)
- DNS resolution verified from Admin PC (`ping server.local`)
- Internet connectivity verified from student laptop (`ping 203.0.113.2`)

---

## [0.2.0] — Week 2: Core Configuration

### Added
- VLAN setup on Core Switch, Basement Switch, 1st Floor Switch, 2nd Floor Switch
- Inter-VLAN routing subinterfaces configured on Router0
- DHCP pools created on Server0 for all 5 VLANs
- NAT overload configured with access-list 1
- ip helper-address added to VLANs 30, 40, 50 subinterfaces

---

## [0.1.0] — Week 1: Planning & Setup

### Added
- Network design document and IP addressing scheme
- Device placement in Packet Tracer
- Physical cabling: core switch to router, floor switches to core switch
- Initial VLAN planning and floor layout
