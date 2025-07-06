# Linux Administration and Networking Basics (Level 2) <br> Լինուքսի կառավարում և ցանցային հիմունքներ (փուլ 2)

## Networking basics

Network configuration is one of the most different part between various Linux distributions and even release versions of the same distros.
Here we discuss only RHEL/CentOS/AlmaLinux modern information.
Most useful way to find more information about particular network configuration detail, 
is searching the up-to-date Internet resources.

### Predictable Network Interface Names
For a long time Linux kernel was detecting network devices and assigning them 
interface names "**eth0**", "**eth1**", "**wlan0**", etc, 
which became traditional, but not so flexible.

Current versions of **SystemD** suite (and its part - **udev** device manager) 
automatically assign predictable, 
stable _network interface names_ for all local network interfaces. 

Recent distributions have special _man_ for it: `man systemd.net-naming-scheme`

The names have two-character prefixes based on the type of interface:<br />
`en` for Ethernet,<br />
`wl` for wireless LAN (WLAN),<br />

Most of the modern Linux distributions will have first network interface name 
`enp0s3` <br>
Second network interface most probably will be `enp0s8`

`enp0s3` meaning:<br />
**en** - ethernet<br />
**p0** - peripheral/prefix/bus number 0<br />
**s3** - slot/device number 3<br />

There are different naming schemes supported by **systemd**. Interface name examples:<br>
* `ens33`, `ens192`
* `eno16780032`
* `enx78e7d1ea46da`


### Configuration of network interfaces. <br />  

#### List all network interfaces
Several ways to list interfaces:<br />
* `ip l | grep mtu | sed 's/://g' | awk '{print $1,$2}' `
* `nmcli c`


#### `ip` command

The **ip** command can be used to check current settings:

```bash
ip a
```

> NOTE! There is old, deprecated `ifconfig` command, that should be avoided to use.


To configure static IP manually (without NetworkManager) we may use following command:
(but this will not remain after reboot)

```bash
ip addr add 10.11.12.13/24 dev enp0s3
```

Now you can check

```bash
ip a
```

To delete address, we can run

```bash
ip addr del 10.11.12.13/24 dev enp0s3
```




### NetworkManager
Most modern Linux versions come with **Network Manager**, 
a service that runs by default and has a graphical, 
text as well as command line interface.

You can check if **Network Manager** is enabled and running:<br>

```bash
systemctl is-enabled NetworkManager
```

```bash
systemctl is-active NetworkManager
```

#### nmtui

`nmtui` is  **Network Manager** text interface to create, edit and remove interfaces.
It can be used to configure Ethernet, WiFi and other connection types. 

Run it

```bash
nmtui
 ```

#### NetworkManager new config directory

NetworkManager stores new network profiles in keyfile format in the
`/etc/NetworkManager/system-connections/` directory.


```bash
ls -l /etc/NetworkManager/system-connections/
```


Previously, NetworkManager stored network profiles in `ifcfg` format in directory `/etc/sysconfig/network-scripts/`.
However, the `ifcfg` format is deprecated. 
By default, NetworkManager no longer creates new profiles in this format.

Look inside:

```bash
ls -l /etc/sysconfig/network-scripts/
```



### ARP Table
`arp` command allows to see current ARP table (`arp –an`)  
It is old command and the new equivalent is: <br>

`ip n[eigh]`


<pre>
State	      Meaning
REACHABLE	  ARP entry is valid and recently used
STALE	      Entry may be valid, needs checking
DELAY	      Waiting before probing
FAILED	      Address resolution failed
PERMANENT	  Static ARP, never times out
</pre>


Add `watch` to see changes in real time:

```bash
watch -n 1 ip neigh show dev enp0s3
```



### Important Network Files.

* `/etc/resolv.conf`	  - IP addresses of DNS servers

Depending on the way you configure network it may be possible to manually 
edit it or not.Unless configured to do otherwise, the network initialization
scripts. More info can be found in the file itself, 
under comment lines in the top.


* `/etc/hosts`	          - Local hostname-IP mappings

The use of this file **before** DNS resolving from `/etc/resolv.conf` is defined in `/etc/nsswitch.conf` on line: <br> 
`hosts:      files dns`

```bash
grep hosts: /etc/nsswitch.conf
```


* `/etc/hostname`	System hostname  (now should be configured by `hostnamectl`)


* `/etc/nsswitch.conf`	Name service lookup order


* `/etc/NetworkManager/system-connections/`	Connection profiles (keyfile format)

```bash
ls -l /etc/NetworkManager/system-connections/
```


### Routing. Routing tables.

Linux **routing table** is obtained by one of the following commands:
* `/sbin/ip r`
* `/bin/netstat -rn` 
* `/sbin/route -n`

Last two commands are old, deprecated should be avoided to use.



### Network Port Management

It is not enough just to deliver packets from one host to another. 
Information should be delivered to from one process/program on one host
to another process/program on another host.
That is why not one, bur **TWO** pairs of identifiers play important role:

|  _SourceAddress_ | _DestinationAddress_ 
| --- | --- |
|  **_SourcePort_** |  **_DestinationPort_**

Second pair presents **UDP/TCP port numbers**. 
It is 16 bit number (**0-65535**). 
Each UDP/TCP packet (on transport layer) contains source & destination port.  
This allows to uniquely identify a conversation between processes.

Many ‘well known’ ports published for client-server applications can be found in: 
`/etc/services` file. Some examples are below:

|  TCP Port | Service 
| --- | --- |
| 20,21 | FTP
| 22 | SSH (remote access)
| 25 | SMTP (mail)
| 80 | HTTP (web)
| 143| IMAP (mail)
| 443| HTTPS (HTTP over SSL/TLS)
| 465| SMTPS (SMTP over SSL/TLS
| 993 | IMAPS (IMAP over SSL/TLS) 


* `netstat -nlpt` - Current TCP ports and appropriate processes listening
* `netstat -nlpu` - Current UDP ports and appropriate processes listening
* `netstat -ant`  - Active TCP connections
* `netstat -anu`  - Active UDP connections
* `ss -nlpt` - Alternative command to `netstat`


#### lsof - list open files (ports)

Install `lsof`

```bash
yum -y install lsof
```

List all TCP ports open

```bash
lsof -i TCP
```

List all UDP ports open

```bash
lsof -i UDP
```

List only connections with some IP

```bash
lsof -i @[IP]
```

Check who listens port 22

```bash
lsof -i  :22
```


### Network Tools

* `ping`
* `traceroute`
* `mtr`

Install `traceroute` and `mtr` if absent.

```bash
yum -y install traceroute mtr
```

Examples:
* `ping -n -c3 8.8.8.8`
* `traceroute -n 9.9.9.9`
* `mtr 8.8.4.4`


#### Manual Network Config 

* Create additional network interface in VM

* Teacher will assign a number to each student - use your number below instead of `x`

* Configure static `10.10.x.1/24` network on it 
  * Use `nmtui` to create `enp0s8` interface
  * Assign static IP address `10.10.x.1/24`
    * `10.10.x.111/24` address will be teacher's IP in each student's subnet
    

This example is based on the environment like follows.
```bash
--------+---------------------+----------------------+------------
        | [enp0s8]            | [enp0s8]             | [enp0s8]
        | 10.10.0.1           | 10.10.1.1            | 10.10.2.1
        | 10.10.1.111         |                      |
        | 10.10.2.111         |                      |
        | 10.10.x.111         |                      |
+-------+--------+   +--------+---------+   +--------+---------+
|     lt00.am    |   |      lt01.am     |   |     lt02.am      |
|     Teacher    |   |    Student 1     |   |    Student 2     |
+----------------+   +------------------+   +------------------+

```

#### Disable ICMP Redirects

In TCP/IP (inside Linux Kernel) ICMP redirects are used to inform hosts of a "better" next-hop route.
Below we configure Linux not to accept it to have clear model of our routing.
(This is required only for our training, not for production, because here we put many subnets in same network).

```bash
echo "net.ipv4.conf.all.accept_redirects=0" >> /etc/sysctl.conf
echo "net.ipv4.conf.default.accept_redirects=0" >> /etc/sysctl.conf
sysctl -p  # Apply now + persist
```

Check if it works

* Teacher
  * `ping 10.10.0.1`
  * `mtr 10.10.0.1`
  
* Another student
  * `ping 10.10.x.1`
  * `mtr 10.10.x.1`


#### PACKET FORWARDING
It is important to understand that apart from routing itself 
PACKET FORWARDING is controlled by additional kernel setting `/proc/sys/net/ipv4/ip_forward`

* **1** means enabled 
* **0** means disabled

(NOTE! Default setting differs depending on Linux distribution version)

Check current setting: 

```bash
cat /proc/sys/net/ipv4/ip_forward
```

If we want to make this setting **permanent**, so they persist after reboot
we need to do:

```bash
echo  'net.ipv4.ip_forward = 1'  >>  >> /etc/sysctl.conf
sysctl -p # Apply now + persist
```

### Network Traffic Analysis and Monitoring Tools

#### tcpdump

**tcpdump** - basic tool to troubleshoot network.<br>

Install `tcpdump`

```bash
yum -y install tcpdump
```

Run several `ping` commands in background and check

```bash
ping 127.1.2.3 >/dev/null & 
ping 127.4.5.6 > /dev/null &  
ping 127.7.8.9 > /dev/null &
```

```bash
tcpdump -i lo host 127.1.2.3
```

```bash
tcpdump -i lo src 127.4.5.6
```

```bash
tcpdump -i lo dst 127.7.8.9
```

You should see difference:
**host** filter captures both (destination) & (source) traffic.
**src** / **dst** - only packets going one way. 
<br>
<br><br>
You can also capture whole subnet traffic:
* `tcpdump -i lo net 127.0.0.0/8`

Or only traffic to/from specific port.<br>
* `tcpdump -i lo dst port 22`

now on another terminal run:
* `ssh student@127.1.2.3`
<br>
<br>

We can show IP/Port in numbers.

On VM1 run:
* `tcpdump -i lo -nn -v dst port 22`

Options:<br>
**-nn** : 
_A single (n) will not resolve hostnames. A double (nn) will not resolve hostnames or ports. This is handy for not only viewing the IP / port numbers but also when capturing a large amount of data, as the name resolution will slow down the capture._<br>
**-v** : 
_Verbose, using (-v) or (-vv) increases the amount of detail shown in the output, often showing more protocol specific information._

Or see ICMP traffic only.<br>

On VM1 run:
* `tcpdump -i lo icmp`


#### iftop

**iftop** - interactive interface monitor tool.

Install `iftop`

```bash
yum -y install iftop
```

Keywords:
* **n** – toggles DNS hostnames resolution 
* **N** – toggles port names resolution
* **p** – toggles port numbers display
* **P** – pauses the screen
* **t** – toggles display modes
* **h** – toggles the help screen (more options here)

Simply run:

```bash
iftop
```

Show traffic for particular interface:
(you can specify interface with `-i`, but if not specified it tries to find the one that looks like an external interface)

```bash
iftop -i lo
```

```bash
ping 127.1.2.3 >/dev/null & 
ping 127.4.5.6 > /dev/null &  
ping 127.7.8.9 > /dev/null &
iftop -i lo
```

You can only filter some traffic

```bash
iftop -i lo -f 'dst host 127.7.8.9'
```

You can also disable resolving hostnames (-n), port numbers (-N) and show bandwidth bytes/sec instead of bits/sec:

```bash
iftop -i lo -n -N -B 
```

Show traffic for specific subnet

`iftop -i lo -F 127.1.2.0/24`

More info about `iftop` can be found in `man iftop`

### PRACTICE

You can try the same above examples with `enp0s8` interface IPs
