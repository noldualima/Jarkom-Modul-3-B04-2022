# Jarkom-Modul-3-B04-2022

## Praktikum Jaringan Komputer Modul 3 - DHCP dan Proxy Server
NRP | Nama
-----------|---------------------------
5025201025 | Dawamul Fikri Aqil
5025201048 | Afril Muzaqqi Arif
5025201165 | Gabriel Solomon Sitanggang
---------------------------------------

## Soal Shift Modul 3
Link : https://docs.google.com/document/d/1asm7lgnTJxr17DxsE_McdUimPsRjesi6ZrHRpmXPZ4s/edit

## Topologi

## Pembahasan Soal DHCP
### 1. Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server
Jawab:
![1](https://user-images.githubusercontent.com/72547769/202501594-f76803e0-cdba-48df-bce9-55224c3917b4.png)
- (DNS Server) Wise
network configuration
```auto eth0
iface eth0 inet static
           address 10.5.2.2
           netmask 255.255.255.0
           gateway 10.5.2.1
```
Bashrc
```
echo "nameserver 192.168.122.1" > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
```

- (DHCP Server) Westalis
network configuration
```
auto eth0
iface eth0 inet static
           address 10.5.2.4
           netmask 255.255.255.0
           gateway 10.5.2.1
```
Bashrc
```
apt-get update
apt-get install isc-dhcp-server -y
```
- (Proxy Server) Berlint
network configuration
```
auto eth0
iface eth0 inet static
           address 10.5.2.3
           netmask 255.255.255.0
           gateway 10.5.2.1
```
Bashrc
```
apt-get update
apt-get install squid -y
```

### 2. Ostania sebagai DHCP Relay
Jawab:
- (DHCP relay) Ostania
network configuration
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
       address 10.5.1.1
       netmask 255.255.255.0

auto eth2
iface eth2 inet static
       address 10.5.2.1
       netmask 255.255.255.0

auto eth3
iface eth3 inet static
      address 10.5.3.1
      netmask 255.255.255.0
```

Bashrc
```
apt-get update
apt-get install isc-dhcp-relay -y
```
Kemudian edit file `/etc/default/isc-dhcp-relay` dengan menambahkan SERVER = "10.5.2.4" dan INTERFACES = "eth1 eth2 eth3"
```
service isc-dhcp-relay restart
```

### 3 & 4. Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu: Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85
Jawab:

- (Client) SSS,Garden,KemonoPark,NewstonCastle
network configuration
```
auto eth0
iface eth0 inet dhcp
```

- (Client) Eden
network configuration
```
auto eth0
iface eth0 inet dhcp
    hwaddress ether ca:9f:1e:08:97:97
```
Westalis kemudian tambahkan `INTERFACES=\"eth0\"` pada `/etc/default/isc-dhcp-server` lalu tambahkan juga
```
subnet 10.5.1.0 netmask 255.255.255.0 {
    range  10.5.1.50 10.5.1.88;
    range  10.5.1.120 10.5.1.155;
    option routers 10.5.1.1;
    option broadcast-address 10.5.1.255;
    option domain-name-servers 10.5.2.2;
}

subnet 10.5.3.0 netmask 255.255.255.0 {
    range  10.5.3.10 10.5.3.30;
    option routers 10.5.3.1;
    option broadcast-address 10.5.3.255;
    option domain-name-servers 10.5.2.2;
}
```
pada `/etc/dhcp/dhcpd.conf` dan jangan lupa restart `service isc-dhcp-server restart` lalu cek dengan command `ip a` pada client

switch1

![image](https://user-images.githubusercontent.com/72547769/202515381-270dd85a-a28e-4c0f-8f89-13450cd8085d.png)

switch2

![image](https://user-images.githubusercontent.com/72547769/202515489-17220708-0cc7-43a5-8bee-f2f1e9a1a781.png)

### 5. Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.
Jawab:
pada setiap client akan mendapatkan DNS dari WISE sehingga diperlukan konfigurasi pada file /etc/dhcp/dhcpd.conf dengan isian sebagai berikut

```
option domain-name-servers 10.45.2.2;
```
melakukan setup pada WISE dengan melakukan editing pada file `/etc/bind/named.conf.options` dan menambahkan isian sebagai berikut :
```
options {
        directory \"/var/cache/bind\";

        forwarders {
                8.8.8.8;
                8.8.8.4;
        };

        // dnssec-validation auto;
        allow-query { any; };
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
untuk memastikan code berjalan sesuai dengan aturan maka dilakukann pengecekan dengan melakukan ping pada setiap client :
- SSS

![2](https://user-images.githubusercontent.com/72547769/202519788-1da00e42-2705-4f79-bd90-ab50b0ce7ec1.png)

- Garden

![3](https://user-images.githubusercontent.com/72547769/202519802-f7cc9fe0-bfaf-46c7-ad30-19e4bdd7970b.png)

- KemonoPark

![4](https://user-images.githubusercontent.com/72547769/202519808-840bf803-a5c1-4499-b32c-46db600d6e1a.png)

- NewstonCastle

![5](https://user-images.githubusercontent.com/72547769/202519818-21da52c8-5e15-4ff6-b840-ede1876f0d13.png)

- Eden

![6](https://user-images.githubusercontent.com/72547769/202519826-cdfa8ecc-12ac-41c0-a132-52dbbe14bd46.png)


### 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.
Jawab:
Pada node Wesalis lakukan perintah sebagai berikut :
- edit file `/etc/bind/named.conf.local` dengan cara
```
nano /etc/bind/named.conf.local
```
menyesuaikan isi file dengan
```
subnet 10.5.1.0 netmask 255.255.255.0 {
    range  10.5.1.50 10.5.1.88;
    range  10.5.1.120 10.5.1.155;
    option routers 10.5.1.1;
    option broadcast-address 10.5.1.255;
    option domain-name-servers 10.5.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 10.5.3.0 netmask 255.255.255.0 {
    range  10.5.3.10 10.5.3.30;
    option routers 10.5.3.1;
    option broadcast-address 10.5.3.255;
    option domain-name-servers 10.5.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}
```


### 7. Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13
Jawab:
Pada node Wesalis lakukan perintah sebagai berikut :

- edit file /etc/bind/named.conf.local dengan cara
```
nano /etc/bind/named.conf.local
```
manambhakan isi file dengan
```
host Eden {
    hardware ethernet 92:0c:bd:77:cd:85;
    fixed-address 10.5.3.13;
}
 ```
Pada node Eden melakukan konfigurasi network configuration sebagai berikut :
```
auto eth0
iface eth0 inet dhcp
    hwaddress ether ca:9f:1e:08:97:97
```

untuk melakukan validasi maka pada node Eden dilakukan pengecekan dengan menggunakan `ip a`, akan muncul hasil sebagai berikut
![7 1](https://user-images.githubusercontent.com/72547769/202521093-4536852f-5ba9-4f72-b08d-2d0c5656826f.png)


## Pembahasan Soal Proxy Server
### 1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)
Jawab:

Melakukan perintah pada node Berlint:
- Mengedit file /etc/squid/acl.conf
```
nano /etc/squid/acl.conf
```
- Menambahkan isi file
```
acl AVAILABLE_WORKING time MTWHF 17:01-23:59
acl AVAILABLE_WORKING time MTWHF 00:00-07:59
acl AVAILABLE_WORKING time AS 00:00-23:59
acl jam_kerja time MTWHF 08:00-17:00
acl weekend time AS 00:00-23:59
```
- Mengedit file /etc/squid/acl.conf
```
nano /etc/squid/squid.conf
```
- Menambahkan isi file
```
include etc/squid/acl.conf
http_port 8080
visible_hostname Berlint
http_access allow AVAILABLE_WORKING
http_access deny all
```
- Melakukan restart service squid
```
service squid restart
```
- Export http pada client
```
export http_proxy="http://10.5.2.3:8080"
```

Melakukan testing untuk validasi kode
- Testing hari kerja
  - Menset jam dan mengganti dengan jam kerja
  ```
  date --set "14 nov 2022 09:00:00"
  ```
  - Mencoba koneksi
  ```
  lynx google.com
  ```
  - Menampilkan hasil koneksi ditolak
  ![proxy1 1](https://user-images.githubusercontent.com/72547769/202528883-5d053cf2-8633-4f6a-b66d-cc7657e546cd.png)
  ![proxy1 2](https://user-images.githubusercontent.com/72547769/202529078-38036bf9-b608-405a-8eea-c4a23e4aeec1.png)

- Testing non hari kerja
  - Menset jam dan mengganti dengan bukan jam kerja
  ```
  date --set "6 nov 2022 18:00:00"
  ```
  - Mencoba koneksi
  ```
  lynx http://its.ac.id
  ```
  - Menampilkan hasil koneksi diterima
  ![proxy1 3](https://user-images.githubusercontent.com/72547769/202529442-5021fc4d-6dc0-4f64-b526-a85d576f04b7.png)


### 2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)
Jawab:

- Buat domain pada wise
```
mkdir /etc/bind/jarkom

echo '
zone "loid-work.com" {
        type master;
        file "/etc/bind/jarkom/loid-work.com";
};' > /etc/bind/named.conf.local

echo '
zone "franky-work.com" {
        type master;
        file "/etc/bind/jarkom/franky-work.com";
};' >>/etc/bind/named.conf.local

echo "
options {
        directory \"/var/cache/bind\";

        forwarders {
                8.8.8.8;
                8.8.8.4;
        };

        // dnssec-validation auto;
        allow-query { any; };
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
" > /etc/bind/named.conf.options
service bind9 restart

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       10.5.2.2
@       IN      AAAA    ::1 ' > /etc/bind/jarkom/franky-work.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       10.5.2.2
@       IN      AAAA    ::1 ' > /etc/bind/jarkom/loid-work.com

service bind9 restart
```
- (Berlint) Membuat file `/etc/squid/access.acl`
```
nano /etc/squid/access.acl
```
- Menambahkan isi file
```
loid-work.com
franky-work.com
```
- Mengedit file /etc/squid/squid.conf
```
nano /etc/squid/squid.conf
```
- Mengganti isi file
```
include /etc/squid/acl.conf
http_port 8080
acl WORKSITES dstdomain "/etc/squid/access.acl"
http_access allow WORKSITES
http_access allow jam_kerja
http_access deny all
visible_hostname Berlint
```
- Melakukan testing untuk validasi kode
  - Testing hari kerja
    - Menset jam dan mengganti dengan jam kerja
    ```
    date --set "14 nov 2022 09:00:00"
    ```
    - Mencoba koneksi
    ```
    lynx loid-work.com
    ```
    - Koneksi diterima namun menampilkan hasil berikut, karena site belum diavailable
    ![proxy2 1](https://user-images.githubusercontent.com/72547769/202531807-03202e30-b60e-41a8-82a1-a3428d1c7238.png)


  - Testing non hari kerja
    - Menset jam dan mengganti dengan bukan jam kerja
    ```
    date --set "6 nov 2022 18:00:00"
    ```
    - Mencoba koneksi
    ```
    lynx loid-work.com
    ```
    - Koneksi ditolak dan menampilkan hasil
    ![proxy1 1](https://user-images.githubusercontent.com/72547769/202531309-65cb35ef-2ec3-4a8e-8870-e2ebbb795575.png)



### 3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)
Jawab:

Berlint
- Mengedit file /etc/squid/acl.conf
```
nano /etc/squid/acl.conf
```
- Menambahkan isi file
```
.....
acl SSL_ports port 443
http_access deny !SSL_ports
.....
```

Melakukan testing untuk validasi kode
- Melakukan dengan HTTP
  - Mencoba koneksi
  ```
  lynx http://its.ac.id
  ```
  - Koneksi ditolak karena bukan HTTPS
  ![proxy1 1](https://user-images.githubusercontent.com/72547769/202531309-65cb35ef-2ec3-4a8e-8870-e2ebbb795575.png)
  
- Melakukan dengan HTTPS
  - Mencoba koneksi
  ```
  lynx https://its.ac.id
  ```
  - Koneksi diterima karena HTTPS
  ![proxy3 2](https://user-images.githubusercontent.com/72547769/202534853-0017c22c-5b60-4bfc-98b3-8a51446c57cd.png)



### 4. Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
Jawab:

- Membuat file /etc/squid/acl.conf
```
nano /etc/squid/acl-bandwidth.conf
```
- Menambahkan isi file
```
.....
delay_pools 1
delay_class 1 1
delay_access 1 allow all
delay_parameters 1 16000/16000
.....
```
-Mengedit file /etc/squid/acl.conf
```
nano /etc/squid/acl.conf
```
-Menambahkan isi file
```
.....
include /etc/squid/acl-bandwidth.conf
.....
```
- Melakukan test
  - Dengan pembatasan speed
  ![proxy4 1](https://user-images.githubusercontent.com/72547769/202535907-238f5476-e1a7-4c75-8da5-c86a14c0b60d.png)

  - Tanpa pembatasan speed
  ![proxy4 2](https://user-images.githubusercontent.com/72547769/202535913-0be52600-d8e5-4522-a87f-7a41de172d4b.png)



### 5. Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur
Jawab:

- Mengedit file /etc/squid/acl.conf
```
nano /etc/squid/acl.conf
```
- Menambahkan isi file
```
.....
http_access allow !WORKSITES !AVAILABLE_WORKING
http_access allow WORKSITES AVAILABLE_WORKING
.....
```



## Kendala
