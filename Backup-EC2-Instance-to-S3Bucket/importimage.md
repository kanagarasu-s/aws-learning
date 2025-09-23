# Import VMDK from S3 to AWS EC2

## 1. Prerequisites

* VMDK image uploaded in an S3 bucket.
* IAM Role or User with permissions for `vmimport` and S3 access.
* AWS CLI configured with necessary credentials.
* EC2 instance quota for importing images.

## 2. Create IAM Role for VM Import

The role allows EC2 to access S3 for importing:

```json
{
  "RoleName": "vmimport",
  "AssumeRolePolicyDocument": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": { "Service": "vmie.amazonaws.com" },
        "Action": "sts:AssumeRole"
      }
    ]
  }
}
```

Attach policy `vmimport`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": ["arn:aws:s3:::your-bucket-name","arn:aws:s3:::your-bucket-name/*"]
    },
    {
      "Effect": "Allow",
      "Action": ["ec2:ModifySnapshotAttribute","ec2:CopySnapshot","ec2:RegisterImage","ec2:Describe*"],
      "Resource": "*"
    }
  ]
}
```

## 3. Import Image Using AWS CLI

```bash
aws ec2 import-image \
    --description "My imported VMDK" \
    --license-type BYOL \
    --disk-containers '[{
        "Description": "VMDK Disk",
        "Format": "vmdk",
        "UserBucket": {
            "S3Bucket": "your-bucket-name",
            "S3Key": "vmdk-image.vmdk"
        }
    }]'
```

* **BYOL** = Bring Your Own License (or use `AWS` for AWS licensed OS).
* Returns an `ImportTaskId`.

## 4. Monitor Import Task

```bash
aws ec2 describe-import-image-tasks \
  --query "ImportImageTasks[*].{Description:Description, Progress:Progress, Status:Status, ImportTaskId:ImportTaskId, StatusMessage:StatusMessage}" \
  --output table | \
    sed 's/\(.\{120\}\).*/\1|/'
```

Status can be:

* `pending`
* `active`
* `completed`
* `deleted`

## 5. Launch EC2 from Imported AMI

```bash
aws ec2 run-instances \
    --image-id ami-xxxxxxxx \
    --instance-type t2.medium \
    --key-name MyKeyPair \
    --security-group-ids sg-xxxxxxx \
    --subnet-id subnet-xxxxxxx
```

## Notes

* Ensure your VMDK is **compatible** (VMware vSphere VMDK, not snapshot-based or sparse disk type).
* Import may take **several minutes to hours** depending on image size.
* Region matters: the S3 bucket and import should be in the **same region**.

