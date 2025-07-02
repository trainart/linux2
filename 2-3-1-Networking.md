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

#### PRACTICE
1. Create second network interface on two VMs in VirtualBox. 
Assign each one to `VirtualBox Host-Only Ethernet Adapter`<br>
After starting VMs use `nmtui` to manually assign IP addresses 
to second interface on each VM:
* **10.10.10.10/24** - on VM1
* **10.10.10.11/24** - on VM2<br><br>
After that you should be able to `ping` from one VM to another by these IPs
and even connect another VMs port.

2. Manually assign another IP addresses to the same interfaces:

* `ip a a 172.16.11.5/24 dev enp0s8` - on VM1
* `ip a a 172.16.11.6/24 dev enp0s8` - on VM2<br><br>
After that you should be able to `ping` from one VM to another by these IPs 
and even connect another VMs port.


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

### Network Tools

* `ping`
* `traceroute`
* `mtr`

Examples:
* `ping -nc3 ya.ru`
* `traceroute -n goo.gl`
* `mtr yahoo.com`
* `mtr -nrc 1 fb.com`

Options:
* **-n**	_No DNS (Do not try to resolve the host names)_
* **-r** 	_put mtr into “wide report mode” (report after the number of cycles specified by the 
-c option (default 10) wide allows not to cut hostnames in the report)_
* **-c N**  _set the number of pings sent to determine both the machines on 
the network and the reliability of those machines. Each cycle lasts one second_
* **-u**	_Use UDP datagrams instead of ICMP ECHO (useful if “ICMP limiting” is found somewhere)_
