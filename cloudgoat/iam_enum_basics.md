Goal - Find the 5 hidden flags (formatted as `HSM{text}` or similar) by manually enumerating IAM resources and metadata using the AWS CLI.
```zsh
cloudgoat config whitelist --auto
cloudgoat create iam_enum_basics
aws configure --profile bob
aws sts get-caller-identity --profile bob
#cg-bob-cgidzooap76wsu
```

Time to enumerate
```zsh
##### 1.inline policies
aws iam get-user --profile bob
"Key": "Message",
"Value": "Enumerate me"

aws iam list-user-policies --user-name cg-bob-cgidzooap76wsu --profile bob
"PolicyNames":
"cg-flag2-inline-policy-cgidzooap76wsu"

aws iam get-user-policy --user-name cg-bob-cgidzooap76wsu --policy-name cg-flag2-inline-policy-cgidzooap76wsu --profile bob
HSM1nl1n3p0l1cyd1sc0v3r3d

##### 2.attached policies
aws iam list-attached-user-policies --user-name cg-bob-cgidzooap76wsu --profile bob
"PolicyName": "cg-flag1-managed-policy-cgidzooap76wsu",
"PolicyArn": "arn:aws:iam::266877689244:policy/cg-flag1-managed-policy-cgidzooap76wsu"

"PolicyName": "IAMReadOnlyAccess",
"PolicyArn": "arn:aws:iam::aws:policy/IAMReadOnlyAccess"

aws iam get-policy --policy-arn arn:aws:iam::266877689244:policy/cg-flag1-managed-policy-cgidzooap76wsu --profile bob
HSM{m4n4g3d_p0l1cy_m4st3r}

aws iam get-policy-version --policy-arn arn:aws:iam::266877689244:policy/cg-flag1-managed-policy-cgidzooap76wsu --version-id v1 --profile bob
HSM{s3cr3t_js0n_str1ng}

##### 3.group memberships
aws iam list-groups-for-user --user-name cg-bob-cgidzooap76wsu --profile bob
HSM_gr0up_m3mb3rsh1p_f0und

##### 4.assumable roles
aws iam list-roles --profile bob
#cg-flag4-role-cgidzooap76wsu
aws iam get-role --role-name cg-flag4-role-cgidzooap76wsu --profile bob
HSM-r0l3_trus1_f0und
```