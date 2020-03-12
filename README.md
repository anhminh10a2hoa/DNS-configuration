# DNS Configuration

### [How a DNS Server works](https://www.youtube.com/watch?v=mpQZVYPuDGU)

- Step 1: Check your hostname by using command:

```
hostnamectl
```

And you have your hostname

```
Static hostname: server.anhminh.com
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
type master;
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

Now open `forward.anhminh.com` by using command `sudo nano /etc/bind/forward.anhminh.com` and you need to config like that

- Note:

  - You need to have another machine as client then get its ip address
  - Change "server.anhminh.com" to your hostname
  - After hostname we have a dot (.) that is delimeter
  - Change 192.168.1.112 to your server ipaddress
  - Change 192.168.1.114 to your another machine
  - If you are confusing about what is NS, what is PTR and A you can follow this [link](http://dns-record-viewer.online-domain-tools.com/)

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     server.anhminh.com. server.anhminh.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      server.anhminh.com.
@       IN      A       192.168.1.112
server  IN      A       192.168.1.112
host    IN      A       192.168.1.112
client  IN      A       192.168.1.114
www     IN      A       192.168.1.114
```

After that copy file forward.anhminh.com to reverse.anhminh.com (you need to copy to your own file) by command:

```
sudo cp forward.anhminh.com reverse.anhminh.com
```

Then open `reverse.anhminh.com` by using command `sudo nano /etc/bind/reverse.anhminh.com` and config it like that:

- Note:

  - Change the second line like that `@ IN PTR anhminh.com.`
  - Add two last line and its first row we need to change the last ip address of server and client

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     server.anhminh.com. server.anhminh.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      server.anhminh.com.
@       IN      PTR     anhminh.com.
server  IN      A       192.168.1.112
host    IN      A       192.168.1.112
client  IN      A       192.168.1.114
www     IN      A       192.168.1.114
112     IN      PTR     server.anhminh.com.
114     IN      PTR     client.anhminh.com.
```

- Step 4: Check your configuration

  - Run the command: `sudo named-checkconf -z /etc/bind/named.conf` and you can see it is showing loaded serial it means server configuration is right. If you have any error then it will show and error and you have to go this file and make changes to
  - Then run the command: `sudo named-checkconf -z /etc/bind/named.conf.local` and two command to check everything is working:

  ```
  sudo named-checkzone forward /etc/bind/forward.anhminh.com
  ```

  ```
  sudo named-checkzone forward /etc/bind/reverse.anhminh.com
  ```

  - Now start the services bind9 by using command:

  ```
  sudo systemctl start bind9
  ```

  - And change the ownership and permissions of the file:

  ```
  sudo chown -R bind:bind /etc/bind
  ```

  ```
  sudo chmod -R 755 /etc/bind
  ```

  - Now restart the service bind9:

  ```
  sudo systemctl restart bind9
  ```

  - Check the status of bind9 `sudo systemctl status bind9` and make sure it's working like that:

  ```
  ● bind9.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/bind9.service; enabled; vendor preset: en
   Active: active (running) since Thu 2020-03-12 16:06:19 EET; 58min ago
     Docs: man:named(8)
  Main PID: 890 (named)
    Tasks: 4 (limit: 4915)
   CGroup: /system.slice/bind9.service
           └─890 /usr/sbin/named -f -u bind
  ```

  - Enable bind9 `sudo systemctl enable bind9`
  - If your firewall is active (you can check status of firewall : `sudo ufw status`), you need to allow bind9 by using command `sudo ufw allow bind9`
  - Go to `sudo nano /etc/network/interfaces` and add dns-search and domain name:

  ```
  # interfaces(5) file used by ifup(8) and ifdown(8)
  auto lo
  iface lo inet loopback
  auto enp0s3
  iface enp0s3 inet static
  address 192.168.1.112
  netmask 255.255.255.0
  dns-search anhminh.com
  dns-nameserver 192.168.1.112
  ```

  - And go to `sudo nano /etc/resolv.conf` of 2 machines (server and client) and fix it like that:

  ```
  nameserver 192.168.1.112
  search anhminh.com
  ```

  - After config this file, you need to run 2 command on both machines

  ```
  systemctl restart netwoking
  ```

  ```
  systemctl restart NetworkManager
  ```

  - Finally, test to ping server, host, client, www and run `nslookup server` to see the server, address and hostname.
