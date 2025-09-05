# Linux Administration and Networking Basics (Level 2) <br> Լինուքսի կառավարում և ցանցային հիմունքներ (փուլ 2)

## Managing Boot Process (SystemD)<br> Լինուքսի միացման գործընթացի կառավարում (SystemD)


### Linux Boot Process

Boot Process briefly:

**Power ON** → **Firmware** → **Bootloader** → **Kernel** → **INIT** → **System UP**


Detailed Boot Process:

<pre>
┌─────────────────────────────────────────────────────┐
│ 1. Power ON / Firmware → Checks, Load Bootloader    │
├─────────────────────────────────────────────────────┘
│
├─ 1.1 Hardware checks → POST (Power-On Self-Test)
├─ 1.2 Locate Boot Device, Load Bootloader (GRUB) → MBR(BIOS)/GPT(UEFI)
│
▼
┌─────────────────────────────────────────────────────┐
│ 2. Bootloader (GRUB) → Load Operating System (Kernel) │
├─────────────────────────────────────────────────────┘
│
├─ 2.1 Provide boot menu
├─ 2.2 Load Linux Kernel (PID 0) and pass control to it
├─ 2.3 Load Initramfs (Initial RAM File System) image into memory
│
▼
┌─────────────────────────────────────────────────────┐
│ 3. Linux Kernel (PID 0) → System Initialization     │
├─────────────────────────────────────────────────────┘
│
└─ 3.1 Mount Initramfs as temporary root filesystem (in RAM)
    │
    ├─ 3.1.1 Start temporary INIT process from Initramfs (PID 1)
    ├─ 3.1.2 Hardware checks (kernel level), Load drivers (storage, filesystem , etc.)
    ├─ 3.1.3 Detect root device
    ├─ 3.1.4 Pre-mount operations (decryption, LVM activation, RAID assembly)
    ├─ 3.1.5 Mount the real root filesystem
    ├─ 3.1.6 Switch (pivot) from initramfs (in RAM) to the real root filesystem.
┌───┘
├─ 3.2 Start INIT Process from the real root filesystem (SystemD/INIT - PID 1)
▼
┌─────────────────────────────────────────────────────┐
│ 4. INIT Process (SystemD/INIT - PID 1)              │
├─────────────────────────────────────────────────────┘
├─ 4.1 Load configuration and determine the boot goal 
│      (default.target/runlevel)
│
└─ 4.2 Achieve default.target/runlevel, by detecting dependencies and activating them
    │
    ├─ 4.2.1 Mount filesystems
    ├─ 4.2.2 Start services
    ├─ 4.2.3 Initialize user sessions
    │
    ▼
┌─────────────────────────────────────────────────────┐
│                System Up and Running                │
│        (User login prompt appears on console)       │
│ (Enabled network services listen appropriate ports) │
└─────────────────────────────────────────────────────┘
</pre>


Linux system initialization for a long time was handled by the _Unix-inspired SystemV_ **INIT** 
process, which ran scripts to start services in a defined and configurable order to reach a 
series of states, called **runlevels**. 

Modern Linux versions have special manual for it.

That manual also contains pretty **structural overview chart**

```bash
man bootup
```

Current most popular initialization system for Linux is **SystemD**. 
It is more flexible and modular. 
And it does not follow a strict sequence to get processes started.

>
> Recommended reading: 
> 1. "A pragmatic guide to systemd for Linux sysadmins"
> https://opensource.com/downloads/pragmatic-guide-systemd-linux
> 
> 2. "Rethinking PID 1"
> https://0pointer.de/blog/projects/systemd.html
>
> 

The main **object** that **SystemD** works with are known as **units**. 
Systemd doesn't just stop and start services.
It can mount filesystems, monitor your network sockets, etc. 
It has different types of **units** it operates. 

The most common units are:

* **Mount units** - these mount filesystems, these unit files end in `.mount`

* **Service units** - these are the services we've been starting and stopping, these unit files end in `.service`

* **Target units** - these **group together other units**, the files end in `.target`


**SystemD** uses **targets** instead of **runlevels** to define the state of the system. 

Goal of **runlevels**/**targets** is to process system initialization 
and bring the Linux system to specific state.


By default, there are two main final targets (including many other intermediate targets, used as checkpoints):

**multi-user.target** <-> **runlevel 3**

**graphical.target** <-> **runlevel 5**


**systemd** almost always starts the **default.target**. 
The **default.target** file is a symbolic link to the true target file.

To check the default target, which determines what services are started during boot, we can run:

```bash
systemctl get-default
```

For a Linux server, the default is more likely to be the **multi-user.target**

To change to text mode we can run (same as `init 3`):

```bash
systemctl isolate multi-user.target 
```


To change to graphical mode we can run (same as `init 5`):

```bash
systemctl isolate graphical.target
```

> Term "isolate" is strange, but never mind, just remember that's it

nently set default to text mode (to work after reboot) we can run:

```bash
systemctl set-default multi-user.target 
```

Now we can reboot:

```bash
reboot
```

> 
> Old alternative to `reboot` is `init 6` and it still works
> Old alternative to `poweroff` is `init 0` and it still works
 

You can also list the dependencies of a target
(to see which units run certain target):

```bash
systemctl list-dependencies multi-user.target
```
You may see different colors:
* **Green** - unit is active/running.
* **White**/**gray** - unit is inactive/not running.
* **Red** - unit failed to run.


`systemctl` is a powerful command-line tool for managing `systemd`-driven Linuxes. 
It's a **system** and **service** manager for Linux. More details are below.


Table below presents **SystemV runlevel** and **SystemD target** equivalents.

| SystemV Runlevel | SystemD equivalent | Description
| --- | --- | --- |
| 0 (HALT)|poweroff.target |Shuts down the system |
| 1 (SINGLE-USER MODE) | rescue.target | Mode for administrative and system rescue tasks. Only the root user can log in. |
| 2 (MULTI-USER MODE) | | All users can log in, but network interfaces aren’t configured and networks services are not exported. Display manager is not started. |
| 3 (MULTI-USER MODE WITH NETWORKING) | **multi-user.target** | Starts the system normally. Display manager is not started. |
| 5 (START THE SYSTEM NORMALLY WITH APPROPRIATE DISPLAY MANAGER (WITHGUI)) | **graphical.target** | Same as runlevel 3, but with a display manager.|
| 6 (REBOOT) | reboot.target | Reboots the system.|

Old way to understand where you are was:

```bash
runlevel
```

It shows current and previous runlevel.


To see systemd configuration run:

```bash
ls -l /etc/systemd/system
```

You can see that `default.target` entry is a symbolic link.


**Note:** Previous Linux versions, which were distributed with **SystemV init** ,
used init scripts located in the `/etc/rc.d/init.d/` directory. 

These **init** scripts were typically written in Bash, and allowed the system 
administrator to control the state of services and daemons in their system.

**SystemD** still can also run old _SystemV_ **init** scripts.

But **SystemD** driven systems have init scripts replaced with **service units**. 
**Service units** end with the **.service** file extension and serve a 
similar purpose as init scripts.

To view, `start`, `stop`, `restart`, `enable`, or `disable` system services, use the `systemctl` command as described below. 

Old `service` and `chkconfig` commands are still available in some versions 
and work as expected, but are only included for compatibility reasons 
and better to be avoided. 

**Note**
When working with system services, it is possible to omit this file `.service` unit
extension to reduce typing. When the `systemctl` utility encounters a unit name without a file extension, 
it automatically assumes it is a `.service` unit. 

For example: 

`systemctl status rsyslog.service`

is the same as:

`systemctl status rsyslog`


#### PRACTICE

##### Find out which initialization process (with PID **1**) is running in your Linux.
_HINT:_ 
* You can find an option of `ps` (with `ps --help list` or `man ps`) to show only process with PID 1.
* Or you can run command that shows all processes in **tree-like** manner and see the name of top process.


&nbsp;
&nbsp;

#### Service Management with `systemctl`

1. List all services on your system.

```bash
systemctl 
```

2. List the running services on your system.

```bash
systemctl | grep running
```

3. Check to see if the service (daemon) is running on your system. 

```bash
systemctl | grep cron
```

4. Restart, Stop, Start the service.

```bash
systemctl restart crond
```

```bash
systemctl stop crond
```

```bash
systemctl start crond
```

5. Disable/Enable the service start automatically at boot time.

```bash
systemctl is-enabled crond
```

```bash
systemctl disable crond
```

```bash
systemctl is-enabled crond
```

```bash
systemctl enable crond
```

```bash
systemctl is-enabled crond
```

6. Reboot & shutdown the system

You can reboot with you can either run

```bash
reboot
```

or

```bash
init 6
```

You can shutdown with either

```bash
poweroff
```

or

```bash
init 0
```


> INTERESTING NOTE: 
> 
> Look at output of `ls -l /usr/sbin/reboot`
> 
> Look at output of `ls -l /usr/sbin/poweroff`

#### Service Configuration files

To see configuration of some service we can use `cat` keyword:

```bash
systemctl cat sshd
```

```bash
systemctl cat crond
```

#### PRACTICE

Create you own startup test service, that will run for `multi-user.target`.

```bash
cat  > /etc/systemd/system/startuptest.service  << "LASTLINE"
[Unit]
Description=Startup Test Service

[Service]
ExecStart=/opt/startuptest.sh

[Install]
WantedBy=multi-user.target
LASTLINE

```

Check its contents 

```bash
systemctl cat startuptest
```

Create script mentioned in the service.

```bash
cat  > /opt/startuptest.sh  << "FINISH"
#!/bin/bash
echo "$(date) - Linux server $(hostname) started !" >> /var/log/startuptest.log
FINISH

chmod +x /opt/startuptest.sh

```

Now enable it to run at startup:

```bash
systemctl enable startuptest
```

Check:

```bash
systemctl is-enabled startuptest
```

Test run without restart
```bash
systemctl start startuptest
```

Check log file
```bash
cat /var/log/startuptest.log
```

Check in the list
```bash
systemctl list-dependencies multi-user.target | grep startuptest
```

Reboot

```bash
systemctl reboot
```

Login and check log file again.
You should see **one more line** from last startup.

```bash
cat /var/log/startuptest.log
```

&nbsp;
&nbsp;

### dmesg - dmesg (diagnostic message)

`dmesg` command gives log messages from kernel. It shows messages generated by the Linux kernel 
at boot time and during runtime (e.g., hardware detection, driver loading, errors, warnings)

```bash
dmesg
```

Make the message time human-readable
```bash
dmesg -T
```

View recent kernel messages

```bash
dmesg -T | tail -10 
```

Wait for new messages (works like `tail -f`)

```bash
dmesg -T -w
```

Clear the buffer (only clears buffer viewable via `dmesg`, not the logs in files)

```bash
dmesg -C
```

&nbsp;
&nbsp;


### Recovering root password

Below is important way to recover root password.
While booting the server you should interrupt normal boot and do the following steps.

1. Press `Esc` to prevent GRUB automatic system load. Ensure first line is selected
at the GRUB prompt and press `e`
2. Scroll down and locate kernel arguments line (starting with `linux` or similiar and
 having generally `rhgb quiet` keywords. Move your cursor ( HINT: move to end of the line with CTRL+E ) on `rhgb quiet` keywords and replace them with `init=/bin/bash`
3. Press `Ctrl X` to boot with custom parameters. Kernel will boot and run single **bash** process. 
To make changes you need to remount root partition, since it is now mounted
in _read-only_ mode. In order to mount root partition with read/write flag 
we need to remount it as follows: `mount -o remount,rw /`
4. Now **any** changes can be done, like root password change with `passwd`
5. **IMPORTANT !** This additional step needs to be taken on SELinux-enabled systems to relabel SELinux context. 
   * Check  `cat /etc/selinux/config  | grep ^SELINUX=`  (if you don't get any lines as result it means this option is commented).
      if this option is set to `permissive`or `disabled`, you can omit this step.
      But if it is set to `enforcing`, then this step is very important. If not done **you will not be able to login**. 
      What you need to do is run the command:

   ```bash
   touch /.autorelabel
   ```
    It will ensure that the SELinux context for entire system is relabeled after reboot. So you can boot normally.

6. Final important step is to `sync` the changes on disk. After that we can simply
power off the system (_no need to reboot, since there is no process which can do rebooting_)
  
_Same way can be used to do needed maintenance (eg. `fsck /dev/sda1` )_

#### How to Password Protect GRUB2 Boot Loader

Use `grub2-setpassword` to set a password for the `root` user (_it's not Linux `root`_)
```bash
grub2-setpassword
```
This creates a file `/boot/grub2/user.cfg` if not already present, 
which contains the hashed GRUB bootloader password. 
This utility only supports configurations where there is a single root user.

This protection prevents unauthorized users from:
* Editing boot parameters (pressing 'e')
* Accessing GRUB command line 
* Booting into recovery/single-user modes

To remove GRUB password-protect from boot menu, simply delete the file `/boot/grub2/user.cfg`

>
> More info at:
> https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/assembly_protecting-grub-with-a-password_managing-monitoring-and-updating-the-kernel
> 

#### GRUB2 Bootloader

GRUB2 (GRand Unified Bootloader 2) is the default bootloader on most modern Linux systems.

It controls:
* Which OS/kernel boots by default 
* Boot arguments (e.g., enabling rescue mode)
* Timeout before auto-booting

Key files:

* `/etc/default/grub` – Main settings (timeout, default OS, etc.)

* `/etc/grub.d/` – Scripts generating boot entries

* `/boot/grub2/grub.cfg` – Final config (automatically generated, **do not edit directly**!)



#### Simple Example: Changing Boot Timeout

Goal: Change the boot menu timeout from `5s` to `10s`.

Step 1: Edit /etc/default/grub


1. Open `/etc/default/grub`

```bash
nano /etc/default/grub`
```

2. Change `GRUB_TIMEOUT=5` to `GRUB_TIMEOUT=10` and save the file.

3. Change `GRUB_CMDLINE_LINUX` line and remove words **rhgb** and **quiet**
   1. rhgb - rhgb stands for Red Hat Graphical Boot, and it displays the little Fedora 
icon animation during the kernel initialization instead of showing boot-time messages. 
   2. quiet - quiet parameter, prevents displaying the startup messages that 
document the progress of the startup and any errors that occur. 
   
Without that options we will see all boot messages on the console screen.  

4. Save `/etc/default/grub` and exit

Step 2: Regenerate grub.cfg

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

Step 3: Verify Changes

We now search in newly generated config if our changes applied:

```bash
grep "timeout" /boot/grub2/grub.cfg
```

If you see `set timeout=2`, then it is ok.
We can reboot to see the effect

```bash
reboot
```

Important Notes:

* Never edit `/boot/grub2/grub.cfg` directly – Changes will be lost on update!

* Test changes before rebooting (use grep as shown above).

* For kernel arguments, modify `GRUB_CMDLINE_LINUX` in `/etc/default/grub`.




#### BONUS - Enable additional terminal configuration

After Linux system boots by default it enables about 6 terminal sessions on F1-F6 keys.

But we can add new terminal sessions.

For example, to immediately start new terminal on F12 run

```bash
systemctl start getty@tty12.service
```

Now try to login from `F12` terminal and after login type `w` or `tty`, to see your terminal name.


But the above will not remain after reboot.

We can enable that service

```bash
systemctl enable getty@tty12.service
```

Or we can do both at the same time.
Below will enable and immediately start new terminal on F10 run

```bash
systemctl enable --now getty@tty10.service
```


> Other way is to configure `systemd-logind` process, that manages user logins. 
> If we want more virtual terminals we can edit `/etc/systemd/logind.conf` and add or modify line:
> `NAutoVTs=12`, which will enable terminal to all F1-F12 keys.



