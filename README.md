S3, VPC, Lambda, and EFS Lab
Overview
This project demonstrates an AWS-based pipeline for the AWS Certified Solutions Architect – Associate (SAA) exam preparation. Files uploaded to an S3 bucket (saa-lab-5-7) are automatically moved to an EFS file system (SAA-Lab-EFS) using a Lambda function (S3ToEFSLambda). The setup includes a VPC with public and private subnets, EC2 instances for secure S3 access, and detailed troubleshooting to ensure functionality.
Key Steps to Complete the Project

Create S3 Bucket:
Set up saa-lab-5-7 and upload a test file (test.txt).
Apply a VPC-only bucket policy for secure access.


Set Up VPC:
Create SAA-Lab-VPC with public (SAA-Public-Subnet-A) and private (SAA-Private-Subnet-A) subnets.
Configure route tables, NACLs, and security groups.


Launch EC2 Instances:
Bastion host (SAA-Bastion) in the public subnet.
Private instance (S3-Lab-Private) to access S3 securely.


Integrate EFS and Lambda:
Create SAA-Lab-EFS with an Access Point (/lambda).
Set up S3ToEFSLambda to move S3 files to EFS, triggered by S3 events.
Mount EFS on the private instance to verify file transfers.


Document and Clean Up:
Document the process in s3-vpc-lambda-efs-lab.md with troubleshooting notes.
Clean up resources (EC2, S3, EFS, Lambda).



Troubleshooting Highlights

Fixed S3 access issues by associating the correct route table and adding an S3 VPC Gateway Endpoint.
Resolved EFS mount failures by allowing port 2049 and installing nfs-utils.
Addressed Lambda EFS errors by adding an Access Point and mapping files to /mnt/efs/lambda/.

Full Documentation
For detailed steps, commands, and screenshots, see s3-vpc-lambda-efs-lab.md.
Screenshots

Lambda Logs: Successful file transfer. 


