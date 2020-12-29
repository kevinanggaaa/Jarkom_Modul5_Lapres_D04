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

### Melakukan konfigurasi UML

```
SURABAYA

auto eth0
iface eth0 inet static
address 10.151.78.22
netmask 255.255.255.252
gateway 10.151.78.21

auto eth1
iface eth1 inet static
address 192.168.6.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.1.1
netmask 255.255.255.252
ip route add 192.168.0.0/24 via 192.168.1.2 dev eth2
ip route add 192.168.4.0/23 via 192.168.6.2 dev eth1
ip route add 10.151.79.40/29 via 192.168.1.2 dev eth2

```

```
KEDIRI

auto eth0
iface eth0 inet static
address 192.168.6.2
netmask 255.255.255.252
gateway 192.168.6.1

auto eth1
iface eth1 inet static
address 192.168.5.1
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.4.1
netmask 255.255.255.0

```

```
BATU

auto eth0
iface eth0 inet static
address 10.151.79.41
netmask 255.255.255.248

auto eth1
iface eth1 inet static
address 192.168.1.2
netmask 255.255.255.252
gateway 192.168.1.1

auto eth2
iface eth2 inet static
address 192.168.0.1
netmask 255.255.255.0

```

```
GRESIK

auto eth0
iface eth0 inet dhcp

```

```
SIDOARJO

auto eth0
iface eth0 inet dhcp

```

```
MALANG

auto eth0
iface eth0 inet static
address 10.151.79.43
netmask 255.255.255.248
gateway 10.151.79.41

```

```
MOJOKERTO

auto eth0
iface eth0 inet static
address 10.151.79.42
netmask 255.255.255.248
gateway 10.151.79.41

```

```
MADIUN

auto eth0
iface eth0 inet dhcp

```

```
PROBOLINGGO

auto eth0
iface eth0 inet dhcp

```

```
RELAY SURABAYA

10.151.79.42
eth1 eth2

```

```
RELAY BATU

10.151.79.42
eth0 eth1 eth2

```

```
RELAY KEDIRI

10.151.79.42
eth0 eth1 eth2

```

```
MOJOKERTO

etc/default/isc-dhcp-server
eth0

```

```
subnet 10.151.79.40 netmask 255.255.255.248 {
authoritative;
range 10.151.79.42 10.151.79.46;
default-lease-time 3600;
max-lease-time 3600;
option subnet-mask 255.255.255.248;
option broadcast-address 10.151.79.47;
option routers 10.151.79.41;
}

subnet 192.168.0.0 netmask 255.255.255.0{
range 192.168.0.2 192.168.0.202;
option routers 192.168.0.1;
option broadcast-address 192.168.0.255;
default-lease-time 3600;
max-lease-time 3600;
}

subnet 192.168.4.0 netmask 255.255.255.0{
range 192.168.4.2 192.168.4.212;
option routers 192.168.0.1;
option broadcast-address 192.168.4.255;
default-lease-time 3600;
max-lease-time 3600;
}

subnet 192.168.5.0 netmask 255.255.255.248{
range 192.168.5.2 192.168.5.6;
option routers 192.168.5.1;
option broadcast-address 192.168.5.7;
option domain-name-servers 10.151.79.43;
default-lease-time 3600;
max-lease-time 3600;
}

host PROBOLINGGO {
hardware ethernet 6e:a4:94:6f:fc:48;
fixed-address 192.168.5.2;
}

host MADIUN {
hardware ethernet 62:72:b6:1e:46:53;
fixed-address 192.168.5.3;
}

```

```
PADA UML MADIUN

hwaddress ether 62:72:b6:1e:46:53

```

```
PADA UML PROBOLINGGO

hwaddress ether 6e:a4:94:6f:fc:48;

```

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
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level info
iptables -A LOGGING -j DROP

```

### 4. Akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin sampai Jumat.

```
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 17:01 --timestop 06:59 -j REJECT

```

### 5. Akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya.

```
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:01 --timestop 16:59 -j REJECT
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 17:00 --timestop 07:00 -j ACCEPT

```

### 6. SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80.

```
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.43 --dport 80 -m statistic --mode random --probability 0.5 -j DNAT --to 192.168.5.3:80
iptables -t nat -A PREROUTING -p tcp -d 10.151.79.43 --dport 80 -j DNAT --to-destination 192.168.5.2:80
iptables -t nat -A POSTROUTING -p tcp -d 192.168.5.3 --dport 80 -j SNAT --to-source 10.151.79.43:80

iptables -t nat -A POSTROUTING -p tcp -d 192.168.5.2 --dport 80 -j SNAT --to-source 10.151.79.43:80

```

### 7. semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.

```
iptables -N LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
(SURABAYA)

iptables -N LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
(MALANG)

iptables -N LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
iptables -A LOGGING -j DROP
(MOJOKERTO)

```
