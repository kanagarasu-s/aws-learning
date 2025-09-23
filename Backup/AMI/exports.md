# Backup AMI to S3 in .VMDK Format
## Recommended to follow official Documentation:
```
https://docs.aws.amazon.com/vm-import/latest/userguide/required-permissions.html#vmimport-role
```
This guide explains how to export an EC2 instance and store it as a **VMDK** file in an **S3 bucket**.  
Storing EC2 Instance as .vmdk file in S3 is less cost than snapshots.
 
## Prerequisites
1. S3 Bucket - to store the Backup file
2. IAM User with Policy - to perform the Backup actions and logged in using Access Key
3. IAM Role with Policy - to perform the Backup actions (background)
 
(The "vmimport" role is not attached to a specific user or EC2 instance—it is a service role that the AWS VM Import/Export service itself assumes to perform tasks on behalf of the account during the VM import or export process.)
 
## 1. Create an S3 Bucket to store the .VMDK backup file
```sh
Go to S3 → Create bucket
Example bucket name: ec2instancebackup
Object Ownership : ACLs enabled
Object Ownership : Object writer
 
Create Bucket
```
## Update Bucket ACL
```sh
Go to S3 → Bucket → Permissions → Access Control List (ACL)
Add Read ACP + Write for the AWS export service
```
Add the Region-specific canonical account ID for your region
```sh
https://docs.aws.amazon.com/vm-import/latest/userguide/vmexport-prerequisites.html
```
Or Use Common for all canonical account ID:
```sh
All other Regions – c4d8eabf8db69dbe46bfe0e517100c554f01200b104d59cd408e777ba442a322 
```
## 2. Stop the AMI
Before exporting, stop the instance. Replace the instance ID with yours.
```sh
aws ec2 stop-instances --instance-ids i-0abcd1234efgh567
```
## 3. Create IAM Role for Export
```
Go to IAM → Roles → Create Role
```
## Choose Custom trust policy and paste:
```
{
   "Version": "2012-10-17",		 	 	 
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": { "Service": "vmie.amazonaws.com" },
         "Action": "sts:AssumeRole",
         "Condition": {
            "StringEquals":{
               "sts:Externalid": "vmimport"
            }
         }
      }
   ]
}

```
Name it something like VMImportExportRole
 
## Attach Policy to Role
```
Go to IAM → Roles → Add Permission → Create Inline Policy
```
Create a custom policy INLINE and attach it to the role:
```
{
   "Version": "2012-10-17",		 	 	 
   "Statement":[
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket" 
         ],
         "Resource": [
            "arn:aws:s3:::amzn-s3-demo-import-bucket",
            "arn:aws:s3:::amzn-s3-demo-import-bucket/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "s3:GetBucketLocation",
            "s3:GetObject",
            "s3:ListBucket",
            "s3:PutObject",
            "s3:GetBucketAcl"
         ],
         "Resource": [
            "arn:aws:s3:::amzn-s3-demo-export-bucket",
            "arn:aws:s3:::amzn-s3-demo-export-bucket/*"
         ]
      },
      {
         "Effect": "Allow",
         "Action": [
            "ec2:ModifySnapshotAttribute",
            "ec2:CopySnapshot",
            "ec2:RegisterImage",
            "ec2:Describe*"
         ],
         "Resource": "*"
      }
   ]
}

```
## 4a. Create a policy for IAM user
```
Go to IAM → policy → Create policy
```
Paste the following in json
```
{
  "Version": "2012-10-17",		 	 	 
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::amzn-s3-demo-import-bucket",
        "arn:aws:s3:::amzn-s3-demo-import-bucket/*",
        "arn:aws:s3:::amzn-s3-demo-export-bucket",
        "arn:aws:s3:::amzn-s3-demo-export-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CancelConversionTask",
        "ec2:CancelExportTask",
        "ec2:CreateImage",
        "ec2:CreateInstanceExportTask",
        "ec2:CreateTags",
        "ec2:DescribeConversionTasks",
        "ec2:DescribeExportTasks",
        "ec2:DescribeExportImageTasks",
        "ec2:DescribeImages",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeInstances",
        "ec2:DescribeSnapshots",
        "ec2:DescribeTags",
        "ec2:ExportImage",
        "ec2:ImportInstance",
        "ec2:ImportVolume",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:ImportImage",
        "ec2:ImportSnapshot",
        "ec2:DescribeImportImageTasks",
        "ec2:DescribeImportSnapshotTasks",
        "ec2:CancelImportTask"
      ],
      "Resource": "*"
    }
  ]
}

```
## 4b.Create an IAM User
```
Open AWS Console → IAM → Users → Add user
Enter a username (example: backup-user)
Set permissions → Attach policies directly → Filter by Type (Custom Managed)
```
## Create an Access Key for the IAM User
```
Select Programmatic access (CLI access)
Download and store the Access Key ID and Secret Key
```
## 5. Open a Terminal
Install AWS CLI
To install the AWS CLI, run the following commands.
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
Configure AWS
```
aws configure
```
use the above created Access Key of IAM User
 
## Start the Export Task
Run the command to start export:
```
aws ec2 export-image \
    --description "$(date '+%b %d %H:%M') My image export" \
    --image-id ami-1234567890abcdef0 \
    --disk-image-format VMDK \
    --s3-export-location S3Bucket=amzn-s3-demo-export-bucket,S3Prefix=exports/
```
## Monitor Export Progress
Check the export task status:
```
aws ec2 describe-export-image-tasks \
  --query "ExportImageTasks[*].{\
    Description:Description,\
    ExportImageTaskId:ExportImageTaskId,\
    ImageId:ImageId,\
    Status:Status,\
    Progress:Progress,\
    S3Bucket:S3ExportLocation.S3Bucket}" \
  --output table

```
## cancel an export image task
```
aws ec2 cancel-export-task \
    --export-task-id export-ami-1234567890abcdef0
```
