Jasa setting Mikrotik Berpengalaman 
PURWANTO 
https://wa.me/6282233483221

### 1. **Konfigurasi IP Address**
Konfigurasi dua IP address, satu untuk interface yang terkoneksi ke penyedia IP transit (ether1) dan satu untuk subnet yang akan diiklankan (ether3).
```bash
/ip address
add address=36.92.254.158/30 interface=ether1 network=36.92.254.156
add address=160.191.62.1/24 interface=ether3 network=160.191.62.0
```

### 2. **Konfigurasi DNS**
Menggunakan Google DNS (8.8.8.8 dan 8.8.4.4) dan mengizinkan perangkat MikroTik untuk melayani permintaan DNS dari jaringan lokal.
```bash
/ip dns
set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
```

### 3. **Konfigurasi Default Gateway**
Menambahkan default route untuk akses ke internet melalui gateway yang diberikan oleh penyedia IP transit.
```bash
/ip route
add disabled=no distance=1 dst-address=0.0.0.0/0 gateway=36.92.254.157
```

### 4. **Konfigurasi BGP**
Konfigurasi BGP agar MikroTik bisa melakukan pertukaran rute dengan penyedia IP transit (AS7713) dan mengiklankan subnet `160.191.62.0/24`.
```bash
/routing bgp connection
add address-families=ip as=153137 disabled=no hold-time=30s input.filter="" \
    local.role=ebgp name=bgp-ip-transit nexthop-choice=default output.filter-chain=bgp-out \
    redistribute=connected router-id=36.92.254.158 remote.address=36.92.254.157/32 remote.as=7713
```

### 5. **Konfigurasi BGP Filter**
Menambahkan filter untuk mengiklankan hanya subnet `160.191.62.0/24` dan men-drop rute lainnya.
```bash
/routing filter rule
add chain=bgp-out rule="if (dst in 160.191.62.0/24 && dst-len in 24-32) { accept }"
add chain=bgp-out rule="reject;"
```

### Penjelasan Konfigurasi:
1. **IP Address:**
   - `ether1` diberi IP address `36.92.254.158/30`, dengan gateway ke penyedia IP transit pada `36.92.254.157`.
   - `ether3` diberi IP address `160.191.62.1/24`, yang merupakan subnet yang akan diiklankan melalui BGP.

2. **DNS:**
   - Menggunakan Google DNS untuk resolusi domain pada router dan mengizinkan permintaan DNS dari klien lokal.

3. **Default Route:**
   - Rute default (`0.0.0.0/0`) diarahkan ke gateway `36.92.254.157` untuk akses ke internet.

4. **BGP:**
   - Router diatur untuk menggunakan AS number `153137`, dan terkoneksi ke penyedia IP transit dengan AS number `7713`.
   - Subnet `160.191.62.0/24` diiklankan ke penyedia IP transit.
   - `redistribute=connected` digunakan untuk mengiklankan rute-rute connected secara otomatis.

5. **BGP Filter:**
   - Hanya subnet `160.191.62.0/24` yang akan diiklankan ke penyedia IP transit.
   - Rute lainnya akan di-drop agar tidak diiklankan secara tidak sengaja.

Dengan konfigurasi ini, MikroTik akan terhubung ke penyedia IP transit menggunakan BGP, mengiklankan jaringan `160.191.62.0/24`, dan mendapatkan akses internet melalui IP transit.
