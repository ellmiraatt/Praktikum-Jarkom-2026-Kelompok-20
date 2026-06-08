# Laporan Modul 4 - DMZ Firewall
## Kelompok 20

## 1. Topologi Jaringan
![Topologi](topologi%20tumod%204.png)

## 2. Tabel IP Address
| Perangkat | Interface | IP Address | Gateway |
|---|---|---|---|
| MikroTik ISP | ether1 | DHCP | DHCP lab |
| MikroTik ISP | ether2 | 10.10.10.1/30 | - |
| MikroTik ISP | ether3 | 172.16.100.1/24 | - |
| FortiGate | port1 | 10.10.10.2/30 | 10.10.10.1 |
| FortiGate | port2 | 10.20.20.1/30 | - |
| FortiGate | port3 | 192.168.20.1/24 | - |
| Cisco Router | Gi0/0 | 10.20.20.2/30 | - |
| Cisco Router | Gi0/1 | 192.168.10.1/24 | - |
| Client LAN | eth0 | 192.168.10.10/24 | 192.168.10.1 |
| Client WAN | eth0 | 172.16.100.10/24 | 172.16.100.1 |
| Ubuntu DMZ | eth0 | 192.168.20.10/24 | 192.168.20.1 |

## 3. Hasil Pengujian

### Test 1 & 2 - Client LAN ping ke Cisco Router dan FortiGate
![Test 1 & 2](ping%20lan%20ke%20cisco%20dan%20fortiget.png)

### Test 3 & 4 - Client LAN ping ke DMZ dan akses web
![Test 3 & 4](ping%20lan%20ke%20server%20dmz%20dan%20we.png)

### Test 5 & 6 - Client WAN ping ke MikroTik dan FortiGate
![Test 5 & 6](Wan%20ke%20mikrotik%20dan%20fortiget.png)

### Test 7 - Client WAN akses web http://10.10.10.2
![Test 7](wan%20ke%20web.png)

### Test 8 & 9 - Client WAN ping ke LAN dan DMZ (GAGAL)
![Test 8 & 9](ping%20wan%20gagal.png)

### Test 10 - Server DMZ ping ke Client LAN
![Test 10](DMZ%20ke%20LAN.png)

## 4. Kesimpulan
Implementasi DMZ Firewall menggunakan FortiGate berhasil dilakukan. Firewall berhasil memisahkan zona WAN, LAN, dan DMZ. Client WAN tidak dapat mengakses LAN maupun IP asli DMZ secara langsung, namun dapat mengakses web server melalui VIP/Port Forwarding. Semua 10 pengujian berhasil sesuai hasil yang diharapkan.
