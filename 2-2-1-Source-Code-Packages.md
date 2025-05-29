# Linux Administration and Networking Basics (Level 2) Linux-ի կառավարում և ցանցային հիմունքներ (փուլ 2)

## Managing Software packages (Source Code)

### Linux File Archives (tar,gzip,bzip2,lzma)

Ի՞նչ է Linux File Archive-ը

Արխիվացումը ֆայլերի և դիրեկտորիաների 1) մեկ ֆայլի մեջ միավորելու ու 2) ծավալը նվազեցնելու մեթոդ է՝ հեշտ տեղափոխման և պահպանման համար: 

Լինուքսում ամենատարածված արխիվացման գործիքներն են `tar`, `gzip`, `bzip2` և `lzma`: 

Ի տարբերություն ZIP-ի, Լինուքսում վերը նշված 2 գործառույթները **բաշխված են** գործիքների միջև:

Նախ տվյալները `tar`-ով միավորում մեկ արխիվում:

Ապա օգտագործում են սեղմման (ծավալը նվազեցնելու) գործիքներից մեկը (`gzip`, `bzip2` կամ `xz/lzma`)


##### **tar (Tape Archive)**

Օգտագործվում է մի քանի ֆայլեր մեկ արխիվում միավորելու համար՝ առանց սեղմման:

>  Ուշադրություն դարձրեք.<br>
> `tar`-ի option-ները առանց գծիկի են<br>
> 


Create a tar archive

```bash
tar cf f.tar /etc
```

> TASK: Move errors to `/tmp/err` file

> ANALYZE: <br>
> Run `head -1 /tmp/err` <br>
> Let's understand what it means <br>
> It makes archive path relative and helps to avoid mistakes to overwrite original files.

Create a tar archive **with verbose output** (`v` option)
 
```bash
tar cvf f.tar /etc
```

Extract tar archive

```bash
tar xf f.tar
```

List tar archive

```bash
tar tf f.tar | less
```


##### **gzip (GNU zip)**

Օգտագործվում է ֆայլերը սեղմելու համար (սովորաբար `.tar` արխիվների հետ):


```bash
gzip f.tar
```


```bash
gzip -d f.tar.gz
```

> Another command is `gunzip`<br>
> Find what it is with `which` and `file` <br>
> Examine contents



```bash
tar zcvf f.tar.gz /etc
```

```bash
tar xf f.tar.gz
```

- **NOTE! You don't need to specify `z`**

```bash
tar tf f.tar.gz
```

- **NOTE! You don't need to specify `z`**


##### **Bzip2**

Ավելի արդյունավետ սեղմման ալգորիթմ, քան `gzip`, բայց ավելի դանդաղ:

```bash
bzip2 f.tar
```

```bash
bzip2 -d f.tar.bz2
```
> Another command is `bunzip2`<br>
> Find what it is with `which` and `file` <br>


```bash
tar jcf f.tar.bz2 /etc
```

```bash
tar xf f.tar.bz2
```
- **NOTE! You don't need to specify `j`**

```bash
tar tf f.tar.bz2
```
- **NOTE! You don't need to specify `j`**


##### **xz / lzma**

Ամենաարդյունավետ սեղմման մեթոդներից մեկը:

Տարբերություն `lzma` և `xz` միջև

`lzma` — հին գործիք, օգտագործում է **LZMA** ալգորիթմ:
`xz` — նոր գործիք, օգտագործում է **LZMA2** և առաջարկում լավագույն սեղմման հարաբերակցություն:

> Ներկա պահին `https://kernel.org/` կայքում օգտագործվում է `xz` սեղմման մեթոդը: 

```bash
xz f.tar
```


```bash
xz -d f.tar.xz
```

> Another command is `unxz`<br>
> Find what it is with `which` and `file` <br>



```bash
tar Jcf  f.tar.xz /etc
```

```bash
tar xf f.tar.xz
```

- **NOTE! You don't need to specify `J`**

```bash
tar tf f.tar.xz
```

- **NOTE! You don't need to specify `J`**


### Source Code Packages install

Source code Linux packages are basically one of the following: 
`<file>.tgz`
`<file>.tar.gz`
`<file>.tar.bz2`
`<file>.tar.xz`

Source code install consists of the following steps:

* `tar xf <file>.tgz` 
* `cd <dir>`
* `./configure`
* `make`
* `sudo make install`

More installation details are to be inside the package in either `README` or `INSTALL` file



#### Source Code Install Example `htop`

First install `htop` from repository
```bash
sudo dnf -y install htop
```

Install `wget`

```bash
sudo dnf -y wget
```

Install all needed stuff for compilation

```bash
sudo dnf -y install gcc make autoconf automake
```

Now let's install `htop` from source code

```bash
wget --inet4-only https://github.com/htop-dev/htop/archive/refs/tags/3.4.1.tar.gz
```

> `--inet4-only` forces `wget` to use IPv4 Only (not IPv6)

```bash
tar xf 3.4.1.tar.gz
```

```bash
cd htop-3.4.1
```

```bash
./autogen.sh && ./configure && make
```

You may get an error. Read the docs to resolve the issue<br>
(HINT: You should find how to install missing `ncurses` package for your distribution).

```bash
less README
```

After installing missing package try again.

```bash
./autogen.sh && ./configure && make
```

Now check

```bash
./htop --version
```

```bash
htop --version
```

* what is the difference ?

* can we have more than one version of same program ?

Install source code compiled `htop` in the system
```bash
sudo make install
```

Now check again

```bash
htop --version
```
* what is the difference ? why ?

Find out more
```bash
which htop
```

```bash
whereis htop
```

#### PRACTICE

You can read install details in README/INSTALL inside the packages.

1. Install latest version of `mc` (`mc-4.8.33.tar.xz` or newer ) from source (http://ftp.midnight-commander.org/)

HINTS!
* For **GLIB** error install `glib2-devel` package (for RH/Centos/Rocky/AlmaLinux...)

* For **S-Lang** error try `cat INSTALL | grep with-screen` to find solution.
  (as variant you may have already `ncurses` installed above)



2. Install latest version of `nano` from source (https://www.nano-editor.org/download.php)
