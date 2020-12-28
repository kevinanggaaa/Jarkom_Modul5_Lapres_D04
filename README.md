# Jarkom_Modul5_Lapres_D04

## Tim D4

## Kevin Angga Wijaya - 05111840000024

## Samuel C. Y. Sinambela -05111840000036

### Membuat topologi sesuai ketentuan

![topologi](/img/topologi.JPG)

### Melakukan subnetting dan routing menggunakan cidr

![hitung](/img/hitung1.JPG)

![hitung](/img/hitung2.JPG)

![hitung](/img/hitung3.JPG)

![subnetting](/img/subnetting.JPG)

![cidr](/img/cidr.JPG)

![tree](/img/tree.JPG)

### Menyetting DHCP server

### 1. mengkonfigurasi SURABAYA menggunakan iptables tanpa menggunakan MASQUERADE.

```
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 10.151.78.22 (SURABAYA)

```

### 2. mendrop semua akses SSH dari luar Topologi (UML) pada server yang memiliki ip DMZ (DHCP dan DNS SERVER) pada SURABAYA.

```
iptables -A FORWARD -p tcp --dport 22 -d 10.151.79.40/29 -i eth0 -j DROP (SURABAYA)

```

### 3. membatasi DHCP dan DNS server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing masing server, selebihnya akan di DROP.

```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP (MALANG & MOJOKERTO)

```

### 4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat.

```
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT (MALANG & (SUBNET SIDOARJO))
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 17:01 --timestop 07:59 -j REJECT (MALANG & (SUBNET SIDOARJO))


```

### 5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya.

```
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:01 --timestop 16:59 -j REJECT (MALANG & (SUBNET GRESIK))

```

### 6. SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

```
iptables -A INPUT -s 192.168.5.2,192.168.5.3 -d 10.151.79.43 -p tcp --dport 80 -j ACCEPT

```

### 7. semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.

```
iptables -N LOGGING
iptables -A FORWARD -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
(SURABAYA)

iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
(MALANG)

iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
(MOJOKERTO)


```
