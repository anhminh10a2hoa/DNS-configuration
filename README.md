# DNS Configuration

### [How a DNS Server works](https://www.youtube.com/watch?v=mpQZVYPuDGU)

- Step 1: Check your hostname by using command:

```
hostnamectl
```

And you have your hostname

```
Static hostname: anhminh-VirtualBox
```

You can set up new hostname by using command:

```
hostnamectl set-hostname [name of new hostname]
```

- Step 2: Check your ip address by using command:

```
ifconfig
```

Now we can see the loopback is lo and main is enp0s3

```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.112  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::4711:4d36:46ba:4a50  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:21:6b:eb  txqueuelen 1000  (Ethernet)
        RX packets 13812  bytes 15109190 (15.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5408  bytes 817488 (817.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 544  bytes 57445 (57.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 544  bytes 57445 (57.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

And we have to open this file by using command:

```
sudo nano /etc/network/interfaces
```

then you need to have 4 lines in this file

```
auto enp0s3
iface enp0s3 inet static
address 192.168.1.112
netmask 255.255.255.0
```

Save and reboot your machine:

```
sudo systemctl restart networking
```

- Step 3: Install bind 9 and configure DNS Server

You need to run command `sudo apt-get update` to ensure that everyting is updated

Then install bind 9 by using command:

```
sudo apt-get install bind9 bind9utils
```

Now go to `sudo nano /etc/bind/named.conf.local` to add zones

```
zone "anhminh.com" IN {
tpye master;
file "/etc/bind/forward.anhminh.com";
} ;

zone "112.0.168.in-addr.arpa" IN {
type master;
file "/etc/bind/reverse.anhminh.com";
};

```

I explain about this file:

- You can create another name of zone but I use anhminh.com
- IN is for the internet
- Type of zone is master because we need to create master DNS Server
- File is the forward zone
- And the second zone, we need to reverse your ipaddress but do not need the last one:
  - For example: my ip address is 192.168.0.112, I will reverse is 112.0.168

Now go to the folder /etc/bind by using command:

```
cd /etc/bind
```

And copy file db.local to forward.anhminh.com (you need to copy to your own file) by command:

```
sudo cp db.local forward.anhminh.com
```

Now open `forward.anhminh.com` by using command sudo nano `/etc/bind/forward.anhminh.com`
