# Linux Administration and Networking Basics (Level 2) <br> Լինուքսի կառավարում և ցանցային հիմունքներ (փուլ 2)

# sudo (SuperUser DO)

`sudo` (SuperUser DO) թույլ է տալիս վստահված օգտագործողներին կատարել հրամաններ այլ օգտագործողների անունից, մուտքագրելով սեփական գաղտնաբառը:

Հիմնականում կիրառվում է `root`-ի անունից հրամաններ կատարելու համար:

Օգտագործելով `sudo`-ն կարող եք.<br>

✅ Ադմինիստրատիվ իրավունքներ տալ վստահված օգտատերերին՝ առանց `root`-ի գաղտնաբառը հայտնելու<br>
✅ Սահմանել իրավունքներ ըստ օգտագործողներին/հրամանների/սերվերների<br>
✅ Մանրամասն լոգավորել բոլոր գործողությունները (ներառյալ ձախողված փորձերը)<br>
✅ Չփոխել `root`-ի գաղտնաբառը, երբ `sudo` մուտք ունեցող օգտատերը փոխվում է<br>


Հրամանները կատարվում են դեմը գրելով `sudo`

Մոտակա հինգ րոպեի ընթացքում չի պահանջվի նորից գաղտնաբառ մուտքագրել (այս կարգավորումը կարելի է փոխել):

```bash
bash -c 'echo "EUID: $EUID, RUID: $UID"'
```

```bash
sudo bash -c 'echo "EUID: $EUID, RUID: $UID"'
```

```bash
sudo -u nobody bash -c 'echo "EUID: $EUID, RUID: $UID"'
```

```bash
sudo -u sshd bash -c 'echo "EUID: $EUID, RUID: $UID"'
```


Օգտագործողը կարող է տեսնել իրեն թույլատրված հրամանները հետևյալ հրամանով.

```bash
sudo -l
```

Բոլոր կարգավորումները կատարվում են `/etc/sudoers` ֆայլում:
Ուսումնասիրենք այն.

```bash
less /etc/sudoers
```

`/etc/sudoers` ֆայլը խմբագրելու նախատեսված միջոցն է `visudo` հրամանը: 
(փոփոխությունները պահպանելուց առաջ ստուգում է կոնֆիգուրացիայի ճշգրտությունը):

Եթե `/etc/sudoers` ֆայլի կարգավորումներում սխալներ կան, `sudo` հրամանը ամբողջությամբ դադարում է աշխատել։

`visudo` հրամանը կբացի `vim` խմբագրիչը: Որպեսզի դրա փոխարեն բացի `nano`-յով կարող ենք նախապես տալ հետևյալ հրամանը.

```bash
export VISUAL=nano;  visudo
```


Սակայն կա կարգավորումներ ավելացնելու այլ տարբերակ:<br> 
Նկատեք այս տողերը.

```bash
tail -2 /etc/sudoers
```

Դա նշանակում է, որ կարելի է `/etc/sudoers` ֆայլը խմբագրելու փոխարեն ստեղծել լրացուցիչ կարգավորումներ<br>
որպես նոր ֆայլ `/etc/sudoers.d` դիրեկտորիայի մեջ:



`/etc/sudoers`-ի հիմնական հատվածի կառուցվածք.

**USER/GROUP &nbsp;&nbsp;&nbsp;&nbsp;  HOST=(RUNAS_USER:RUNAS_GROUP)&nbsp;&nbsp;&nbsp;&nbsp;   COMMANDS**

Բացատրություն.

`USER/GROUP` կարող է կատարել `COMMANDS` հրամանները (`RUNAS_USER:RUNAS_GROUP`-ի անունից) `HOST` սերվերների վրա

Օրինակներ.
(Այս կարգավորումները կան `/etc/sudoers` ֆայլում)

<pre>
root            ALL=(ALL:ALL)        ALL           # root-ը կարող է ամեն ինչ
</pre>

<pre>
%wheel          ALL=(ALL)            ALL           # wheel խմբի անդամները կարող են ամեն ինչ
</pre>



Այժմ ավելցնենո նոր կարգավորումներ:
Նախ ստեղծեք `devopusr1` օգտագործող և `devops` խումբ: 
Ավելացրեք `devopusr1` օգտագործողին `devops` խումբ:


Այժմ ավելացրեք նոր կարգավորում, ըստ որի `devops` խմբի անդամները կարող են կատարել ամեն ինչ առանց գաղտնաբառի: 

```bash
echo "%devops         ALL=(ALL)  NOPASSWD: ALL"     >> /etc/sudoers.d/devopsgrp
```

Ստուգենք.

```bash
su - devopusr1
```

```bash
sudo -l
```

```bash
sudo id
```

```bash
sudo su -
```

Ստեղծեք այլ օգտագործողներ

```bash
useradd user1 ; useradd user2; useradd user3
```

Ավելացրեք այլ կարգավորումներ.

```bash
echo "user1           ALL=(nobody)        NOPASSWD: /usr/bin/id"     >> /etc/sudoers.d/users123
echo "user3           ALL=(root)          NOPASSWD: /usr/bin/whoami, /usr/bin/touch, /usr/bin/id"     >> /etc/sudoers.d/users123
echo "user3           ALL=(nobody,sshd)   NOPASSWD: /usr/bin/whoami, /usr/bin/touch, /usr/bin/id"     >> /etc/sudoers.d/users123
```

Ստուգեք.

```bash
su - user1
```

```bash
sudo touch /opt/test.txt
```

```bash
sudo id
```

```bash
sudo whoami
```

