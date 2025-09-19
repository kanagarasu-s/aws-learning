Backup EC2-Instance to s3 bucket store .vmdk format.
=====================================================
## ec2-instance stop 
```
aws ec2 stop-instances --instance-ids i-0abcd1234efgh567-sample
```
## create IAM User account and permission in aws 
```
Sign in to AWS Management Console → IAM → Users → Add user.

Enter a username (e.g. backup-user). Check Programmatic access (and Console access if you want them to sign in).

Permissions: attach a policy (see below for a least-privilege example) or add to a group. Finish and download/store the credentials (Access key ID & Secret).
(Official IAM user guide).
```
## create custom role path.
```
IAM -> role -> trust policy
```
## create custom role for newrole
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
## create custom policy for same role.
```
add custom policy and check s3 bucket name
```
## trust policy 
```
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
## create custom policy add IAM USER Attach.
```
create permission linein policy
```
## trust policy
```
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
