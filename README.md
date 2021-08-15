# vm-import



aws ec2 describe-instances

aws sts get-caller-identity

aws sts  decode-authorization-message --encoded-message

bucket_name="vmma"

vm_image_name="*****.vmdk"

reference
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
if [ "$pippo" = "true" ]; then echo "viva"; else echo "nulla"; pippo=false;  fi




