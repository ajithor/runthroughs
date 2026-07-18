nmap shows ports 80 and 22 open
visiting the webpage, we see it is just "hello world".
But the source code asks us to go to /nibbleblog
There, we find nibbleblog is a CMS.

`sudo feroxbuster -u http://10.129.235.221/nibbleblog -w /root/.local/share/AutoRecon/wordlists/dirbuster.txt`
feroxbuster on /nibbleblog reveals a /nibbleblog/README page, which tells us the version of the CMS is 4.0.3 which is vulnerable to RCE, but we need the login

After some inspection, we find a /nibbleblog/install.php, which asks us to upgrade.
Which, when updated, takes us to /nibbleblog/content/private dir
And says db has been updated at/private/config.xml and comments.xml where we find admin's email admin@nibbles.com and username admin

Through a walkthrough, we find the password is nibbles. There is a blacklisting in place. So we wouldnt have been able to get the hydra running anyways.
Once we login, we can go to the metasploit exploit's instructions, and follow it mannually.
Basically, the POC is,
1. Obtain admin creds
2. Activate my image plugin by visiting admin.php?controller=plugins&action=install&plugin=my_image
3. upload php rev shell, ignore warnings
4. visit /content/private/plugins/my_image/image.php
Alternatively, there's an exploit on github for it, and it directly gives us the shell

We follow the mannual method, copied the `rshell.php` to the cwd
Go to admin.php?controller=plugins&action=install&plugin=my_image upload the rshell
Go to /content/private/plugins/my_image/image.php while listening, and got nibbler shell

Privesc was pretty simple. Just had to do `sudo -l` it showed we can run a monitor.sh script which we have full access over.
Just changed the code to mkfifo and `sudo monitor.sh` and got root shell