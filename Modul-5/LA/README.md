# Laporan Praktikum Modul
## Implementasi Jaringan Enterprise HQ–Branch dengan VRRP, ISC-DHCP, FortiGate, GRE Tunnel, dan OSPF

---

## 1. Deskripsi Singkat

Topologi ini adalah simulasi jaringan enterprise yang menghubungkan 2 kantor, yaitu kantor pusat di **Jakarta (HQ)** dan kantor cabang di **Surabaya (Branch)**. Kedua kantor dihubungkan menggunakan teknologi **GRE Tunnel** agar bisa saling berkomunikasi meskipun berada di lokasi yang berbeda.

Perangkat yang digunakan antara lain: Cisco VIoS, Cisco Switch, MikroTik RouterOS, Fortinet FortiGate, Ubuntu Server, Tinycore Linux, dan VPCS.

---

## 2. Tabel Addressing

### 2.1 VLAN Jakarta

| VLAN | Nama VLAN | Network | Gateway Virtual | Keterangan |
|------|-----------|---------|-----------------|------------|
| 10 | FINANCE | 192.168.10.0/24 | 192.168.10.1 | DHCP dari Ubuntu Server Jakarta |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | DHCP dari Ubuntu Server Jakarta |
| 60 | SERVER-HQ | 192.168.60.0/24 | 192.168.60.1 | VLAN server Ubuntu Jakarta |

### 2.2 IP Address Cisco Router Jakarta

| Interface | VLAN / Link | IP Address | Keterangan |
|-----------|------------|------------|------------|
| Gi0/1.10 | VLAN 10 | 192.168.10.2/24 | IP fisik Cisco untuk VLAN 10 |
| Gi0/1.20 | VLAN 20 | 192.168.20.2/24 | IP fisik Cisco untuk VLAN 20 |
| Gi0/1.60 | VLAN 60 | 192.168.60.2/24 | IP fisik Cisco untuk VLAN 60 |
| Gi0/0 | Link ke FortiGate Jakarta | 10.10.100.2/30 | Transit Cisco Jakarta ke FortiGate Jakarta |

### 2.3 IP Address MikroTik Router Jakarta

| Interface | VLAN / Link | IP Address | Keterangan |
|-----------|------------|------------|------------|
| vlan10-finance | VLAN 10 | 192.168.10.3/24 | IP fisik MikroTik untuk VLAN 10 |
| vlan20-it | VLAN 20 | 192.168.20.3/24 | IP fisik MikroTik untuk VLAN 20 |
| vlan60-server | VLAN 60 | 192.168.60.3/24 | IP fisik MikroTik untuk VLAN 60 |
| ether1 | Link ke FortiGate Jakarta | 10.10.101.2/30 | Transit MikroTik Jakarta ke FortiGate Jakarta |

### 2.4 VRRP Jakarta

| VLAN | Virtual IP | Master | Backup | Keterangan |
|------|-----------|--------|--------|------------|
| 10 | 192.168.10.1 | Cisco Router Jakarta | MikroTik Router Jakarta | Gateway virtual VLAN 10 |
| 20 | 192.168.20.1 | MikroTik Router Jakarta | Cisco Router Jakarta | Gateway virtual VLAN 20 |
| 60 | 192.168.60.1 | Cisco Router Jakarta | MikroTik Router Jakarta | Gateway virtual VLAN 60 |

### 2.5 FortiGate Jakarta

| Interface | Terhubung ke | IP Address | Keterangan |
|-----------|-------------|------------|------------|
| port1 | Cisco Router Jakarta | 10.10.100.1/30 | Link ke Cisco Jakarta |
| port2 | MikroTik Router Jakarta | 10.10.101.1/30 | Link ke MikroTik Jakarta |
| port3 | MikroTik ISP | 10.0.12.2/30 | Link WAN ke ISP |
| GRE-JKT-SBY | FortiGate Surabaya | 172.16.0.1/32 | IP GRE Tunnel Jakarta |

### 2.6 MikroTik ISP

| Interface | Terhubung ke | IP Address | Keterangan |
|-----------|-------------|------------|------------|
| ether1 | Cloud NAT | DHCP | Akses internet simulasi |
| ether2 | FortiGate Jakarta | 10.0.12.1/30 | Link ISP ke Jakarta |
| ether3 | FortiGate Surabaya | 10.0.13.1/30 | Link ISP ke Surabaya |

### 2.7 GRE Tunnel Jakarta–Surabaya

| Tunnel | Perangkat | Local WAN | Remote WAN | Tunnel IP |
|--------|----------|-----------|-----------|-----------|
| GRE-JKT-SBY | FortiGate Jakarta | 10.0.12.2 | 10.0.13.2 | 172.16.0.1/32 |
| GRE-SBY-JKT | FortiGate Surabaya | 10.0.13.2 | 10.0.12.2 | 172.16.0.2/32 |

---

## 3. Hasil Konfigurasi dan Bukti Screenshot

---

### Tugas Modul 1 — Konfigurasi Cisco Switch Jakarta

**Perangkat:** Cisco Switch Jakarta

**Konfigurasi yang dilakukan:**
- Membuat VLAN 10 (FINANCE), VLAN 20 (IT), dan VLAN 60 (SERVER-HQ)
- Mengatur port client sebagai access port pada VLAN yang sesuai
- Mengatur link ke Cisco Router dan MikroTik Router sebagai trunk (membawa VLAN 10, 20, 60)

#### Screenshot `show vlan brief`

![show vlan brief](show_vlan_brief_jakarta.png)

Hasil `show vlan brief` menunjukkan:
- VLAN 10 (FINANCE) aktif dengan port Gi0/1 sebagai access VLAN 10
- VLAN 20 (IT) aktif dengan port Gi0/2 sebagai access VLAN 20
- VLAN 60 (SERVER-HQ) aktif dengan port Gi0/3 dan Gi1/0 sebagai access VLAN 60

#### Screenshot `show interfaces trunk`

![show interfaces trunk](show_interfaces_trunk_switch_jakarta.png)

Hasil `show interfaces trunk` menunjukkan:
- Port Gi0/0 dan Gi0/1 berstatus trunking dengan enkapsulasi 802.1q
- Kedua port trunk membawa VLAN 10, 20, dan 60
- Native VLAN adalah VLAN 1

---

### Tugas Modul 2 — Konfigurasi Cisco Router Jakarta

**Perangkat:** Cisco Router Jakarta

**Konfigurasi yang dilakukan:**
- Membuat subinterface Gi0/1.10, Gi0/1.20, Gi0/1.60 untuk masing-masing VLAN
- Mengkonfigurasi VRRP untuk VLAN 10, 20, dan 60
- Cisco sebagai VRRP Master untuk VLAN 10 dan 60 (prioritas 110)
- Mengkonfigurasi DHCP Relay menuju Ubuntu Server (192.168.60.10)
- Mengkonfigurasi link Gi0/0 ke FortiGate Jakarta (10.10.100.2/30)

#### Screenshot `show ip interface brief`

![show ip interface brief](show_ip_interface_brief_cisco_jakarta.png)

Interface yang aktif:
- Gi0/0: 10.10.100.2 (link ke FortiGate Jakarta)
- Gi0/1.10: 192.168.10.2 (subinterface VLAN 10)
- Gi0/1.20: 192.168.20.2 (subinterface VLAN 20)
- Gi0/1.60: 192.168.60.2 (subinterface VLAN 60)

#### Screenshot `show vrrp brief`

![show vrrp brief](show_vrrp_brief_cisco_jakarta.png)

Status VRRP:
- Gi0/1.10 (VLAN 10): **Master** dengan prioritas 110, Group addr 192.168.10.1
- Gi0/1.20 (VLAN 20): **Master** dengan prioritas 90, Group addr 192.168.20.1 (Backup di Cisco)
- Gi0/1.60 (VLAN 60): **Master** dengan prioritas 110, Group addr 192.168.60.1

#### Screenshot Ping dari Cisco Router ke FortiGate Jakarta

![ping cisco ke fortigate](cisco_jakarta_ping_ke_fortiget_jakarta.png)

```
CISCO-JAKARTA#ping 10.10.100.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
```

Cisco Router Jakarta berhasil ping ke FortiGate Jakarta (10.10.100.1) dengan success rate 100%.

---

### Tugas Modul 3 — Konfigurasi MikroTik Router Jakarta

**Perangkat:** MikroTik Router Jakarta

**Konfigurasi yang dilakukan:**
- Membuat VLAN interface untuk VLAN 10, 20, dan 60
- Mengkonfigurasi VRRP (Master untuk VLAN 20 dengan prioritas 120)
- Mengkonfigurasi DHCP Relay menuju Ubuntu Server (192.168.60.10)
- Mengkonfigurasi link ether1 ke FortiGate Jakarta
- Menambahkan default route menuju FortiGate Jakarta

#### Screenshot `/ip address print`

![ip address print mikrotik jakarta](ip_address_print_mikrotik_jakarta.png)

IP Address yang terkonfigurasi:
- `192.168.10.3/24` pada vlan10-finance
- `192.168.20.3/24` pada vlan20-it
- `192.168.60.3/24` pada vlan60-ubuntu-server
- `10.10.101.2/30` pada ether1 (link ke FortiGate)
- VRRP IP: `192.168.10.1/32`, `192.168.20.1/32`, `192.168.60.1/32`

#### Screenshot `/ip dhcp-relay print`

![ip dhcp-relay print](ip_dhcp-relay_print_mikrotik_jakarta.png)

DHCP Relay dikonfigurasi untuk:
- relay-vlan10: interface vlan10-finance → DHCP Server 192.168.60.10
- relay-vlan20: interface vlan20-it → DHCP Server 192.168.60.10

> **Catatan:** Pada screenshot terlihat flag `XI` (disabled + invalid). Pastikan relay diaktifkan agar DHCP berfungsi.

#### Screenshot `/ip route print`

![ip route print mikrotik jakarta](ip_route_print_mikrotik_jakarta.png)

Routing table MikroTik Jakarta:
- Default route `0.0.0.0/0` via `10.10.101.1` (FortiGate Jakarta)
- Connected route untuk semua interface dan VRRP

#### Screenshot Ping dari MikroTik Jakarta ke FortiGate Jakarta

![ping mikrotik ke fortigate](mikrotik_jakarta_ping_ke_fortinet_jakarta.png)

MikroTik Jakarta berhasil ping ke FortiGate Jakarta (10.10.101.1) dengan packet-loss=0%.

---

### Tugas Modul 5 — Konfigurasi FortiGate Jakarta

**Perangkat:** FortiGate Jakarta

**Konfigurasi yang dilakukan:**
- Interface port1 ke Cisco Router (10.10.100.1/30)
- Interface port2 ke MikroTik Router (10.10.101.1/30)
- Interface port3 ke MikroTik ISP / WAN (10.0.12.2/30)
- Default route menuju MikroTik ISP
- Static route menuju jaringan internal Jakarta
- Firewall policy dan NAT untuk internet
- GRE Tunnel menuju FortiGate Surabaya
- OSPF over GRE dengan redistribute static route

#### Screenshot `get system interface physical`

![get system interface physical](get_system_interface_physical_fortinet_jakarta.png)

Status interface FortiGate Jakarta:
- port1: `10.10.100.1/30` — UP (link ke Cisco Jakarta)
- port2: `10.10.101.1/30` — UP (link ke MikroTik Jakarta)
- port3: `10.0.12.2/30` — UP (link WAN ke ISP)
- port4: down (tidak digunakan)

#### Screenshot `get router info routing-table all`

![routing table all](get_router_info_routing-table_fortiget_jakarta.png)

Routing table FortiGate Jakarta mencakup:
- `S* 0.0.0.0/0` via 10.0.12.1 (default route ke ISP)
- `C 10.0.12.0/30` connected port3
- `C 10.10.100.0/30` connected port1
- `C 10.10.101.0/30` connected port2
- `C 172.16.0.1/32` dan `172.16.0.2/32` connected GRE-JKT-SBY
- `S 192.168.10.0/24`, `S 192.168.20.0/24`, `S 192.168.60.0/24` via 10.10.100.2 (static route ke VLAN Jakarta)
- `O E2 192.168.30.0/24` dan `O E2 192.168.40.0/24` via 172.16.0.2 GRE-JKT-SBY (route Surabaya via OSPF)

#### Screenshot Ping ke 8.8.8.8

![ping 8.8.8.8 fortigate jakarta](execute_ping_8_8_8_8_fortiget_jakarta.png)

```
Fortinet-Jakarta # execute ping 8.8.8.8
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 21.4/21.6/21.8 ms
```

FortiGate Jakarta berhasil mengakses internet (8.8.8.8) dengan 0% packet loss.

#### Screenshot Ping ke IP Tunnel Surabaya (172.16.0.2)

![ping tunnel surabaya](fortiget_jakarta_ping_fortiget_surabaya.png)

```
Fortinet-Jakarta # execute ping 172.16.0.2
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.0/1.3/1.8 ms
```

GRE Tunnel aktif — FortiGate Jakarta berhasil ping FortiGate Surabaya melalui tunnel.

#### Screenshot `get router info ospf neighbor`

![ospf neighbor](get_router_info_ospf_neighbor_fortiget_jakarta.png)

```
OSPF process 0, VRF 0:
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   Full/ -         00:00:30    172.16.0.2      GRE-JKT-SBY
```

OSPF Neighbor dengan FortiGate Surabaya (Router ID 2.2.2.2) berstatus **Full** melalui interface GRE-JKT-SBY.

#### Screenshot `get router info routing-table ospf`

![ospf routing table](get_router_info_routing-table_ospf_fortiget_jakarta.png)

Route yang diterima dari Surabaya via OSPF:
- `O E2 192.168.30.0/24` via 172.16.0.2, GRE-JKT-SBY (VLAN Sales Surabaya)
- `O E2 192.168.40.0/24` via 172.16.0.2, GRE-JKT-SBY (VLAN Operations Surabaya)

---

### Tugas Modul 6 — Konfigurasi MikroTik ISP

**Perangkat:** MikroTik ISP

**Konfigurasi yang dilakukan:**
- IP ether1: DHCP dari Cloud NAT PNETLab
- IP ether2: 10.0.12.1/30 (link ke FortiGate Jakarta)
- IP ether3: 10.0.13.1/30 (link ke FortiGate Surabaya)
- Default route menuju Cloud NAT
- NAT Masquerade via ether1 untuk akses internet

#### Screenshot `/ip address print`

![ip address print mikrotik isp](ip_address_print_mikrotik_isp.png)

IP Address MikroTik ISP:
- `10.0.12.1/30` pada ether2 (ke FortiGate Jakarta)
- `10.0.13.1/30` pada ether3 (ke FortiGate Surabaya)
- `10.4.89.200/24` pada ether1 (DHCP dari cloud — nilai dinamis, bisa berbeda)

#### Screenshot `/ip route print`

![ip route print mikrotik isp](ip_route_print_mikrotik_isp.png)

Routing table MikroTik ISP:
- `S 0.0.0.0/0` via 10.4.89.1 (default route ke cloud NAT)
- `C 10.0.12.0/30` via ether2
- `C 10.0.13.0/30` via ether3
- `C 10.4.89.0/24` via ether1

#### Screenshot `/ip firewall nat print`

![ip firewall nat print](ip_firewall_nat_print_mikrotik_isp.png)

```
0   chain=srcnat action=masquerade out-interface=ether1
```

NAT Masquerade aktif pada ether1 sehingga semua traffic dari jaringan lab dapat mengakses internet melalui Cloud NAT.

---

### Tugas Modul 4 — Konfigurasi Ubuntu Server Jakarta

**Perangkat:** Ubuntu Server Jakarta

**Konfigurasi yang dilakukan:**
- Konfigurasi IP static pada VLAN 60 (192.168.60.10/24)
- Default gateway ke VRRP virtual IP VLAN 60 (192.168.60.1)
- Instalasi ISC-DHCP Server dan konfigurasi pool VLAN 10 dan VLAN 20
- Instalasi Nginx sebagai web server Jakarta

> **⚠️ Catatan Kendala — Screenshot Nginx tidak tersedia**
>
> Pada saat pengerjaan modul ini, Ubuntu Server **tidak dapat menginstall Nginx** karena kendala koneksi jaringan saat proses `apt install nginx -y`. Paket tidak dapat diunduh dari repository sehingga instalasi Nginx tidak berhasil diselesaikan. Oleh karena itu, screenshot tampilan web server Nginx dari browser tidak dapat disertakan dalam laporan ini. Screenshot untuk `ip a`, `ip route`, isi `/etc/dhcp/dhcpd.conf`, dan ping 8.8.8.8 sudah didokumentasikan pada sesi sebelumnya.

---

### Tugas Modul 6 — Konfigurasi MikroTik ISP (Lanjutan — Bukti Ping)

**Perangkat:** MikroTik ISP

#### Screenshot Ping ke 8.8.8.8

![ping 8.8.8.8 mikrotik isp](ping_8_8_8_8_mikrotik_isp.png)

MikroTik ISP berhasil ping ke internet (8.8.8.8) dengan sent=6, received=6, packet-loss=0%, avg-rtt=20ms. Membuktikan koneksi ke Cloud NAT berjalan normal.

#### Screenshot Ping ke FortiGate Jakarta (10.0.12.2)

![ping ke fortigate jakarta](ping_10_0_12_2.png)

MikroTik ISP berhasil ping ke FortiGate Jakarta (10.0.12.2) dengan packet-loss=0%. Koneksi link ISP ↔ Jakarta aktif.

#### Screenshot Ping ke FortiGate Surabaya (10.0.13.2)

![ping ke fortigate surabaya](ping_10_0_13_2_mikrotik_isp.png)

MikroTik ISP berhasil ping ke FortiGate Surabaya (10.0.13.2) dengan packet-loss=0%. Koneksi link ISP ↔ Surabaya aktif. Kedua FortiGate saling reachable melalui MikroTik ISP, yang merupakan syarat agar GRE Tunnel dapat terbentuk.

---

### Tugas Modul 7 — Konfigurasi Switch dan MikroTik Surabaya

**Perangkat:** Cisco Switch Surabaya dan MikroTik Router Surabaya

**Konfigurasi yang dilakukan:**
- Membuat VLAN 30 (sales) dan VLAN 40 (operations) di Switch Surabaya
- Mengatur port Gi0/1 sebagai access VLAN 30, port Gi0/2 dan Gi0/3 sebagai access VLAN 40
- Mengatur Gi0/0 sebagai trunk membawa VLAN 30 dan 40
- Membuat VLAN interface vlan30-sales dan vlan40-operations di MikroTik Surabaya
- Mengkonfigurasi DHCP Server lokal untuk VLAN 30 (range 192.168.30.100–192.168.30.200)
- Mengkonfigurasi link ether1 ke FortiGate Surabaya (10.10.200.2/30)
- Menambahkan default route menuju FortiGate Surabaya

#### Screenshot `show vlan brief` — Switch Surabaya

![show vlan brief surabaya](show_vlan_brief_surabaya.png)

VLAN yang aktif di Switch Surabaya:
- VLAN 30 (sales): aktif, port Gi0/1
- VLAN 40 (operations): aktif, port Gi0/2 dan Gi0/3

#### Screenshot `show interfaces trunk` — Switch Surabaya

![show interfaces trunk surabaya](show_interfaces_trunk_switch_surabaya.png)

Port Gi0/0 berstatus trunking (802.1q), membawa VLAN 30 dan 40. Native VLAN adalah VLAN 1.

#### Screenshot `/ip address print` — MikroTik Surabaya

![ip address print mikrotik surabaya](ip_address_print_mikrotik_surabaya.png)

IP Address MikroTik Surabaya:
- `10.10.200.2/30` pada ether1 (link ke FortiGate Surabaya)
- `192.168.30.1/24` pada vlan30-sales (gateway VLAN 30)
- `192.168.40.1/24` pada vlan40-operations (gateway VLAN 40)

#### Screenshot `/ip dhcp-server print` — MikroTik Surabaya

![ip dhcp-server print](ip_dhcp_server_print_mikrotik_surabaya.png)

DHCP Server `dhcp1` aktif pada interface vlan30-sales, menggunakan pool `dhcp_pool0` dengan lease-time 10 menit.

#### Screenshot `/ip pool print` — MikroTik Surabaya

![ip pool print](ip_pool_print_mikrotik_surabaya.png)

Pool DHCP untuk VLAN 30 Sales: range `192.168.30.100 – 192.168.30.200`, sesuai ketentuan modul.

#### Screenshot `/ip route print` — MikroTik Surabaya

![ip route print mikrotik surabaya](ip_route_print_mikrotik_surabaya.png)

Routing table MikroTik Surabaya:
- Default route `0.0.0.0/0` via `10.10.200.1` (FortiGate Surabaya)
- Connected route untuk ether1, vlan30-sales, dan vlan40-operations

#### Screenshot Ping dari MikroTik Surabaya ke 8.8.8.8

![ping 8.8.8.8 mikrotik surabaya](ping_8_8_8_8_mikrotik_surabaya.png)

MikroTik Surabaya berhasil ping ke internet (8.8.8.8) dengan packet-loss=0%, membuktikan koneksi dari jaringan internal Surabaya ke internet berjalan melalui FortiGate Surabaya → MikroTik ISP → Cloud NAT.

---

### Tugas Modul 8 — Konfigurasi FortiGate Surabaya

**Perangkat:** FortiGate Surabaya

**Konfigurasi yang dilakukan:**
- Interface port1 ke MikroTik ISP (WAN): 10.0.13.2/30
- Interface port2 ke MikroTik Surabaya (LAN): 10.10.200.1/30
- Default route menuju MikroTik ISP (10.0.13.1)
- Static route menuju VLAN 30 dan VLAN 40 via MikroTik Surabaya
- Firewall policy dan NAT untuk akses internet
- GRE Tunnel (GRE-SBY-JKT) menuju FortiGate Jakarta
- OSPF over GRE dengan redistribute static route

#### Screenshot `get system interface physical`

![get system interface physical surabaya](get_system_interface_physical_fortinet_surabaya.png)

Status interface FortiGate Surabaya:
- port1: `10.0.13.2/30` — UP (link WAN ke MikroTik ISP)
- port2: `10.10.200.1/30` — UP (link ke MikroTik Surabaya)
- port3 dan port4: down (tidak digunakan)

#### Screenshot `get router info routing-table all`

![routing table all surabaya](get_router_info_routing-table_fortiget_surabaya.png)

Routing table lengkap FortiGate Surabaya:
- `S* 0.0.0.0/0` via 10.0.13.1, port1 (default route ke ISP)
- `C 10.0.13.0/30` connected port1
- `C 10.10.200.0/30` connected port2
- `C 172.16.0.1/32` dan `172.16.0.2/32` connected GRE-SBY-JKT (tunnel aktif)
- `O E2 192.168.10.0/24`, `192.168.20.0/24`, `192.168.60.0/24` via 172.16.0.1 GRE-SBY-JKT (route Jakarta via OSPF)
- `S 192.168.30.0/24` dan `S 192.168.40.0/24` via 10.10.200.2, port2 (static route ke VLAN Surabaya)

#### Screenshot Ping ke 8.8.8.8 — FortiGate Surabaya

![ping 8.8.8.8 fortigate surabaya](ping_8_8_8_8_mikrotik_surabaya.png)

FortiGate Surabaya berhasil mengakses internet (8.8.8.8) dengan 0% packet loss.

#### Screenshot Ping ke IP Tunnel Jakarta (172.16.0.1)

![ping tunnel jakarta](ping_172_16_0_1_fortiget_surabaya.png)

```
Fortinet-Surabaya # execute ping 172.16.0.1
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.2/1.3/1.5 ms
```

GRE Tunnel aktif — FortiGate Surabaya berhasil ping FortiGate Jakarta melalui tunnel dengan latensi sangat rendah.

#### Screenshot `get router info ospf neighbor`

![ospf neighbor surabaya](get_router_info_ospf_neighbor_fortiget_surabaya.png)

```
OSPF process 0, VRF 0:
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   Full/ -         00:00:39    172.16.0.1      GRE-SBY-JKT
```

OSPF Neighbor dengan FortiGate Jakarta (Router ID 1.1.1.1) berstatus **Full** melalui interface GRE-SBY-JKT.

#### Screenshot `get router info routing-table ospf`

![ospf routing table surabaya](get_router_info_routing-table_ospf_fortiget_surabaya.png)

Route yang diterima dari Jakarta via OSPF:
- `O E2 192.168.10.0/24` via 172.16.0.1, GRE-SBY-JKT (VLAN Finance Jakarta)
- `O E2 192.168.20.0/24` via 172.16.0.1, GRE-SBY-JKT (VLAN IT Jakarta)
- `O E2 192.168.60.0/24` via 172.16.0.1, GRE-SBY-JKT (VLAN Server Jakarta)

---

## 4. Analisis Jalur Traffic Jakarta ke Surabaya

Berdasarkan routing table dan konfigurasi yang telah dilakukan, berikut adalah jalur traffic dari Client Jakarta ke Client Surabaya:

```
Client Jakarta (VLAN 10/20/60)
  → VRRP Gateway Virtual (192.168.10.1 / 192.168.20.1 / 192.168.60.1)
    → Cisco Router Jakarta atau MikroTik Jakarta (sesuai master VRRP)
      → FortiGate Jakarta (10.10.100.1 atau 10.10.101.1)
        → GRE Tunnel (172.16.0.1 → 172.16.0.2)
          → FortiGate Surabaya
            → MikroTik Surabaya
              → Client Surabaya (VLAN 30/40)
```

Rute dikelola secara dinamis oleh **OSPF over GRE**:
- FortiGate Jakarta meng-advertise route 192.168.10.0/24, 192.168.20.0/24, 192.168.60.0/24 ke OSPF
- FortiGate Surabaya meng-advertise route 192.168.30.0/24, 192.168.40.0/24 ke OSPF
- Kedua sisi saling menerima route sebagai **OSPF External Type 2 (E2)** karena menggunakan redistribute static

Untuk **failover gateway Jakarta**, VRRP memastikan bahwa jika salah satu router (Cisco atau MikroTik) down, router lainnya otomatis mengambil alih sebagai Master tanpa intervensi manual.

---

## 5. Kesimpulan

Praktikum ini berhasil mengimplementasikan jaringan enterprise yang menghubungkan kantor Jakarta (HQ) dan Surabaya (Branch). VLAN, VRRP, dan DHCP terpusat berjalan dengan baik di sisi Jakarta, begitu pula VLAN dan DHCP lokal di sisi Surabaya. GRE Tunnel antar-FortiGate berhasil terbentuk, dan OSPF over GRE berhasil mendistribusikan route antar-site secara dinamis dengan status neighbor Full. Seluruh perangkat dapat mengakses internet melalui MikroTik ISP dengan NAT Masquerade. Satu-satunya kendala yang dihadapi adalah instalasi Nginx di Ubuntu Server yang tidak berhasil diselesaikan karena paket tidak dapat diunduh saat menjalankan `apt install nginx -y` akibat kendala koneksi ke repository.
