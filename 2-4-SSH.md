# Linux Administration and Networking Basics (Level 2) <br> Լինուքսի կառավարում և ցանցային հիմունքներ (փուլ 2)

## SSH

**SSH** stands for ‘Secure SHell’. 
Today it's Client/Server-based de-facto standard for **network remote terminal access**.
SSH Clients are included in almost all Linux versions out of the box (mostly as part of **OpenSSH** package)
(recent versions of Windows also include SSH Client).

> Read more on "How does the SSH protocol work" here: _https://www.ssh.com/academy/ssh/protocol#how-does-the-ssh-protocol-work_
> 
> It's important to understand that _asymmetric_ keys are **not used for traffic encryption/decryption**. 
> Key-based access is used to generate _symmetric_ one-time keys used for current session.

SSH allows:
* Remote **terminal access** `ssh` (either as _session_ or as _single command execution_)
* Remote **file transfer** `sftp`/`scp`


Examples:<br>

* `ssh student@10.1.10.1`
* `ssh -l student 10.1.10.1 date`
* `ssh student@10.1.10.1 ' echo "Hello Linux" > /tmp/hello' `
* `scp -r student@10.1.10.1:/etc/sysconfig /tmp`
* `scp  student@10.1.10.1:/bin/ls  student@10.1.10.2:/tmp`
* `sftp 10.1.10.1`


### SSH Windows Clients

Some free SSH/SFTP/SCP clients for Windows are:
* **Xshell**/XFTP (https://www.netsarang.com/ru/free-for-home-school/) - Great solution. Now is free for non-commercial use. <br><br>
* **PuTTY** (https://www.putty.org/) <br><br>
* **Tabby** (https://tabby.sh/)<br><br>
* **MobaXterm** (https://mobaxterm.mobatek.net/) - Enhanced terminal for Windows with X11 server, tabbed SSH client, network tools and much more<br><br>
* **OpenSSH Client** has been implemented as a Windows feature (in Windows 10/11 _from ver.1803_). If not found can be added via `Optional features` (start typing in search `optional`…)<br><br>
* **WinSCP** (https://winscp.net/eng/downloads.php) <br><br>
(plus other commercial ones, like: SecureCRT,...)


### Using **ssh** without password (authenticating via public/private keypairs instead of password)


> * SSH Client puts it's public key in server's directory in `~/.ssh/authorized_keys`. When that client tries to connect, server generates a random message and asks to encrypt it with its private key.
> * SSH Server gets encrypted message from SSH Client and tries to decrypt it with Client's public key. 
> * If OK then it trusts that Client. SSH Server generates symmetric key and securely sends it to the Client.
> * This key can also be regenerated during a session upon mutual agreement.

> Remember again that asymmetric keys are **not used for traffic encryption/decryption**. 
> Key-based access is used to generate symmetric one-time key used for current session

#### Generate SSH Public/Private keypair

Use `ssh-keygen` on your local system to generate public and private keys 
in SSH config directory: `~/.ssh`

Fully non-interactive way to create SSH Public/Private keypair:
(if you run just `ssh-keygen` you will need to interactively answer some questions)

```bash
ssh-keygen -t rsa -f ~/.ssh/id_rsa -N "" -C "student@$(hostname)"
```

> _As a result we will get two new keys:_ 
> * `~/.ssh/id_rsa`
> * `~/.ssh/id_rsa.pub`<br><br>
>_KEEP IN SECRET `~/.ssh/id_rsa`_<br> 
>_it is your **private key**_<br>
>_`~/.ssh/id_rsa.pub` is your **public key**_ <br>
>_It could be copied to the location you want access without password._

```bash
ll ~/.ssh/id_rsa*
```


##### Alternative algorithm `ed25519` 

On modern systems (OpenSSH 6.5+) we can use better `ed25519` algorithm.
(`ed25519` is modern alternative algorithm that's more secure and performant than older algorithms)

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N "" -C "student@$(hostname)"
```

Check
```bash
ll ~/.ssh/id_ed25519*
```

> NOTE! If you generate 2 keypair by default `id_rsa` will be used


#### Copy Public key to the remote system.

Now securely copy your public key the `~/.ssh/id_rsa.pub` file to the `~/.ssh` directory on the remote system.
(in case of `ed25519` you should copy `~/.ssh/id_ed25519.pub`)

You can copy it in various ways like:

1. Using `ssh-copy-id`:<br>
* `ssh-copy-id user@host`

2. Or manually, like for the older SSH version cases, where `ssh-copy-id` is not present:
`cat ~/.ssh/id_rsa.pub | ssh user@host 'mkdir ~/.ssh ; chmod 700 ~/.ssh; cat >> ~/.ssh/authorized_keys ; chmod 600 ~/.ssh/authorized_keys'`

Now you should be able to connect to the remote system via ssh without being prompted for a password.
This means you can run `ssh` to either get a remote shell or just run a single command remotely. 
Also, you can use `scp` commands as well - all that without password.

```bash
ssh  10.1.10.1
ssh  student@10.1.10.1 /bin/date
scp  student@10.1.10.1:/bin/ls  ./
```

#### PRACTICE

1. Create keypair on one system and copy to another. 
Then try connecting without password.

2. Connect from Windows to Linux using Windows client.

Enter Windows `PowerShell`. 
Generate keypair and transfer to Linux to connect without password.
* `ssh-keygen -t rsa`
* `cat ~/.ssh/id_rsa.pub | ssh user@host 'mkdir ~/.ssh ; chmod 700 ~/.ssh; cat >> ~/.ssh/authorized_keys ; chmod 600 ~/.ssh/authorized_keys'`

Try connecting:

`ssh user@host`


### SSH HardeningTips

Changes below are to be done in the SSH Server configuration file:

`/etc/ssh/sshd_config`

#### Changing the listening port
Some say that obscurity is not security but that's not true. 
Any measure that makes attacking your system harder can be valid. 
One of the effective measures is **changing the SSH port**. 

> NOTE: If firewall (`firewalld`/`ufw`) is active, then its settings 
> need to be adjusted for the new SSH port too.


```bash
#Port 22
Port 5022
```

#### Turning IPv6 off
```bash
AddressFamily inet  #  inet - means IPv4 only AddressFamily
```

#### Listen to particular IP
```bash
#ListenAddress 0.0.0.0
#ListenAddress ::
ListenAddress 10.1.10.1
```

Restrict long login session and turn off root login
```bash
#LoginGraceTime 2m
LoginGraceTime 1m
#PermitRootLogin yes
PermitRootLogin no
```

Manage Public key  &  Password Authentication
```bash
#PubkeyAuthentication yes
PubkeyAuthentication no
PasswordAuthentication yes
```

You can require all logins use keys with 
```bash
PasswordAuthentication no
```

you can specify that only the root user must use a key with
```bash
PermitRootLogin without-password
```

Save the file and restart the SSH daemon:<br>
`systemctl restart sshd`

Check the port to ensure IPv6 is off now:<br>
`lsof -i  | grep ssh`

Now SSH is listening new port. 
You can try to connect:<br>
`ssh -p 5022 user@host`

Changing the port mostly brings the number of SSH brute-force attacks to zero.

### Limiting SSH access per user and per IP-address

OpenSSH provides the possibility to restrict access for specific user and/or specific IP addresses. 

SSH allows you to restrict users and groups by host or IP address. 
There are four different directives you can use in your `/etc/ssh/sshd_config` file (they are evaluated in this order):

```bash
DenyUsers
AllowUsers
DenyGroups
AllowGroups
```

The format for all of them will be the same - 
a space-separated list of users or group names, 
with optional host names.

#### PRACTICE

1. Try to limit any SSH users to access only from specific IP range (e.g. network 9.9.9.0/24) 
at the bottom of the file `/etc/ssh/sshd_config` add:
```bash
AllowUsers student@127.0.0.1
```

Save the file, restart SSH daemon, Now only users coming from network
10.10.10.0/24 should be able to login by ssh, any other source IP will always get “Wrong username or password”
Try connecting from localhost:

```bash
ssh -p 5502 stident@127.0.0.1
```
You should be able to connect, but below variants should not work:

```bash
ssh -p 5502 student@10.1.10.1
```

```bash
ssh -p 5502 root@127.0.0.1
```


2. Add per-user access config:

```bash
AllowUsers  student@127.0.0.1  *@10.1.10.*  specialuser
```

You should now be able to connect, with below variants:

```bash
ssh -p 5502 student@10.1.10.1
```

```bash
ssh -p 5502 root@127.0.0.1
```


### Rsync

Rsync provides fast incremental file transfer. It synchronizes files and 
directories from one location to another. 
An important feature of rsync not found in most similar programs/protocols 
is that the mirroring takes place with only one transmission in each direction. 
rsync can copy or display directory contents and copy files, optionally using 
compression and recursion. 

```bash
rsync [OPTIONS] SOURCE DESTINATION
```

The most important `rsync` options are: 

```bash
-a, --archive		archive mode; same as -rlptgoD
  -r, --recursive          	recurse into directories
  -l, --links                 	copy symlinks as symlinks
  -p, --perms              	preserve permissions
  -t, --times                 	preserve modification times
  -g, --group               	preserve group
  -o, --owner              	 preserve owner (super-user only)
  -D, --devices --specials      preserve special files

--delete		delete files that don not exist on sender
-v, --verbose		increase verbosity
-u, --update		skip files that are newer on the receiver
-n, --dry-run           perform a trial run with no changes made
-z, --compress	compress file data during the transfer
```
#### PRACTICE Simple rsync Examples

1. Create some initial directories & files
```bash
mkdir /tmp/rs1 /tmp/rs2 ; \
fallocate -l 10K /tmp/rs1/f1 ; \
fallocate -l 10K /tmp/rs1/f2 ; \
fallocate -l 101M /tmp/rs1/f3 ; \
fallocate -l 122M /tmp/rs1/f4 
```
2. Run `rsync`
```bash
rsync -av --delete /tmp/rs1 127.0.0.1:/tmp/rs2
```

Now you have a full backup copy of `/tmp/rs1` directory 
on "remote" server's `127.0.0.1:/tmp/rs2` directory (inside it !)

3. Add a file and run `rsync` again
```bash
fallocate -l 5M /tmp/rs1/a5 ; \
rsync -av --delete /tmp/rs1 127.0.0.1:/tmp/rs2
```
You should see new file `/tmp/rs1/a5` added to "remote" copy.

4. Remove some file and run `rsync` again
```bash
rm /tmp/rs1/f3 ; \
rsync -av --delete /tmp/rs1 127.0.0.1:/tmp/rs2
```
You should see `f3` file removed from "remote" copy too.<br>
> NOTE: this is because of `--delete` option. 
> if you run `rsync` without this option it will only add files

5. Create new files in both dirs, but in second one first
   (we imitate we have newer file on destination)
```bash
fallocate -l 1M /tmp/rs1/f77 ; \
touch /tmp/rs2/rs1/f77 
```

6. Run `rsync` again <br> 
(NOTE: now we add `-u` option) <br>
and check if file `f77` file copied (it should not)
```bash
rsync -avu --delete /tmp/rs1 127.0.0.1:/tmp/rs2
ls -l /tmp/rs2/rs1
```

