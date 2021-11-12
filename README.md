# Jarkom-Modul-3-E15-2021
Lapres Praktikum Jarkom Modul 3  
kelompok E15 : M. Iqbal Abdi 

## **Konten**
* [**Cara Pengerjaan**](#cara-pengerjaan)
* [**Hasil Akhir**](#hasil-akhir)
* [**Catatan**](#catatan)
* [**Kendala**](#kendala)


## Cara Pengerjaan
### Nomor 1
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server  
* Setup DNS
```
apt-get update
apt-get install bind9 -y
service bind9 start
```
* Setup DHCP Server
```
apt-get update
apt-get install isc-dhcp-server -y
```
* DHCP Server interface pada ```/etc/default/isc-dhcp-server```
```
INTERFACES="eth0"
service isc-dhcp-server start
```
* Setup Proxy Server
```
apt-get update
apt-get install squid -y
service squid start
```
* Setup Web Server
```
apt-get update
apt-get install php -y
apt-get install apache2 -y
apt-get install libapache2-mod-php7.0 -y
```
### Nomor 2
dan Foosha sebagai DHCP Relay
* Setup DHCP Realy
```
apt-get update
apt-get install isc-dhcp-relay -y
service isc-dhcp-relay start
```
* Pasang DHCP Relay ```/etc/default/isc-dhcp-relay```
```
SERVERS="10.7.2.4"
INTERFACES="eth1 eth2 eth3"
OPTIONS=
```
* Pasang IPV4 forward agar bisa menerima network lain ```/etc/sysctl.conf```
```
net.ipv4.ip_forward=1
```

### Nomor 3
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

* Konfigurasi IP Range DHCP untuk switch 1
```
subnet 10.7.1.0 netmask 255.255.255.0 {
    range 10.7.1.20 10.7.1.99;
    range 10.7.1.150 10.7.1.169;
    option routers 10.7.1.1;
    option broadcast-address 10.7.1.255;
}
```
### Nomor 4
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50
* Konfigurasi IP Range DHCP untuk switch 3
```
subnet 10.7.3.0 netmask 255.255.255.0 {
    range 10.7.3.30 10.7.3.50;
    option routers 10.7.3.1;
    option broadcast-address 10.7.3.255;
}
```
### Nomor 5
Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.
* Menambahkan konfigurasi forwarders pada ```/etc/bind/named.conf.options```
```
options {
        directory "/var/cache/bind";
        forwarders {
                192.168.122.1;
        };
        // dnssec-validation auto;
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
* Tambahkan konfigurasi pada ```/etc/dhcp/dhcpd.conf``` pada network 10.7.1.0 dan 10.7.3.0
```
option domain-name-servers 10.7.2.2;
```
### Nomor 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.
* Buka ```/etc/dhcp/dhcpd.conf``` dan tambahkan konfigurasi berikut pada network 10.7.1.0
```
    default-lease-time 360;
    max-lease-time 7200;
```
* Buka ```/etc/dhcp/dhcpd.conf``` dan tambahkan konfigurasi berikut pada network 10.7.3.0
```
    default-lease-time 720;
    max-lease-time 7200;
```
### Nomor 7
Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69
* Setup ```/etc/network/interfaces``` dan tambahkan konfigurasi berikut di Skypie
```
auto eth0
iface eth0 inet dhcp
hwaddress ether 52:69:bd:d1:b5:14
```
* Kemudian tambahakan konfigurasi pada ```/etc/dhcp/dhcpd.conf``` di Jipangu
```
host Skypie {
    hardware ethernet 52:69:bd:d1:b5:14;
    fixed-address 10.7.3.69;
}
```
### Nomor 8
Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000
* Buat konfigurasi hosts pada enieslobby ```/etc/bind/named.conf.local```
```
zone "jualbelikapal.A16.com" {
    type master;
    file "/etc/bind/jarkom/jualbelikapal.A16.com";
};
```
* Tambahkan konfigurasi DNS jualbelikapal.A16.com pada ```/etc/bind/jarkom/jualbelikapal.A16.com```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     jualbelikapal.A16.com. root.jualbelikapal.A16.com. (
                        2021110701      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@                       IN      NS      jualbelikapal.A16.com.
@                       IN      A       10.7.2.3        ; IP Water7
```
* Konfigurasi pada ```/etc/squid/squid.conf``` di Water7
```
http_port 5000
visible_hostname jualbelikapal.A16.com
```
* Pasang proxy pada loguetown
```
export http_proxy="http://jualbelikapal.A16.com:5000"
```
* Tambahkan hosts pada water7 di ```/etc/hosts``` untuk memastikan bahwa proxy menuju ke ip yang benar
```
10.7.2.3        jualbelikapal.A16.com
```
### Nomor 9
Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy
* Buat username dan password
```
htpasswd -c -b /etc/squid/passwd luffybelikapalA16 luffy_A16
htpasswd -b /etc/squid/passwd zorobelikapalA16 zoro_A16
```
* Tambahkan konfigurasi pada ```/etc/squid/squid.conf``` di Water7
```
http_port 5000
visible_hostname jualbelikapal.A16.com
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm AmpunBangJago
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS
```
### Nomor 10
Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)
* Buat konfigurasi untuk akses waktu pada ```/etc/squid/acl.conf``` di Jipangu
```
acl AVAILABLE_WORKING time MTWH 07:00-11:00
acl AVAILABLE_WORKING time TWHF 17:00-23:59
acl AVAILABLE_WORKING time A 00:00-03:00
```
* Tambahkan konfigurasi pada ```/etc/squid/squid.conf``` di Water7
```
include /etc/squid/acl.conf
http_port 5000
visible_hostname jualbelikapal.A16.com
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm Proxy
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow AVAILABLE_WORKING USERS
```
### Nomor 11
Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

* Tambahkan konfigurasi pada ```/etc/squid/squid.conf``` di Water7
```
acl BLKSite dstdomain google.com
http_access deny BLKSite
deny_info http://super.franky.A16.com BLKSite
```
### Nomor 12
Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps
* Buat konfigurasi hosts pada enieslobby ```/etc/bind/named.conf.local```
```
zone "super.franky.A16.com" {
    type master;
    file "/etc/bind/jarkom/super.franky.A16.com";
};
```
* Tambahkan konfigurasi DNS super.franky.A16.com pada ```/etc/bind/jarkom/super.franky.A16.com```
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     super.franky.A16.com. root.super.franky.A16.com. (
                        2021110701      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@                       IN      NS      super.franky.A16.com.
@                       IN      A       10.7.3.69        ; IP Skypie
```
* Setting web server skypie pada ```/etc/apache2/sites-available/000-default.conf```
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.A16.com
        ServerName super.franky.A16.com
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Tambahkan hosts pada water7 di ```/etc/hosts``` untuk memastikan bahwa proxy menuju ke ip yang benar
```
10.7.3.69       super.franky.A16.com
```
* Buat konfigurasi pada ```/etc/squid/acl-bandwidth.conf```
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
acl GETImage url_regex -i \.jpg$ \.png$
acl luffy proxy_auth luffybelikapalA16
acl zoro proxy_auth zorobelikapalA16
delay_pools 1
delay_class 1 1
delay_parameters 1 1250/1250
delay_access 1 allow GETImage luffy
```
### Nomor 13
Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya
* Tambahkan konfigurasi pada ```/etc/squid/acl-bandwidth.conf``` untuk memastikan user zoro tidak terbatasi limit bandwith
```
delay_access 1 deny all
```
## Hasil Akhir
### IP masing-masing client
* Loguetown
![image](https://user-images.githubusercontent.com/55046884/141448320-eea17571-4591-4670-ace4-1966003c35fb.png)

* Alabasta
![image](https://user-images.githubusercontent.com/55046884/141448374-45a400f8-d871-4cb8-8f55-b0d9b467c501.png)

* Tottoland
![image](https://user-images.githubusercontent.com/55046884/141448428-f0a6ab3a-07c1-4dc0-a2b5-d5dd70ebed75.png)

* Skypie
![image](https://user-images.githubusercontent.com/55046884/141448476-914496af-d772-45cc-acbe-0bb57265cfdc.png)

### Proxy
* ```lynx jualbelikapal.A16.com```  
![image](https://user-images.githubusercontent.com/55046884/141448639-c4213fee-8855-46a3-9d6e-3657122438f7.png)

### Google.com
* ```lynx google.com```  
![image](https://user-images.githubusercontent.com/55046884/141449665-431e4f07-6fd5-4e32-9d7f-05cbeabd91a2.png)

### Username & Password
* username : luffybelikapalA16 pass : luffy_A16  
![image](https://user-images.githubusercontent.com/55046884/141449554-20179391-e74b-41ac-bd98-4f12ff2a3467.png)

* username : luffybelikapalA16 pass : luffy_A20  
![image](https://user-images.githubusercontent.com/55046884/141449595-3cf635d1-52cf-4114-8d93-48c4606f1f67.png)

### Akses Waktu
* Setting waktu yang dibolehkan akses
```
date -s "20 Nov 2021 01:00:00"
```
* akses super.franky.A16.com  
![image](https://user-images.githubusercontent.com/55046884/141449041-6ad815bd-f724-46d6-a03c-7689d31d5622.png)
* Setting waktu yang tidak diperbolehkan akses
```
date -s "12 Nov 2021 12:00:00"
```
* akses super.franky.A16.com  
![image](https://user-images.githubusercontent.com/55046884/141449253-c740e5fa-6771-4911-a003-12bed4c72160.png)

### Bandwitdh
* User : luffybelikapalA16
* download eyeoffranky.jpg  
![image](https://user-images.githubusercontent.com/55046884/141450128-6127fc81-36dc-461b-9991-b5cc8a64b63f.png)
* download bukanfrankytapirandom.99689  
![image](https://user-images.githubusercontent.com/55046884/141450191-0ce3eb41-aa76-42f0-b7c6-d8508d2ae837.png)

* User : zorobelikapalA16
* download eyeoffranky.jpg  
![image](https://user-images.githubusercontent.com/55046884/141450496-4a379018-801b-4ef4-ae36-e891b7864ce0.png)

* download bukanfrankytapirandom.99689  
![image](https://user-images.githubusercontent.com/55046884/141450551-c2b1e0ca-35d2-4a1f-97e2-8e1a0ae0a7fc.png)

## Catatan
1. Jangan lupa restart gan
2. Kalau menggunakan VMWare harus di power off, jika di suspend akan bermasalah pada NAT nya
3. Kalau menggunakan VMWare jika date disetting ke hari sebelumnya dari jadwal default maka akan ke restart otomatis dalam rentang waktu 30s

## Kendala
1. Kendala lebih ke VMWarenya seperti catatan diatas
