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

