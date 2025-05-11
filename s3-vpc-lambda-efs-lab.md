S3, VPC, Lambda, and EFS Lab (May 10, 2025)
Overview
This lab was completed as part of AWS Certified Solutions Architect – Associate (SAA) exam preparation. The goal was to create an S3 bucket, set up a VPC with public and private subnets, launch EC2 instances to securely access S3, and use a Lambda function to move S3 files to an EFS file system. The lab was performed in the us-west-2 region using an AMI (SAA-Lab-AMI-20250505) and an IAM role (SAA-Lab-Role).
Prerequisites

AWS account with IAM permissions via SAA-Lab-Role (policies: AmazonS3FullAccess, AmazonEC2FullAccess, AmazonVPCFullAccess, AWSLambdaFullAccess, AmazonElasticFileSystemFullAccess).
AMI: SAA-Lab-AMI-20250505 (includes AWS CLI, Python, Git).
.pem file: ~/aws-keys/saa-lab-key.pem for SSH.
GitHub repo: SAA-AMI for documentation.
Estimated time: 3–3.5 hours.

Steps
Step 1: Create an S3 Bucket
Created an S3 bucket to store files:
aws s3 mb s3://saa-lab-5-7 --region us-west-2
echo "Test file for SAA lab" > test.txt
aws s3 cp test.txt s3://saa-lab-5-7/
aws s3 ls s3://saa-lab-5-7/

Initially set a basic bucket policy to allow public GetObject access (later secured):
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::saa-lab-5-7/*"
    }
  ]
}

Applied the policy:
aws s3api put-bucket-policy --bucket saa-lab-5-7 --policy file://policy.json

Step 2: Create a VPC with Public and Private Subnets
Set up a VPC (SAA-Lab-VPC, vpc-0a474013900939c66, 10.0.0.0/16):

Public Subnet: SAA-Public-Subnet-A (subnet-03f3fadf74d9f0d18, 10.0.1.0/24, us-west-2a).
Private Subnet: SAA-Private-Subnet-A (subnet-035ac0e527841cd1a, 10.0.2.0/24, us-west-2a).
Internet Gateway: SAA-Lab-IGW.
Route Tables:
SAA-Public-RT: 0.0.0.0/0 → SAA-Lab-IGW.
SAA-Private-RT: No internet access initially.



Step 3: Configure NACLs and Security Groups

NACL: SAA-Lab-NACL (acl-0f56855e90809de72):
Inbound: Allow TCP 22 (SSH) from my IP, Rule #200 Allow all traffic.
Outbound: Rule #200 Allow all traffic.
Associated with both subnets.


Security Group: SAA-Lab-SG (sg-05e006ab48a2571e4):
Inbound: SSH (port 22) from my IP.
Outbound: All traffic.



Step 4: Launch EC2 in Private Subnet for Secure S3 Access

Launched Private EC2: S3-Lab-Private (i-0359b27988fd4ee4f, 10.0.2.126) in SAA-Private-Subnet-A:
AMI: SAA-Lab-AMI-20250505.
IAM Role: SAA-Lab-Role.
Security Group: SAA-Lab-SG.
No public IP.


Bastion Host: SAA-Bastion (52.38.163.221, 10.0.1.36) in SAA-Public-Subnet-A for SSH:# Local to bastion
scp -i ~/aws-keys/saa-lab-key.pem ~/aws-keys/saa-lab-key.pem ec2-user@52.38.163.221:~/.ssh/
# Bastion to private
ssh -i ~/.ssh/saa-lab-key.pem ec2-user@10.0.2.126


Access S3:aws s3 ls s3://saa-lab-5-7/
aws s3 cp s3://saa-lab-5-7/test.txt ./test-download.txt
cat test-download.txt

Output: Test file for SAA lab.
Secure Bucket Policy (VPC-only):{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::saa-lab-5-7/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVpc": "vpc-0a474013900939c66"
        }
      }
    }
  ]
}

Applied:aws s3api put-bucket-policy --bucket saa-lab-5-7 --policy file://policy-vpc.json



Step 4b: Create EFS and Lambda to Move S3 Files to EFS

Created EFS:
Name: SAA-Lab-EFS.
VPC: SAA-Lab-VPC.
Mount target: SAA-Private-Subnet-A (us-west-2a, SAA-Lab-SG).


Created EFS Access Point:
Name: SAA-Lab-EFS-AccessPoint.
Root directory: /lambda.
POSIX UID/GID: 1000.
Permissions: 755.


Created Lambda IAM Role:
Role: SAA-Lab-Lambda-Role.
Policies: AWSLambdaBasicExecutionRole, AmazonS3ReadOnlyAccess, AmazonElasticFileSystemClientFullAccess.


Created Lambda Function:
Name: S3ToEFSLambda.
Runtime: Python 3.9.
Role: SAA-Lab-Lambda-Role.
VPC: SAA-Lab-VPC, SAA-Private-Subnet-A, SAA-Lab-SG.
File system: SAA-Lab-EFS, Access Point: SAA-Lab-EFS-AccessPoint, Mount path: /mnt/efs.
Timeout: 1 minute.
Code:import json
import boto3
import os

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    local_path = f"/mnt/efs/{key}"
    s3_client.download_file(bucket, key, local_path)
    return {
        'statusCode': 200,
        'body': json.dumps(f"File {key} copied to EFS")
    }




Added S3 Trigger:
Bucket: saa-lab-5-7.
Event: All object create events.


Tested:
Uploaded:echo "Test file 9" > test9.txt
aws s3 cp test9.txt s3://saa-lab-5-7/


Mounted EFS on private instance:sudo mkdir /mnt/efs
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 <efs-dns>:/ /mnt/efs


Verified:ls /mnt/efs/lambda
cat /mnt/efs/lambda/test9.txt


Output:test9.txt
Test file 9





Step 5: Document in GitHub
Documented the lab in this file (s3-vpc-lambda-efs-lab.md).

Cleaned up extraneous test9 file from troubleshooting:rm -rf /mnt/efs/lambda/test9


Committed to GitHub repo (SAA-AMI):git add s3-vpc-lambda-efs-lab.md
git commit -m "Add S3, VPC, Lambda, EFS lab with troubleshooting journey"
git push origin main



Step 6: Clean Up

Terminated instances:aws ec2 terminate-instances --instance-ids i-0359b27988fd4ee4f <bastion-instance-id> --region us-west-2


Deleted bucket:aws s3 rb s3://saa-lab-5-7 --force --region us-west-2


Deleted EFS: SAA-Lab-EFS (via Console).
Deleted Lambda: S3ToEFSLambda (via Console).
Kept VPC for future labs.

Troubleshooting

SSH Issue: Ensured .pem file in ~/aws-keys with chmod 400.
S3 AccessDenied: Fixed by associating the correct route table with the private subnet and adding an S3 VPC Gateway Endpoint.
EFS Mount Failed:
Added security group rule for port 2049:aws ec2 authorize-security-group-ingress --group-id sg-05e006ab48a2571e4 --protocol tcp --port 2049 --cidr 10.0.2.0/24 --region us-west-2


Installed nfs-utils on the private instance.


Lambda EFS Error:
Initial error: EFS mount point not found: /mnt/efs.
Fixed by adding EFS file system to Lambda with mount path /mnt/efs.
Required an EFS Access Point (SAA-Lab-EFS-AccessPoint) with root directory /lambda.
Files located in /mnt/efs/lambda/ on the private instance due to Access Point mapping.




Screenshots

Lambda Logs: Showing successful file transfer. 
I forgot to get a screenshot of the EFS cat of test9, but the logs show success, finally 

AWS-GCP Notes

AWS VPC subnets are AZ-specific, unlike GCP’s region-based subnets.
AWS NACLs are stateless, while GCP firewall rules are stateful.
AWS Lambda with EFS is unique; GCP Cloud Functions lack direct file system integration.


