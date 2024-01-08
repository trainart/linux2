# Linux Administration and Networking Basics (level 2) Linux-ի կառավարում և ցանցային հիմունքներ (փուլ 2)

## Managing Software packages

### Package Management

![img_1.png](images2/img_1.png)
<br><br>
![img_2.png](images2/img_2.png)
<br><br>
![img_3.png](images2/img_3.png)
<br><br>
![img_4.png](images2/img_4.png)
<br><br>
![img_5.png](images2/img_5.png)
<br><br>
![img_6.png](images2/img_6.png)
<br><br>
![img_7.png](images2/img_7.png)
<br><br>

#### Linux File Archives (tar,gzip,bzip2,lzma)


##### tar 
`tar cvf f.tar /etc`    - Create a tar archive

`tar xvf f.tar`		    - Extract tar archive

`tar tvf f.tar | less`	- Test/View tar archive



##### gzip
`gzip f.tar`

`gunzip f.tar.gz`

`tar zcvf   f.tar.gz /etc`

`tar zxvf filename.tar.gz`

`tar ztvf filename.tar.gz`


##### Bzip2
`bzip2 f.tar`

`bunzip2 f.tar.bz2`

`tar jcvf   f.tar.bz2 /etc`

`tar jxvf filename.tar.bz2`

`tar jtvf filename.tar.bz2`


##### xz / lzma
`tar Jcvf  f.tar.xz /etc`

`tar Jxvf filename.tar.xz`

`tar Jtvf filename.tar.xz`




#### Source Code Packages install

Source code Linux packages are basically one of the following: 
`<file>.tgz`
`<file>.tar.gz`
`<file>.tar.bz2`
`<file>.tar.xz`

Source code install consists of the following steps:

* `tar zxvf <file>.tgz` 
* `cd <dir>`
* `./configure`
* `make`
* `make install`


#### Source Code Install Example `htop`

First install `htop` from repository
```bash
yum -y install htop
```

Now from source code

```bash
yum -y install gcc make autoconf automake
```


```bash
wget https://github.com/htop-dev/htop/archive/refs/tags/3.2.2.tar.gz
```

```bash
tar zxvf 3.2.2.tar.gz
```

```bash
cd htop-3.2.2
```

```bash
./autogen.sh && ./configure && make
```

```bash
./htop --version
```

```bash
htop --version
```

* what is the difference ?

* can we have more that one version of same program ?

Install source code compiled `htop` in the system
```bash
make install
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
