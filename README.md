## 1

> Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

![image](https://user-images.githubusercontent.com/64743796/200801000-4e71533c-afd8-4e77-ae2d-7bc163cc5506.png)


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
subnet 192.200.2.0 netmask 255.255.255.0 {
        option routers 192.200.2.1;
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




