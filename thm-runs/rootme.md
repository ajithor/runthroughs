initial scans show 22 and 80
80 had a /panel page, which accepts file uploads.
upload a php reverse shell
go to /uploads, open that php rev shells
rev shell! www-data
SUID shows python is enabled for SUID.

gtfo bins
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
root shell!