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


Բոլոր կարգավորումները կարելի է ստուգել հետևյալ հրամանով

```bash
visudo -c
```



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



Այժմ ավելցնենք նոր կարգավորումներ:

Նախ ստեղծեք `devopusr1` օգտագործող և `devops` խումբ: 
Ավելացրեք `devopusr1` օգտագործողին `devops` խումբ:


Ավելացրեք նոր կարգավորում, ըստ որի `devops` խմբի անդամները կարող են կատարել ամեն ինչ առանց գաղտնաբառի: 

```bash
echo "%devops         ALL=(ALL)  NOPASSWD: ALL"     >> /etc/sudoers.d/devopsgrp ; chmod 440 /etc/sudoers.d/devopsgrp
```

Ստուգենք.

```bash
visudo -c
```

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
echo "user1           ALL=(nobody)        NOPASSWD: /usr/bin/touch"     >> /etc/sudoers.d/user1 ; chmod 440 /etc/sudoers.d/user1
echo "user2           ALL=(nobody,sshd)   NOPASSWD: /usr/bin/whoami, /usr/bin/touch"     >> /etc/sudoers.d/user2 ;  chmod 440 /etc/sudoers.d/user2 
echo "user3           ALL=(root)          NOPASSWD: /usr/bin/id, /usr/bin/mkdir, /usr/bin/touch"     >> /etc/sudoers.d/user3 ; chmod 440 /etc/sudoers.d/user3
```


```bash
visudo -c
```

Ստուգեք տարբեր հրամաններ `user1`-ի, `user2`-ի, `user3`-ի անունից

```bash
su - user1
```

```bash
sudo -l
```

```bash
sudo touch /opt/testfile1
```

```bash
sudo mkdir /opt/dir1
```


```bash
sudo id
```

```bash
sudo whoami
```


Այժմ ստեղծենք սխալներով ֆայլեր

```bash
echo "wrong syntax line"     >> /etc/sudoers.d/err1 ; chmod 440 /etc/sudoers.d/err1
```

```bash
echo "this line is another error"     >> /etc/sudoers.d/err2 ; chmod 440 /etc/sudoers.d/err2
```

Ստուգեք

```bash
visudo -c
```

> Չնայած այս դեպքում մնացած կարգավորումները պետք է աշխատեն,<br> 
> `sudo` կարգավորումներում սխալներ ունենալը ռիսկային է,<br>
> ուստի անհրաժեշտ է ստուգել ամեն ինչ, կամ միշտ օգտվել `visudo`-ից կարգավորումները խմբագրելու համար:<br>
> Լրացուցրի ֆայլերը նույնպես կարելի է խմբագրել `visudo`-ով<br> 
> Արդյունքում նաև ֆայլերը մի անգամից կունենան `440` թույլտվությունը, ինչը նույնպես պահանջ է: <br>
> `visudo -f /etc/sudoers.d/err3`

# PRACTICE

1. Ստեղծեք `project1` խումբ և `tester1` ու `tester2` օգտագործողներ: 

Ավելացրեք `tester1` ու `tester2` օգտագործողներին `project1` խումբ:

Ավելացրեք նոր կարգավորում `visudo -f /etc/sudoers.d/project1`, ըստ որի `project1` խմբի անդամները կարող են 
կատարել միայն `/usr/bin/touch` հրամանը, միայն `root`-ի անունից, բայց մուտքագրելով սեփական գաղտնաբառը:


2. Ստեղծեք `admin1` օգտագործող: 

Ավելացրեք նոր կարգավորում `visudo -f /etc/sudoers.d/admin1`, ըստ որի `admin1` օգտագործողը կարող է մուտքագրելով սեփական գաղտնաբառը 
կատարել հետևյալ հրամանները.

* /usr/bin/mkdir
* /usr/bin/touch
* /usr/bin/id
* /usr/bin/whoami 


