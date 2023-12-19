# Jarkom-Modul-5-F06-2023
 
Kelompok F06:
- Arkana Bilal Imani / 5025211034
- 🤓

## Resource

- [Sheet Perhitungan](https://docs.google.com/spreadsheets/d/14c60BSwHFZ9jMbxMC8V4LwsBkQ76Yv2QWYykRAxKQ2Y/edit?usp=sharing)

## Daftar Isi

- [Daftar Isi](#daftar-isi)
- [Prerequisite](#prerequisite)
- [Pembagian Subnet](#pembagian-subnet)
- [VLSM Subnetting](#vlsm-subnetting)
- [VLSM Routing](#vlsm-routing)
- [DHCP](#dhcp)
- [DNS](#dns)
- [Nomor 1](#nomor-1)

### Prerequisite

- Buat topologi sesuai berikut:
![alt](images/topo.png)
- Keterangan:	
  - Richter adalah DNS Server
  - Revolte adalah DHCP Server
  - Sein dan Stark adalah Web Server
  - Jumlah Host pada SchwerMountain adalah 64
  - Jumlah Host pada LaubHills adalah 255
  - Jumlah Host pada TurkRegion adalah 1022
  - Jumlah Host pada GrobeForest adalah 512

###  Pembagian Subnet
- [Daftar Isi](#daftar-isi)

Topologi soal dibagi menjadi berikut:
![alt](images/pembagian.png)

Dengan pembagian rute sebagai berikut:
![alt](images/rute.png)

### VLSM Subnetting
- [Daftar Isi](#daftar-isi)

Rute dan subnet tersebut langsung dapat di-assign-kan IP dengan metode VLSM menggunakan tree sebagai berikut:

![alt](images/tree.png)

Lalu diberikan broadcast IP sebagai berikut:

![alt](images/nid.png)

### VLSM Routing
- [Daftar Isi](#daftar-isi)

Dengan subnetting tersebut, langsung saja dilakukan routing pada setiap subnet di GNS3. Menggunakan command `iptables add -net (IP NID) netmask (Netmask IP) gw (Gateway IP)`.

Contoh semua command iptables di router Aura:
```bash
route add -net 192.224.4.0 netmask 255.255.252.0 gw 192.224.0.22 #A1
route add -net 192.224.8.0 netmask 255.255.248.0 gw 192.224.0.22 #A2

route add -net 192.224.0.0 netmask 255.255.255.252 gw 192.224.0.18 #A3

route add -net 192.224.2.0 netmask 255.255.254.0 gw 192.224.0.18 #A4
route add -net 192.224.0.12 netmask 255.255.255.252 gw 192.224.0.18 #A8
route add -net 192.224.0.128 netmask 255.255.255.128 gw 192.224.0.18 #A5
route add -net 192.224.0.4 netmask 255.255.255.252 gw 192.224.0.18 #A6
route add -net 192.224.0.8 netmask 255.255.255.252 gw 192.224.0.18 #A7
```

Tentu saja network config di setiap node juga dicocokkan, sesuai dengan routing yang sudah dilakukan.

Berikut adalah contoh network configuration Aura:
![alt](images/netconfig.png)

Pada Aura, eth0 terhubung ke NAT sehingga digunakan DHCP. Lalu eth1 terhubung dengan router Heiter pada subnet A10. Sesuai dengan routing, subnet A10 diberi NID `192.224.0.20` dan karena Aura mengarah ke pusat, maka IP eth1 pada Aura dibuat dengan NID + 1 = `192.224.0.21`. Begitu juga dengan eth2 yang terhubung dengan A9 yang memiliki NID `192.224.0.16` sehingga IP yang diberikan ke eth2 adalah NID + 1 = `192.224.0.17`.

### DHCP
- [Daftar Isi](#daftar-isi)

Server DHCP ditempatkan di Revolte. Setelah menjalankan instalasi server DHCP dengan command `apt-get install isc-dhcp-server -y`, jalankan skrip dibawah untuk melakukan konfigurasi DHCP.
```bash
echo '
subnet 192.224.0.8 netmask 255.255.255.252{
} #A7

subnet 192.224.0.4 netmask 255.255.255.252{
} #A6

subnet 192.224.0.12 netmask 255.255.255.252{
} #A8

subnet 192.224.0.16 netmask 255.255.255.252{
} #A9

subnet 192.224.0.20 netmask 255.255.255.252{
} #A10

subnet 192.224.0.0 netmask 255.255.255.252{
} #A3

subnet 192.224.8.0 netmask 255.255.248.0 {
    range 192.224.8.2 192.224.12.255;
    option routers 192.224.8.1;
    option broadcast-address 192.224.15.255;
    option domain-name-servers 192.224.0.6;
    default-lease-time 180;
    max-lease-time 5760;
} #A2

subnet 192.224.4.0 netmask 255.255.252.0 {
    range 192.224.4.3 192.224.6.255;
    option routers 192.224.4.1;
    option broadcast-address 192.224.7.255;
    option domain-name-servers 192.224.0.6;
    default-lease-time 180;
    max-lease-time 5760;
} #A1

subnet 192.224.0.128 netmask 255.255.255.128 {
    range 192.224.0.131 192.224.0.254;
    option routers 192.224.0.130;
    option broadcast-address 192.224.0.255;
    option domain-name-servers 192.224.0.6;
    default-lease-time 180;
    max-lease-time 5760;
} #A5

subnet 192.224.2.0 netmask 255.255.254.0 {
    range 192.224.2.2 192.224.3.254;
    option routers 192.224.2.1;
    option broadcast-address 192.224.3.255;
    option domain-name-servers 192.224.0.6;
    default-lease-time 780;
    max-lease-time 5760;
} #A4

' > /etc/dhcp/dhcpd.conf

echo '
INTERFACESv4="eth0"
' > /etc/default/isc-dhcp-server

service isc-dhcp-server restart
```

IP DNS server yang diberikan sesuai dengan topologi, yaitu Richter dengan IP `192.224.0.6`.

Perhitungan range pada setiap subnet menggunakan jumlah available IP yang didapat dari netmask setiap subnet.

Setelah server DHCP sudah berjalan, bisa diinstall dan dikonfigurasikan DHCP relay pada setiap router di topologi ini.

Contoh pada Fern, instalasi menggunakan command ``, lalu konfigurasikan sesuai dengan skrip dibawah saat diprompt saat instalasi.

```conf
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="192.224.0.10"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth0 eth1 eth2"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

Tentu saja server DHCP yang digunakan adalah Revolte dengan IP `192.224.0.10`. Lalu interfaces yang digunakan juga disesuaikan pada setiap router, dimana mayoritas router menggunakan eth0, eth1, dan eth2.

Apabila relay sudah berjalan, maka setiap client (SchwerMountains, LaubHills, TurkRegion, GrobeForest) seharusnya sudah bisa mendapatkan IP dari DHCP dengan contoh sebagai berikut:

![alt](images/dhcp.png)

### DNS
- [Daftar Isi](#daftar-isi)

Server DNS ditempatkan di Richter. Setelah menjalankan instalasi DNS dengan command `apt-get install bind9 -y`, dapat dijalankan skrip dibawah ini untuk konfigurasi DNS.
```bash
mkdir /etc/bind/jarkom

echo '
options {
        directory "/var/cache/bind";

         forwarders {
                192.168.122.1;
         };

        //dnssec-validation auto;
        allow-query{any;};
        listen-on-v6 { any; };
};
' > /etc/bind/named.conf.options

service bind9 restart
```

Dibuatkan forwarder yang mengarah ke `192.168.122.1` supaya client dapat terhubung ke internet.

### Nomor 1
- [Daftar Isi](#daftar-isi)

Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Aura menggunakan iptables, tetapi tidak ingin menggunakan MASQUERADE.
