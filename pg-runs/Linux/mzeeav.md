ports 80 and 22 open
port 80 has a file upload page
we start with uploading a test.txt file
It gets renamed to file.tmp
we upload a php rev shell, it gets renamed to file.tmp

It takes us to listing page, as soon as we upload something, and shows us the existing files
there are 2 pre-existing exe files. upon visiting that, it gets downloaded
So I try uploading an exe, same one, renamed as 'whoisi.exe'
gets uploaded
renaming the same thing to whoisi.exe.php
gets uploaded
renamed the php rev shell to rshell.exe.php - > gets renamed to file.tmp
So that means the server is either checking for either filetype or MAGIC bytes

we fire burp up, and change the content type to that of exe and try to upload rshell.exe.php
the filetype for both is application/x-php
So we nee to hexeditor it

we first check the magic bytes for the whoami.exe we downloaded from the server, and it is `4D 5A`
on the first try, it doesnt work, because the hexeditor overwrites `<?` as MZ
and so the file was just getting displayed

Upon realizing, I modify this in vi, put the MZ in first line, `<? php` in the second line, and reupload
upon visiting /upload/rr.exe.php 
www-data!

---
looking into /opt, we find a fileS with strnage permissions, s--s--x
running `./fileS` we see a "."
kinda deduced it is a file lister, lie ls
`./fileS /root` shows the files inside /root

after a little struggle, `./fileS --help` brings up the help menu
it says `-exe CMD` would execuet command, but it errors out

HINT TAKEN - `-version` shows the version as a find binary

This means find binary is disguised as fileS, and so the gtfo bins would work
`./fileS . -exec /bin/sh -p \; -quit`

root!

---
---
---
from a walkthrough, we find the source code for the upload.php was present in the backup.zip, which, clear as day - looks for magic byte