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


### 2. Ostania sebagai DHCP Relay
Jawab:


### 3. Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu: Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155 
Jawab:


### 4. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85
Jawab:


### 5. Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.
Jawab:


### 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.
Jawab:


### 7. Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13
Jawab:


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
export http_proxy="http://192.176.2.3:8080"
```

Melakukan testing untuk validasi kode
- Testing hari kerja
  - Menset jam dan mengganti dengan jam kerja
  ```
  date --set "14 nov 2022 09:00:00"
  ```
  - Mencoba koneksi
  ```
  wget google.com
  ```
  - Menampilkan hasil koneksi ditolak
  ```Lampirkan gambar```

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
  ```Lampirkan gambar```


### 2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)
Jawab:

- Buat domain pada wise
```
mkdir /etc/bind/jarkom3
echo '
zone "loid-work.com" {
        type master;
        file "/etc/bind/jarkom3/loid-work.com";
};' > /etc/bind/named.conf.local

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                        2022110901      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       10.5.2.2
@     IN      AAAA   ::1
' > /etc/bind/jarkom3/loid-work.com

echo '
zone "franky-work.com" {
        type master;
        file "/etc/bind/jarkom3/franky-work.com";
};' >>/etc/bind/named.conf.local

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                        2022110901      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       10.5.2.2
@     IN      AAAA   ::1
' > /etc/bind/jarkom3/franky-work.com
service bind9 restart
```
- (Berlint) Membuat file /etc/squid/access.acl
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
    ```lampirkan gambar```
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
    ```lampirkan gambar```


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
  lynx http://example.com
  ```
  - Koneksi ditolak karena bukan HTTPS
  ```lampirkan gambar```
  
- Melakukan dengan HTTPS
  - Mencoba koneksi
  ```
  lynx https://example.com
  ```
  - Koneksi diterima karena HTTPS
  ```lampirkan gambar```


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
  ```lampirkan gambar```
  - Tanpa pembatasan speed
  ```lampirkan gambar```


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
