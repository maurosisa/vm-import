# vm-import



aws ec2 describe-instances

aws sts get-caller-identity

aws ssm get-inventory

aws sts  decode-authorization-message --encoded-message

bucket_name="vmma"

vm_image_name="*****.vmdk"

reference
https://www.linuxvmimages.com/images/centos-8/
https://docs.aws.amazon.com/vm-import/latest/userguide/vmie_prereqs.html#vmimport-role

IMPORT 

echo '[
  {
    "Description": "centosv7",
    "Format": "vmdk",
    "UserBucket": {
        "S3Bucket": "'${bucket_name}'",
        "S3Key": "'${vm_image_name}'"
    }
}]
' > containers.json

aws ec2 import-image --description "redhat" --disk-containers "file://containers.json"


aws ec2 describe-import-image-tasks --import-task-ids "import-ami-*****"
reference
https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html#import-vm

extra :

user inline policy create role

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:PutRolePolicy"
            ],
            "Resource": "arn:aws:iam::*:role/*"
        }
    ]
}

inline policy EC2 describe 

{
    "Version": "2012-10-17",
    "Statement": [
        {
            
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances"
            ],
            "Resource": "*"
        }
    ]
}


inline policy s3  

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::vmma"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*Object*"
            ],
            "Resource": [
                "arn:aws:s3:::vmma/*"
            ]
        }
    ]
}


decode policy 

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowStsDecode",
            "Effect": "Allow",
            "Action": "sts:DecodeAuthorizationMessage",
            "Resource": "*"
        }
    ]
}



s3 policy

https://docs.databricks.com/administration-guide/cloud-configurations/aws/instance-profiles.html
https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_s3_rw-bucket.html

policy
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html

common errors
https://acloudguru.com/blog/engineering/fixing-5-common-aws-iam-errors

if [ "$pippo" = "true" ]; then echo "viva"; else echo "nulla"; pippo=false;  fi


TUNNEL

0.install AWS cli 
Install the latest AWS Command Line Interface

Windows x64: https://s3.amazonaws.com/aws-cli/AWSCLI64PY3.msi
Linux pip install awscli



1 create programatic ssmuser 

1.1 aws configure

2 attach ssm policy

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ssm:StartSession",
                "ssm:TerminateSession",
                "ssm:ResumeSession",
                "ssm:DescribeSessions",
                "ssm:GetConnectionStatus"
            ],
            "Effect": "Allow",
            "Resource": [
                "*"
            ]
        }
    ]
}

3 create IAM role con AmazonEC2RoleforSSM policy

4 attach role to instance ( check AWS system manager ssm managed instances )

5 Install the latest session manager plugin,

Windows: https://s3.amazonaws.com/session-manager-downloads/plugin/latest/windows/SessionManagerPluginSetup.exe

Linux:

64-bit
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/linux_64bit/session-manager-plugin.rpm" -o "session-manager-plugin.rpm"

sudo yum install -y session-manager-plugin.rpm


sudo systemctl status amazon-ssm-agent


6 
 
# find the instance ID based on Tag Name
INSTANCE_ID=$(aws ec2 describe-instances \
               --filter "Name=tag:Name,Values=CodeStack/NewsBlogInstance" \
               --query "Reservations[].Instances[?State.Name == 'running'].InstanceId[]" \
               --output text)
# create the port forwarding tunnel
aws ssm start-session --target $INSTANCE_ID \
                       --document-name AWS-StartPortForwardingSession \
                       --parameters '{"portNumber":["443"],"localPortNumber":["9999"]}'
# Windows                      
aws ssm start-session --target "Your Instance ID" --document-name AWS-StartPortForwardingSession --parameters "portNumber"=["443"],"localPortNumber"=["9999"]

https://infra.engineer/aws/57-aws-port-forwarding-via-an-ssh-tunnel-to-an-ec2-using-systems-manager

