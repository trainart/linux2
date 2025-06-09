# Linux Administration and Networking Basics (Level 2) <br> Լինուքսի կառավարում և ցանցային հիմունքներ (փուլ 2)

# sudo (SuperUser DO)

`sudo` (SuperUser DO) թույլ է տալիս վստահված օգտագործողներին կատարել հրամաններ այլ օգտագործողների անունից, մուտքագրելով սեփական գաղտնաբառը:

Հիմնականում կիրառվում է `root`-ի անունից հրամաններ կատարելու համար:

Օգտագործելով `sudo`-ն կարող եք.
✅ Ադմինիստրատիվ իրավունքներ տալ վստահված օգտատերերին՝ առանց `root`-ի գաղտնաբառը հայտնելու
✅ Սահմանել իրավունքներ ըստ օգտագործողներին/հրամանների/սերվերների
✅ Մանրամասն լոգավորել բոլոր գործողությունները (ներառյալ ձախողված փորձերը)
✅ Չփոխել `root`-ի գաղտնաբառը, երբ `sudo` մուտք ունեցող օգտատերը փոխվում է


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

Բոլոր կարգավորումները կատարվում են `/etc/sudoers` ֆայլում, որի խմբագրելու նախատեսված միջոցն է `visudo` հրամանը: 
(փոփոխությունները պահպանելուց առաջ ստուգում է կոնֆիգուրացիայի ճշգրտությունը):

Եթե `/etc/sudoers` ֆայլի կարգավորումներում սխալներ կան, `sudo` հրամանը ամբողջությամբ դադարում է աշխատել։

`visudo` հրամանը կբացի `vim` խմբագրիչը: Որպեսզի դրա փոխարեն բացի `nano`-յով կարող ենք նախապես տալ հետևյալ հրամանը.

```bash
export VISUAL=nano;  visudo
```

`/etc/sudoers`-ի հիմնական հատվածի կառուցվածք.

՝USER/GROUP    HOST=(RUNAS_USER:RUNAS_GROUP)   COMMANDS՝

Բացատրություն.

`USER/GROUP` կարող է կատարել `COMMANDS` հրամանները (`RUNAS_USER:RUNAS_GROUP`-ի անունից) `HOST` սերվերների վրա

Օրինակներ.

<pre>
root            ALL=(ALL:ALL)        ALL           # root-ը կարող է ամեն ինչ
</pre>

<pre>
%wheel          ALL=(ALL)            ALL           # wheel խմբի անդամները կարող են ամեն ինչ
</pre>

<pre>
%devops         ALL=(ALL)  NOPASSWD: ALL           # devops խմբի անդամները կարող են ամեն ինչ առանց գաղտնաբառի
</pre>

<pre>
user1           ALL=(ALL)           ALL
user2           ALL=(ALL)           NOPASSWD: /usr/bin/id
user3           ALL=(root)          NOPASSWD: /usr/bin/whoami, /usr/bin/touch, /usr/bin/id
</pre>

```bash
useradd user1 ; useradd user2; useradd user3
```

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










