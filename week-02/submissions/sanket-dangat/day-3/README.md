# Week 2 - Day 3: IAM Roles and STS

## Name
Sanket Dangat

## Topics Practiced
- Trust policy vs permission policy
- STS AssumeRole
- EC2 role and instance profile
- Cross-service role assumption
- Temporary credentials
- Least-privilege S3 access

## What I Built
I created an IAM role named **Week2Day3EC2S3ReadRole** and attached it to an Amazon EC2 instance named **ec2-s3-integration-test**.

The EC2 instance was configured to use the IAM role instead of storing long-term AWS access keys. Temporary AWS credentials were automatically provided through the EC2 Instance Metadata Service (IMDS), allowing the instance to securely access Amazon S3.

I attached an inline IAM permission policy named **s3-read-permission-policy** to the role. The policy granted only the required read permissions (`s3:ListBucket` and `s3:GetObject`) on the S3 bucket **ec2-s3-read-lab-sanket-dangat**, following the principle of least privilege.

## Allowed Test
I verified that the EC2 instance **ec2-s3-integration-test** could successfully access the S3 bucket **ec2-s3-read-lab-sanket-dangat** using the attached IAM role **Week2Day3EC2S3ReadRole**.

The EC2 instance successfully listed the bucket contents and downloaded the object **day3-test.txt** without configuring AWS access keys manually. The AWS CLI automatically used temporary credentials obtained through the attached IAM role.

This confirmed that:

- The IAM role **Week2Day3EC2S3ReadRole** was correctly attached to the EC2 instance **ec2-s3-integration-test**.

![image](./screenshots/role-attached-ec2.png)

- The trust relationship allowed the Amazon EC2 service (`ec2.amazonaws.com`) to assume the IAM role.

![image](./screenshots/role-trust-policy.png)

- The inline IAM permission policy **s3-read-permission-policy** granted the required read permissions (`s3:ListBucket` and `s3:GetObject`) on the S3 bucket **ec2-s3-read-lab-sanket-dangat**.

![image](./screenshots/s3-Inline-permission-policy.png)

- Temporary AWS credentials were automatically provided through the attached IAM role via the EC2 Instance Metadata Service (IMDS), as verified using `aws sts get-caller-identity` and `aws configure list`.

![image](./screenshots/sts-get-caller-identity-success.png)

![image](./screenshots/aws-configure-list-verification.png)

- The EC2 instance successfully listed the bucket contents and downloaded the object **day3-test.txt** from the S3 bucket **ec2-s3-read-lab-sanket-dangat**.

![image](./screenshots/s3-read-test-success.png)

## Denied Test
I tested a write operation by attempting to upload **write-test.txt** from the EC2 instance **ec2-s3-integration-test** to the S3 bucket **ec2-s3-read-lab-sanket-dangat**.

The operation failed with an `AccessDenied` error because the IAM permission policy **s3-read-permission-policy** granted only read permissions (`s3:ListBucket` and `s3:GetObject`). It did not include write permissions such as `s3:PutObject`.

This confirmed that:

- IAM permissions were enforced correctly.
- The IAM role **Week2Day3EC2S3ReadRole** followed the principle of least privilege.
- The EC2 instance could perform only the actions explicitly allowed by the IAM permission policy.

![image](./screenshots/s3-write-access-denied.png)

## What I Learned

This lab helped me understand how AWS IAM roles provide secure access to AWS services without requiring long-term AWS access keys.

AWS IAM uses both **trust policies** and **permission policies**, each serving a different purpose:

- A **trust policy** defines **who can assume an IAM role**. In this lab, the trust policy allowed the Amazon EC2 service (`ec2.amazonaws.com`) to assume the IAM role **Week2Day3EC2S3ReadRole**.

- A **permission policy** defines **what actions the IAM role can perform** after it has been assumed. In this lab, the inline policy **s3-read-permission-policy** allowed the EC2 instance to list the bucket and read objects from the S3 bucket **ec2-s3-read-lab-sanket-dangat**.

I also learned that:

- IAM roles are more secure than storing long-term AWS access keys on EC2 instances because AWS provides temporary credentials automatically.
- AWS Security Token Service (STS) creates temporary credentials when the EC2 service assumes the IAM role. These credentials are automatically rotated and expire after a limited time.
- An **instance profile** is the mechanism that makes an IAM role available to an EC2 instance. When the role is attached to an EC2 instance, the AWS CLI and SDKs automatically retrieve temporary credentials through the EC2 Instance Metadata Service (IMDS).
- Following the **principle of least privilege** reduces security risks by granting only the permissions required. In this lab, the role was limited to `s3:ListBucket` and `s3:GetObject`, preventing write operations such as `s3:PutObject`.
- Commands such as `aws sts get-caller-identity` and `aws configure list` are useful for verifying the IAM identity and confirming that credentials are being obtained from the attached IAM role.

In simple terms:

- **Trust Policy** = "Who can assume this role?"
- **Permission Policy** = "What can this role do after it is assumed?"
- **IAM Role** = "Provides temporary AWS credentials to trusted AWS services."
- **Instance Profile** = "Makes an IAM role available to an EC2 instance."

## Where I Got Stuck

- No blockers.

## Cleanup
Deleted all resources created during the lab:

- Terminated the EC2 instance **ec2-s3-integration-test** and verified that no unnecessary EBS volumes remained.

![image](./screenshots/ec2-clean-up.png)

- Deleted the IAM role **Week2Day3EC2S3ReadRole** and its inline policy **s3-read-permission-policy**.

![image](./screenshots/role-cleanup.png)

- Deleted the S3 bucket **ec2-s3-read-lab-sanket-dangat** and the test objects.

![image](./screenshots/s3-cleanup.png)

## LinkedIn Post
Add your post link.