# AWS-CLI-Crash-Course-IAM-
AWS CLI Crash Course to enumerate IAM and S3 and securely deploy an S3 bucket

# Lab: AWS Command Line Crash Course

## Objectives

- Introduce key AWS CLI commands
- Learn how to enumerate IAM and S3
- Learn how to securely deploy an S3 bucket

## Lab Setup

For this lab, you will need to have completed the previous lab to have the required access keys for interacting with the AWS command line.


## Lab - Step by Step Instructions

### 1: AWS Account Enumeration via CLI

1. Start by calling the Security Token Service to make sure your profiles are setup correctly. You should see output similar to the screenshot below. 

> **_Command_**
```bash
sudo aws sts get-caller-identity --profile AWSCloudAdmin
sudo aws sts get-caller-identity --profile s3user
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/cd3df4e2-fd98-4ee1-9d91-26918de5ff35">
</div>

<p align="center"><i><b>Using STS GetCallerIdentity to Check Profiles</b></i></p>

> **_NOTE:_**
If you got any errors running either of these commands there is likely an issue with the access keys you setup. Also, keep in mind that profiles are case-sensitive. Make sure when you are specifying each profile that you are using the correct casing.

2. If the profiles both appear to be setup correctly we can proceed to account enumeration commands. Let's start by enumerating Identity and Access Management resources. The below `list-users` command will list any IAM users within an AWS account. Try it with both the AWSCloudAdmin and s3user accounts. 

You should get an `AccessDenied` error when you run it with the `s3user`. This is because in the previous lab you only applied a least privilege policy to this user ensuring they only have access to necessary information.

> **_Command_**
```bash
sudo aws iam list-users --profile AWSCloudAdmin
sudo aws iam list-users --profile s3user
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/a237cc3b-ff24-4595-aff5-225f83226e8b">
</div>

<p align="center"><i><b>Enumerate IAM Users</b></i></p>

3. Enumerate access keys for accounts with the `list-access-keys` command. 

> **_NOTE:_**
AWS IAM users can have a max of two access keys.

> **_Command_**
```bash
sudo aws iam list-access-keys --user-name s3user --profile AWSCloudAdmin
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/fc052bc6-4c14-44c3-9d24-1cc96185a17c">
</div>

<p align="center"><i><b>Enumerate IAM User Access Keys</b></i></p>

4. Enumerate MFA devices for an account with `list-mfa-devices`.

> **_Command_**
```bash
sudo aws iam list-mfa-devices --user-name s3user --profile AWSCloudAdmin
```
5. List policies that have been attached to a user with `list-attached-user-policies`. Again, try with both profiles and notice how the s3user can't even list their own policies because of the account restrictions we put in place.

> **_Command_**
```bash
sudo aws iam list-attached-user-policies --user-name s3user --profile AWSCloudAdmin
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/9eda00c7-bd8d-42c1-af21-1b64caa43c9f">
</div>

<p align="center"><i><b>List Policies Attached to IAM User</b></i></p>

6. If you wanted to work with different resource types other than IAM you will need to change the service name in the AWS CLI command. In the previous examples you have been interacting with IAM resources by starting each command with `aws iam`. If you wanted to work with virtual machines you would need to change that to `aws ec2`. For S3, `aws s3`, etc. 

> **_Command_**
```bash
sudo aws ec2 describe-instances --region us-east-1 --profile AWSCloudAdmin
```

> **_NOTE:_**
Note that in this command you specified a `region`. Many AWS CLI commands rely on you stating a specific region. For a current list of AWS regions see here: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html

<div align="center">
  <img src="https://github.com/user-attachments/assets/451fb308-9807-46b9-b578-0efd31302874">
</div>

<p align="center"><i><b>List EC2 Instances</b></i></p>

### 2: Deploy an S3 Bucket from the CLI

1. Next, you will deploy an S3 bucket to your account using the s3user. First, list s3 buckets with following command. There should currently be no S3 buckets in your account. 
> **_Command_**
```bash
sudo aws s3 ls --profile s3user
```
2. When you create an S3 bucket the name has to be unique. Run the following two commands to generate a randomized bucket name. Make note of the bucket name (`cloudbreach-********`) that gets output.
> **_Command_**
```bash
BUCKET_NAME="cloudbreach-$(tr -dc 'a-z0-9' < /dev/urandom | head -c8)"
echo $BUCKET_NAME
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/9a449f87-eafb-43e6-8006-b96f80ee9d51">
</div>

<p align="center"><i><b>Generate a Unique S3 Names</b></i></p>

3. Create the S3 bucket resource with the following command.
> **_Command_**
```bash
sudo aws s3 mb s3://$BUCKET_NAME --region us-east-1 --profile s3user
```
<div align="center">
  <img src="https://github.com/user-attachments/assets/dbe4f6eb-4803-4edb-902d-cc30db5a7237">
</div>

<p align="center"><i><b>Create S3 Bucket</b></i></p>

4. List S3 buckets again to verify it was successfully created.
> **_Command_**
```bash
sudo aws s3 ls --profile s3user
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/b5f53b07-03e2-4ae1-aba7-f9eb2d7d7ed8">
</div>

<p align="center"><i><b>List S3 Buckets Again</b></i></p>

5. By default the bucket should be configured to be private, meaning it should not be accessible from a public, unauthenticated perspective. To check that you can attempt to list files in the bucket without specifying any credentials (`--no-sign-request`).
> **_Command_**
```bash
sudo aws s3 ls s3://$BUCKET_NAME --no-sign-request
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/4fb1709d-6fe7-4221-a72d-77e6858fdd7f">
</div>

<p align="center"><i><b>Attempt to List S3 Bucket Anonymously</b></i></p>

6. Now upload a file to the S3 bucket from the AWS CLI. 

> **_Command_**
```bash
echo "Hello CloudBreacher! If all went according to plan this file is not publicly accessible." > testfile.txt
sudo aws s3 cp testfile.txt s3://$BUCKET_NAME/ --profile s3user
```
Running the following command once again you should see the test file you uploaded.
> **_Command_**
```bash
sudo aws s3 ls s3://$BUCKET_NAME --profile s3user
```

<div align="center">
  <img src="https://github.com/user-attachments/assets/78f63a2a-a54f-4cb4-aa83-9357f322a7b9">
</div>

<p align="center"><i><b>Successful File Upload to S3</b></i></p>

## Conclusion

In this lab, you got to experience the AWS command line. Additionally, you verified AWS policies were configured correctly to prevent certain user accounts from doing certain actions. You explored enumerating IAM users as well as deploying an S3 bucket.  

Find more AWS CLI commands here:
https://github.com/dafthack/CloudPentestCheatsheets/blob/master/cheatsheets/AWS.md
