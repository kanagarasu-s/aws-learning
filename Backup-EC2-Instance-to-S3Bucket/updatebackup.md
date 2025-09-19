# Backup EC2 Instance to S3 in .VMDK Format

This guide explains how to export an EC2 instance and store it as a **VMDK** file in an **S3 bucket**.

---

## 1. Stop the EC2 Instance
Before exporting, stop the instance. Replace the instance ID with yours.

```
aws ec2 stop-instances --instance-ids i-0abcd1234efgh567

```
## Create an IAM User

```
Open AWS Console → IAM → Users → Add user

Enter a username (example: backup-user)

Select Programmatic access (CLI access)

Attach permissions (we’ll add a custom policy later)

Download and store the Access Key ID and Secret Key

```
## Create IAM Role for Export

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
      "Principal": {
        "Service": "vmie.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```
Name it something like VMImportExportRole

## Attach Policy to Role

```
Create a custom policy and attach it to the role:


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:PutObject",
        "s3:GetBucketAcl"
      ],
      "Resource": [
        "arn:aws:s3:::newtestvmi",
        "arn:aws:s3:::newtestvmi/*"
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
## Give IAM User Permissions

```
Edit the inline policy for your IAM user and add:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateInstanceExportTask",
        "ec2:DescribeInstances",
        "ec2:DescribeExportTasks"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::newtestvmi",
        "arn:aws:s3:::newtestvmi/*"
      ]
    }
  ]
}

```
## Create an S3 Bucket

```
Go to S3 → Create bucket

Example bucket name: newtestvmi

Ensure the region matches your EC2 instance region

```
## Update Bucket ACL

```
Go to S3 → Bucket → Permissions → Access Control List (ACL)

Add Read ACP + Write for the AWS export service

Add the Region-specific canonical account ID for your region
Example for Hyderabad:

```

77ab5ec9eac9ade710b7defed37fe0640f93c5eb76ea65a64da49930965f18ca

```
## Start the Export Task

Run the command to start export:

```
aws ec2 create-instance-export-task \
    --description "My EC2 export $(date '+%b %d %H:%M')" \
    --instance-id i-0abcd1234efgh567 \
    --target-environment vmware \
    --export-to-s3-task '{
        "ContainerFormat": "ova",
        "DiskImageFormat": "VMDK",
        "S3Bucket": "newtestvmi",
        "S3Prefix": "vms/"
    }'

```
## Monitor Export Progress

Check the export task status:

```
aws ec2 describe-export-tasks \
    --query "ExportTasks[*].{Description:Description,ExportTaskId:ExportTaskId,State:State,S3Bucket:ExportToS3Task.S3Bucket,InstanceId:InstanceExportDetails.InstanceId}" \
    --output table

```
