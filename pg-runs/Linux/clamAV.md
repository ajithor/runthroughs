open ports 80, 22, 25, smb, 199, 60000 

nothing on smb, webpage (believe me, alot of time wasted on gobuster)
60000 is an ssh port as well, did not look at 199

25 smtp is Sendmail 8.13.4, which has a public exploit
https://www.exploit-db.com/raw/4761

We thought it was python, did some stupid stuff, then realized it was perl
Thought the code was supposed to give reverse shell, and modified accordingly. later realized it was establishing a bind shell

nc -nv IP 31337 gave root shell