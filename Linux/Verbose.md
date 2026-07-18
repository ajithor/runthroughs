only port 80 open, werkzeug python flask webapp
Login page
gobuster shows not much

We can register a user, so we go with test.
Once logged in as test, we see there's "No msgs from admin. no msgs from johnny", listing the users.
Since we had burp listening passively, we see an endpoint `/api/users/all`
`curl --path-as-is ip/api/user/all` gives us passwords and all deets of all users.
There, we find admin : YouWontGetThisPasswordYouNoobLOL123

Once we login, mfa appears. Strange, since the api endpoint indicated mfa:none

Anyways, we see the mfa code entry doesnt lockout even after a bunch of attempts!

So, now we capture a req, use the headers `Content-Type: ` and `Cookie: session=` along with data `code: FUZZ`.
We now need to fuzz all the 4 digit codes, using ffuf
To do that, we first generate all 4 digit codes using `seq `
`seq -w 0 9999 > ids.txt`
Now, we ffuf with 3 headers
```zsh
ffuf -X POST -w ids.txt -u http://10.0.204.56/mfa -d 'code=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Cookie: session=<INSERT SESSION>' -fc 200 -t 3
```