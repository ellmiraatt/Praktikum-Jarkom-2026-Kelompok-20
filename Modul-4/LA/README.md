# Laporan Praktikum Modul 4 - DMZ Firewall
## Kelompok 20

## 1. Topologi Jaringan
[masukkan gambar topologi]

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

## 3. Konfigurasi Perangkat

### MikroTik ISP
[screenshot konfigurasi + command]

### FortiGate
[screenshot konfigurasi + command]

### Cisco Router
[screenshot konfigurasi + command]

### Ubuntu Server DMZ
[screenshot konfigurasi + command]

## 4. Hasil Pengujian

### Test 1: Client LAN ping ke Cisco Router
[screenshot hasil ping]

### Test 2: Client LAN ping ke FortiGate
[screenshot hasil ping]

### Test 3: Client LAN ping ke Server DMZ
[screenshot hasil ping]

### Test 4: Client LAN akses web DMZ
[screenshot browser/wget]

### Test 5: Client WAN ping ke MikroTik
[screenshot hasil ping]

### Test 6: Client WAN ping ke FortiGate
[screenshot hasil ping]

### Test 7: Client WAN akses web http://10.10.10.2
[screenshot browser]

### Test 8: Client WAN ping ke LAN (GAGAL)
[screenshot 100% packet loss]

### Test 9: Client WAN ping ke IP DMZ (GAGAL)
[screenshot 100% packet loss]

### Test 10: Server DMZ ping ke Client LAN
[screenshot hasil ping]

## 5. Analisis dan Kesimpulan
[tulis analisis kelompok]
