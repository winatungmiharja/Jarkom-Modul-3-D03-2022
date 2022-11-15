# DHCP

## 1

> Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

<img width="843" alt="Screenshot 2022-11-15 at 4 16 09 PM" src="https://user-images.githubusercontent.com/57696730/201879600-43790130-e85d-4cc9-a23c-c25b3ecddaa0.png">  



A. WISE sebagai DNS Server
```
apt-get update
apt-get install bind9 -y
```

B. Westalis sebagai DHCP Server
```
apt-get update
apt-get install isc-dhcp-server -y
```

kemudian kita tambahkan pada `/etc/dhcp/dhcpd.conf` dengan
```
nano /etc/dhcp/dhcpd.conf
```

karena `Westalis` terdapat pada subnet 2

```
subnet 192.186.2.0 netmask 255.255.255.0 {
        option routers 192.186.2.1;
}
```
<img width="481" alt="image" src="https://user-images.githubusercontent.com/64743796/200829574-9236510e-a6d8-4494-bf20-4b72fcb236aa.png">


kemudian kita tambahkan `eth0` interfaces pada `/etc/default/isc-dhcp-server` dengan
```
nano /etc/default/isc-dhcp-server
```

![image](https://user-images.githubusercontent.com/64743796/200804934-6b1bd25e-0773-4abc-8398-66b39cc680b2.png)

lakukan restart `isc-dhcp-server`

```
service isc-dhcp-server restart
```

<img width="494" alt="image" src="https://user-images.githubusercontent.com/64743796/200830106-426706c8-c0a8-4561-aea4-1e96d1fe3e1a.png">


C. Berlint sebagai Proxy Server
```
apt-get update
apt-get install squid -y
```

![image](https://user-images.githubusercontent.com/64743796/200801617-8c0d6c7f-68df-4c2c-9801-b688aa545cf8.png)

## 2

> Ostania sebagai DHCP Relay

```
apt-get update
apt-get install isc-dhcp-relay -y
```

disini kita akan melakukan forward DHCP server dari IP `192.186.2.4` karena Westalis memiliki IP address tersebut
dan sesuai topologi, kita juga akan melakukan forward pada `eth1 eth2 eth3`

ketika melakukan instalasi `isc-dhcp-relay`, kita akan ditanya untuk konfigurasi tersebut, atau kita bisa melakukan pengeditan pada `/etc/default/isc-dhcp-relay`

```
nano /etc/default/isc-dhcp-relay
```

<img width="480" alt="image" src="https://user-images.githubusercontent.com/64743796/200832328-bef1ecba-07b2-4ba9-8918-950ed63314e2.png">


lalu kita akan memulai relay dengan command

```
service isc-dhcp-relay start
```

## 3

> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server

client yang ada yaitu SSS, Garden, Eden, NewstonCastle, KemonoPark

oleh karena itu, kita harus mengubah konfigurasi IP statis menjadi konfigurasi IP DHCP dengan menggunakan script berikut

```
auto eth0
iface eth0 inet dhcp
```

lalu kita stop semua server yang bersangkutan melalui gns3, akan kita start lagi setelah konfigurasi DHCP diperbarui

<img width="958" alt="image" src="https://user-images.githubusercontent.com/64743796/200838266-19cfd10c-f3ca-4e45-b363-ca525325fccf.png">

> Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

maka konfigurasi DHCP kita menjadi

```
subnet 192.186.1.0 netmask 255.255.255.0 {
    range 192.186.1.50 192.186.1.88;
    range 192.186.1.120 192.186.1.155;
    ...
}
```

## 4

> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

maka konfigurasi DHCP kita menjadi

```
subnet 192.186.3.0 netmask 255.255.255.0 {
    range 192.186.3.10 192.186.3.30;
    range 192.186.3.60 192.186.3.85;
    ...
}
```

## 5

> Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.

WISE memiliki alamat `192.186.2.2`

maka konfigurasi DHCP kita menjadi

```
subnet 192.186.1.0 netmask 255.255.255.0 {
    ...
    option domain-name-servers 192.186.2.2;
    ...
}

subnet 192.186.3.0 netmask 255.255.255.0 {
    ...
    option domain-name-servers 192.186.2.2;
    ...
}
```

## 6

> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.

maka konfigurasi DHCP kita menjadi


```
subnet 192.186.1.0 netmask 255.255.255.0 {
    ...
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.186.3.0 netmask 255.255.255.0 {
    ...
    default-lease-time 600;
    max-lease-time 6900;
}
```

jadi konfigurasi DHCP pada `Westails` akan kita ubah

*Westails*
```
nano /etc/dhcp/dhcpd.conf
```

lalu kita ganti isinya menjadi 

```
subnet 192.186.1.0 netmask 255.255.255.0 {
    range 192.186.1.50 192.186.1.88;
    range 192.186.1.120 192.186.1.155;
    option routers 192.186.1.1;
    option broadcast-address 192.186.1.255;
    option domain-name-servers 192.186.2.2;
    default-lease-time 300;
    max-lease-time 6900;
}

subnet 192.186.3.0 netmask 255.255.255.0 {
    range 192.186.3.10 192.186.3.30;
    range 192.186.3.60 192.186.3.85;
    option routers 192.186.3.1;
    option broadcast-address 192.186.3.255;
    option domain-name-servers 192.186.2.2;
    default-lease-time 600;
    max-lease-time 6900;
}

subnet 192.186.2.0 netmask 255.255.255.0 {
    option routers 192.186.2.1;
}
```

setelah itu kita restart server DHCP

<img width="325" alt="image" src="https://user-images.githubusercontent.com/64743796/200844326-e22b364d-91ea-4fc3-a3d8-357ecf353d77.png">

setelah semuanya selesai, kita akan start semua node yang kita stop sebelumnya

dapat kita lihat, hasil IP address sesuai dengan yang kita konfigurasikan. yaitu sebagai berikut

- SSS

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200844736-c6d35cbf-655d-4713-b325-167b07b8739b.png">

- Garden

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200844817-bd76cbfe-f68e-4b29-ad78-3473358c2912.png">

- Eden

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200844879-42f1fac5-fa99-4db9-a1e8-96e26b730b54.png">

- NewstonCastle

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200844947-b63e6fde-511e-4f81-bad2-c24b0c0c6de7.png">

- KemonoPark

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200845009-83e33bfd-70af-4605-85ab-771bf098d778.png">

## 7

> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

pertama, kita mengambil hwaddress milik Eden dengan `ip a`

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200850674-85a175e0-7286-4545-8717-6ad1f4185ade.png">

dapat dilihat bahwa hwadress = `ee:cf:43:2c:69:72`

lalu kita menambahkan konfigurasi berikut pada `/etc/dhcp/dhcpd.conf` (Westails)

```
host Eden {
    hardware ethernet ee:cf:43:2c:69:72;
    fixed-address 192.186.3.13;
}
```

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200854201-982b0978-9cfd-4cf4-a4f0-57304bdccd3d.png">

lakukan restart  
```service isc-dhcp-server restart```  

<img width="517" alt="image" src="https://user-images.githubusercontent.com/64743796/200854356-3200fcd5-9b65-4bd2-83c0-7e6c2a69982b.png">

setelah itu, kita menambahkan konfigurasi node eden menjadi seperti berikut

```
auto eth0
iface eth0 inet dhcp
hwaddress ether ee:cf:43:2c:69:72
```

![image](https://user-images.githubusercontent.com/64743796/200851795-9189aef4-1277-4601-ae8a-ecf95f80ee59.png)

lalu kita stop dan start node Eden. Setelah itu, kita melakukan query `ip a` pada terminal Eden, maka dapat dilihat, bahwa IP Eden sekarang menjadi Fixed Ip sesuai yang kita setting sebelumnya

<img width="960" alt="image" src="https://user-images.githubusercontent.com/64743796/200854521-62401cce-5f60-41b9-b18d-340ab75ed5df.png">

# Proxy Server

## Setup

> Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:

kami menggunakan port 8080 karena pada soal tidak sidpesifikasikan

```
http_port 8080
visible_hostname Berlint
```

Lakukan restart squid  
```
service squid restart
```  

<img width="943" alt="image" src="https://user-images.githubusercontent.com/64743796/200872224-37fc5a76-deb9-4cab-8e35-720dc26d1a3d.png">


> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13 (7). SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data.

melakukan stop dan start pada node-node yang terhubung dengan Switch1 dan Switch3

kemudian kita menjalankan script berikut pada SSS, Garden, dan Eden

```
export http_proxy="http://192.186.2.3:8080"
env | grep -i proxy
```

<img width="461" alt="image" src="https://user-images.githubusercontent.com/64743796/200874475-a26b9bd4-5d5e-494f-ba8d-ef19798f2ca9.png">

<img width="462" alt="image" src="https://user-images.githubusercontent.com/64743796/200874360-0f43d263-4d82-40c5-95c6-b2e571cdd5f4.png">

<img width="461" alt="image" src="https://user-images.githubusercontent.com/64743796/200874432-06d2e83e-7fec-41b2-b30c-45a694b2b849.png">

- lalu disini kita mencoba melakukan lynx, namun masih error, jadi kita harus menambbahkan allow all http pada config squid
- jangan lupa untuk menginstall lynx terlebih dahulu
```
apt-get update
apt-get install lynx
```  

<img width="231" alt="image" src="https://user-images.githubusercontent.com/64743796/200885257-a3805b8d-8bf5-4ce0-a25c-70e9cb423a89.png">


<img width="649" alt="image" src="https://user-images.githubusercontent.com/64743796/200885172-5c2d7f18-87f7-4b7b-a8c0-e310d2c47fc0.png">


## 8

> Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

*/etc/squid/acl.conf*

```
acl AVAILABLE_WORKING_1 time MTWHF 00:00-07:59
acl AVAILABLE_WORKING_2 time MTWHF 17:01-24:00
acl AVAILABLE_WORKING_3 time SA 00:00-24:00
```

*/etc/squid/squid.conf*

```
include /etc/squid/acl.conf

http_port 8080
visible_hostname Berlint

http_access allow AVAILABLE_WORKING_1
http_access allow AVAILABLE_WORKING_2
http_access allow AVAILABLE_WORKING_3

http_access deny all
```

- Mari kita coba pada server SSS

pada `Wed Nov  9 16:40:45 UTC 2022`

<img width="457" alt="image" src="https://user-images.githubusercontent.com/64743796/200888828-f1ca9343-04be-49d2-85d6-f557a2ade39f.png">

<img width="548" alt="image" src="https://user-images.githubusercontent.com/64743796/200889119-72f16060-6a6e-4af2-a71c-6e4ea2002a7e.png">

Pada `Wed Nov  9 17:03:49 UTC 2022`

<img width="472" alt="image" src="https://user-images.githubusercontent.com/64743796/200894265-67ed2ee5-8ace-42db-b98e-c1d6968aae54.png">

<img width="573" alt="image" src="https://user-images.githubusercontent.com/64743796/200894485-9dd8e536-61f9-4315-8e3e-bcca6eefbc31.png">

## 9

> Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

kita menambahkan whitelist dengan menambah pada squid.conf

```
acl whitelist dstdomain .loid-work.com .franky-work.com
http_access allow whitelist
```

whitelist harus diletakkan sebelum http_port, alhasil jika tidak, akan error

- disini seharusnya termasuk Jam Kerja dimana Client tidak bisa mengakses, namun setelah ditambahkannya whitelist, maka bisa diakses meskpun jam kerja

<img width="465" alt="image" src="https://user-images.githubusercontent.com/64743796/200893579-914acb86-6515-4ade-9a16-bd04de871d99.png">

<img width="959" alt="image" src="https://user-images.githubusercontent.com/64743796/200893511-8ef78e09-4673-465f-9554-8630204a85d5.png">


## 10

> Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

- untuk ini, kita harus membuat acl untuk port https, yaitu 443 dan 563
```
acl SSL_ports port 443 563
```

- dan aturan ini kita AND dengan jam kerja yang sudah ada

```
include /etc/squid/acl.conf
acl CONNECT method CONNECT
acl SSL_ports port 443 563

acl whitelist dstdomain .loid-work.com .franky-work.com

http_access allow whitelist

http_port 8080
visible_hostname Berlint

http_access allow CONNECT AVAILABLE_WORKING_1 SSL_ports
http_access allow CONNECT AVAILABLE_WORKING_2 SSL_ports
http_access allow CONNECT AVAILABLE_WORKING_3 SSL_ports

http_access deny all
```

- lakukan restart pada squid
`service squid restart`

- Kita testing pada SSS
pada `Wed Nov  9 17:42:18 UTC 2022`
<img width="315" alt="image" src="https://user-images.githubusercontent.com/64743796/200902120-b918e95a-14ef-4a41-9022-188351cbda3b.png">

dan kita `lynx http://example.com`

<img width="587" alt="image" src="https://user-images.githubusercontent.com/64743796/200902237-313db6d1-9c25-4ec1-903b-27e6ba8b6b1c.png">

namun jika kita `lynx https://its.ac.id`

<img width="571" alt="image" src="https://user-images.githubusercontent.com/64743796/200902363-8af80b1e-7a16-4151-a2dc-49516187e3a0.png">

## 11
> Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)  

- Buat file bernama ```acl-bandwidth.conf``` di folder squid
```
nano /etc/squid/acl-bandwidth.conf
```  
- Masukkan script berikut
```
delay_pools 1
delay_class 1 1
delay_access 1 allow all
delay_parameters 1 16000/16000
``` 

- Ubah konfigurasi pada file squid.conf menjadi:  
```
include /etc/squid/acl-bandwidth.conf
http_port 8080
visible_hostname Berlint

http_access allow all
```  
 - Restart Squid
```
service squid restart
```  
 
 - Speed test :  
![Screenshot 2022-11-15 at 5 54 48 PM](https://user-images.githubusercontent.com/57696730/201902616-e62ddd53-3163-4feb-a4ba-22a6a39d111f.png)

## 12
> Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

Tambahkan
```
acl OPEN_TIME time MTWHF
```  
di ```/etc/squid/acl-bandwidth.conf```  
di edit saat masa revisi (Senin) jadi tidak bisa testing pada hari libur, sementara speedtest seperti diatas



