---
title: Terraform IaC DevOps using AWS CodePipeline
description: Create AWS CodePipeline with Multiple Environments Dev and Staging
---
# IaC DevOps using AWS CodePipeline

## Step-00: Introduction
1. Terraform Backend with backend-config
2. How to create multiple environments related Pipeline with single TF Config files in Terraform ? 
3. As part of Multiple environments we are going to create `dev` and `stag` environments
4. We are going build IaC DevOps Pipelines using 
- AWS CodeBuild
- AWS CodePipeline
- Github
5. We are going to streamline the `terraform-manifests` taken from `section-15` and streamline that to support Multiple environments.

[![Image](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-1.png "Terraform on AWS with IAC DevOps and SRE")](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-1.png)

[![Image](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-2.png "Terraform on AWS with IAC DevOps and SRE")](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-2.png)

[![Image](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-3.png "Terraform on AWS with IAC DevOps and SRE")](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-3.png)

[![Image](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-4.png "Terraform on AWS with IAC DevOps and SRE")](https://stacksimplify.com/course-images/terraform-aws-codepipeline-iac-devops-4.png)

## Step-01:
- Update `private-key\terraform-key.pem` with your private key with same name.

### Step-02-04: Create S3 Bucket related folders for both environments for Terraform State Storage
- Go to Services -> S3 -> terraform-on-aws-for-ec2-toantruyen
- Create Folder `iacdevops`
- Create Folder `iacdevops\dev`
- Create Folder `iacdevops\stag`

### Step-02-05: Create DynamoDB Tables for Both Environments for Terraform State Locking 
- Create Dynamo DB Table for Dev Environment
  - **Table Name:** iacdevops-dev-tfstate
  - **Partition key (Primary Key):** LockID (Type as String)
  - **Table settings:** Use default settings (checked)
  - Click on **Create**
- Create Dynamo DB Table for Staging Environment
  - **Table Name:** iacdevops-stag-tfstate
  - **Partition key (Primary Key):** LockID (Type as String)
  - **Table settings:** Use default settings (checked)
  - Click on **Create**  

## Step-08: Create Secure Parameters in Parameter Store
### Step-08-01: Create MY_AWS_SECRET_ACCESS_KEY
- Go to Services -> Systems Manager -> Application Management -> Parameter Store -> Create Parameter 
  - Name: /CodeBuild/MY_AWS_ACCESS_KEY_ID
  - Descritpion: My AWS Access Key ID for Terraform CodePipeline Project
  - Tier: Standard
  - Type: Secure String
  - Rest all defaults
  - Value: ABCXXXXDEFXXXXGHXXX

### Step-08-02: Create MY_AWS_SECRET_ACCESS_KEY
- Go to Services -> Systems Manager -> Application Management -> Parameter Store -> Create Parameter 
  - Name: /CodeBuild/MY_AWS_SECRET_ACCESS_KEY
  - Descritpion: My AWS Secret Access Key for Terraform CodePipeline Project
  - Tier: Standard
  - Type: Secure String
  - Rest all defaults
  - Value: abcdefxjkdklsa55dsjlkdjsakj

## Step-12: Verify if AWS Connector for GitHub already installed on your Github
- Go to below url and verify
- **URL:** https://github.com/settings/installations

## Step-13: Create Github Connection from AWS Developer Tools
- Go to Services -> CodePipeline -> Create Pipeline
- In Developer Tools -> Click on **Settings** -> Connections -> Create Connection
- **Select Provider:** Github
- **Connection Name:** terraform-iacdevops-aws-cp-con1
- Click on **Connect to Github**
- GitHub Apps: Click on **Install new app**
- It should redirect to github page `Install AWS Connector for GitHub`
- **Only select repositories:** terraform-iacdevops-with-aws-codepipeline
- Click on **Install**
- Click on **Connect**
- Verify Connection Status: It should be in **Available** state
- Go to below url and verify
- **URL:** https://github.com/settings/installations
- You should see `Install AWS Connector for GitHub` app installed

## Step-14: Create AWS CodePipeline
- Go to Services -> CodePipeline -> Create Pipeline
### Pipeline settings
- **Pipeline Name:** tf-iacdevops-aws-cp1
- **Service role:** New Service Role
- rest all defaults
  - Artifact store: Default Location
  - Encryption Key: Default AWS Managed Key
- Click **Next**   
### Source Stage
- **Source Provider:** Github (Version 2)
- **Connection:** terraform-iacdevops-aws-cp-con1
- **Repository name:** terraform-iacdevops-with-aws-codepipeline
- **Branch name:** main
- **Change detection options:** leave to defaults as checked
- **Output artifact format:** leave to defaults as `CodePipeline default`
### Add Build Stage
- **Build Provider:** AWS CodeBuild
- **Region:** N.Virginia
- **Project Name:** Click on **Create Project**
  - **Project Name:** codebuild-tf-iacdevops-aws-cp1
  - **Description:** CodeBuild Project for Dev Stage of IAC DevOps Terraform Demo
  - **Environment image:** Managed Image
  - **Operating System:** Amazon Linux 2
  - **Runtimes:** Standard
  - **Image:** latest available today (aws/codebuild/amazonlinux2-x86_64-standard:3.0)
  - **Environment Type:** Linux
  - **Service Role:** New (leave to defaults including Role Name)
  - **Build specifications:** use a buildspec file
  - **Buildspec name - optional:** buildspec-dev.yml  (Ensure that this file is present in root folder of your github repository)
  - Rest all leave to defaults
  - Click on **Continue to CodePipeline**
- **Project Name:** This value should be auto-populated with `codebuild-tf-iacdevops-aws-cp1`
- **Build Type:** Single Build
- Click **Next**
### Add Deploy Stage
- Click on **Skip Deploy Stage**
### Review Stage
- Click on **Create Pipeline**


## Step-15: Verify the Pipeline created
- **Verify Source Stage:** Should pass
- **Verify Build Stage:** should fail with error 
- Verify Build Stage logs by clicking on **details** in pipeline screen
```log
[Container] 2021/05/11 06:24:06 Waiting for agent ping
[Container] 2021/05/11 06:24:09 Waiting for DOWNLOAD_SOURCE
[Container] 2021/05/11 06:24:09 Phase is DOWNLOAD_SOURCE
[Container] 2021/05/11 06:24:09 CODEBUILD_SRC_DIR=/codebuild/output/src851708532/src
[Container] 2021/05/11 06:24:09 YAML location is /codebuild/output/src851708532/src/buildspec-dev.yml
[Container] 2021/05/11 06:24:09 Processing environment variables
[Container] 2021/05/11 06:24:09 Decrypting parameter store environment variables
[Container] 2021/05/11 06:24:09 Phase complete: DOWNLOAD_SOURCE State: FAILED
[Container] 2021/05/11 06:24:09 Phase context status code: Decrypted Variables Error Message: AccessDeniedException: User: arn:aws:sts::180789647333:assumed-role/codebuild-codebuild-tf-iacdevops-aws-cp1-service-role/AWSCodeBuild-97595edc-1db1-4070-97a0-71fa862f0993 is not authorized to perform: ssm:GetParameters on resource: arn:aws:ssm:ap-southeast-2:180789647333:parameter/CodeBuild/MY_AWS_ACCESS_KEY_ID
```
## Step-16: Fix ssm:GetParameters IAM Role issues
### Step-16-01: Get IAM Service Role used by CodeBuild Project
- Get the IAM Service Role name CodeBuild Project is using
- Go to CodeBuild -> codebuild-tf-iacdevops-aws-cp1 -> Edit -> Environment
- Make a note of Service Role ARN
```t
# CodeBuild Service Role ARN 
arn:aws:iam::180789647333:role/service-role/codebuild-codebuild-tf-iacdevops-aws-cp1-service-role
```
### Step-16-02: Create IAM Policy with Systems Manager Get Parameter Read Permission
- Go to Services -> IAM -> Policies -> Create Policy
- **Service:** Systems Manager
- **Actions:** Get Parameters (Under Read)
- **Resources:** All
- Click **Next Tags**
- Click **Next Review**
- **Policy name:** systems-manger-get-parameter-access
- **Policy Description:** Read Parameters from Parameter Store in AWS Systems Manager Service
- Click on **Create Policy**

### Step-16-03: Associate this Policy to IAM Role
- Go to Services -> IAM -> Roles -> Search for `codebuild-codebuild-tf-iacdevops-aws-cp1-service-role`
- Attach the polic named `systems-manger-get-parameter-access`

## Step-17: Re-run the CodePipeline 
- Go to Services -> CodePipeline -> tf-iacdevops-aws-cp1
- Click on **Release Change**
- **Verify Source Stage:** 
  - Should pass
- **Verify Build Stage:** 
  - Verify Build Stage logs by clicking on **details** in pipeline screen
  - Verify `Cloudwatch -> Log Groups` logs too (Logs saved in CloudWatch for additional reference)


## Step-18: Verify Resources
0. Confirm SNS Subscription in your email
1. Verify EC2 Instances
2. Verify Launch Templates (High Level)
3. Verify Autoscaling Group (High Level)
4. Verify Load Balancer
5. Verify Load Balancer Target Group - Health Checks
7. Access and Test
```t
# Access and Test
http://devdemo1.truyenminhtoan.online
http://devdemo1.truyenminhtoan.online/app1/index.html
http://devdemo1.truyenminhtoan.online/app1/metadata.html
```

## Step-19: Add Approval Stage before deploying to staging environment
- Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 -> Edit
### Add Stage
  - Name: Email-Approval
### Add Action Group
- Action Name: Email-Approval
- Action Provider: Manual Approval
- SNS Topic: Select SNS Topic from drop down
- Comments: Approve to deploy to staging environment

## Step-20: Add Staging Environment Deploy Stage
- Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 -> Edit
### Add Stage
  - Name: Stage-Deploy
### Add Action Group
- Action Name: Stage-Deploy
- Region: US East (N.Virginia)
- Action Provider: AWS CodeBuild
- Input Artifacts: Source Artifact
- **Project Name:** Click on **Create Project**
  - **Project Name:** stage-deploy-tf-iacdevops-aws-cp1
  - **Description:** CodeBuild Project for Staging Environment of IAC DevOps Terraform Demo
  - **Environment image:** Managed Image
  - **Operating System:** Amazon Linux 2
  - **Runtimes:** Standard
  - **Image:** latest available today (aws/codebuild/amazonlinux2-x86_64-standard:3.0)
  - **Environment Type:** Linux
  - **Service Role:** New (leave to defaults including Role Name)
  - **Build specifications:** use a buildspec file
  - **Buildspec name - optional:** buildspec-stag.yml  (Ensure that this file is present in root folder of your github repository)
  - Rest all leave to defaults
  - Click on **Continue to CodePipeline**
- **Project Name:** This value should be auto-populated with `stage-deploy-tf-iacdevops-aws-cp1`
- **Build Type:** Single Build
- Click on **Done**
- Click on **Save**

## Step-21: Update the IAM Role
- Update the IAM Role created as part of this `stage-deploy-tf-iacdevops-aws-cp1` CodeBuild project by adding the policy `systems-manger-get-parameter-access1`

## Step-22: Run the Pipeline 
- Go to Services -> AWS CodePipeline -> tf-iacdevops-aws-cp1 
- Click on **Release Change**
- Verify Source Stage
- Verify Build Stage (Dev Environment - Dev Depploy phase)
- Verify Manual Approval Stage - Approve the change
- Verify Stage Deploy Stage
  - Verify build logs

## Step-23: Verify Staging Environment
0. Confirm SNS Subscription in your email
1. Verify EC2 Instances
2. Verify Launch Templates (High Level)
3. Verify Autoscaling Group (High Level)
4. Verify Load Balancer
5. Verify Load Balancer Target Group - Health Checks
7. Access and Test
```t
# Access and Test
http://stagedemo1.truyenminhtoan.online
http://stagedemo1.truyenminhtoan.online/app1/index.html
http://stagedemo1.truyenminhtoan.online/app1/metadata.html
```
 
## Step-24: Make a change and test the entire pipeline
### Step-24-01: c13-03-autoscaling-resource.tf
- Increase minimum EC2 Instances from 2 to 3
```t
# Before
  desired_capacity = 2
  max_size = 10
  min_size = 2
# After
  desired_capacity = 4
  max_size = 10
  min_size = 4
```
### Step-24-02: Commit Changes via Git Repo
```t
# Verify Changes
git status

# Commit Changes to Local Repository
git add .
git commit -am "ASG Min Size from 2 to 4"

# Push changes to Remote Repository
git push
```
### Step-24-03: Review Build Logs
- Go to Services -> CodePipeline -> tf-iacdevops-aws-cp1
- Verify Dev Deploy Logs
- Approve at `Manual Approval` stage
- Verify Stage Deploy Logs

### Step-24-04: Verify EC2 Instances
- Go to Services -> EC2 Instances
- Newly created instances should be visible.
- hr-dev: 4 EC2 Instances
- hr-stag: 4 EC2 Instances

## Step-25: Destroy Resources
### Step-25-01: Update buildspec-dev.yml
```t
# Before
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"
# After
    #TF_COMMAND: "apply"
    TF_COMMAND: "destroy"    
```
### Step-25-02: Update buildspec-stag.yml
```t
# Before
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"
# After
    #TF_COMMAND: "apply"
    TF_COMMAND: "destroy"    
```
### Step-25-03: Commit Changes via Git Repo
```t
# Verify Changes
git status

# Commit Changes to Local Repository
git add .
git commit -am "Destroy Resources"

# Push changes to Remote Repository
git push
```
### Step-25-03: Review Build Logs
- Go to Services -> CodePipeline -> tf-iacdevops-aws-cp1
- Verify Dev Deploy Logs
- Approve at `Manual Approval` stage
- Verify Stage Deploy Logs


## Step-26: Change Everything back to original Demo State
### Step-26-01: c13-03-autoscaling-resource.tf
- Change them back to original state
```t
# Before
  desired_capacity = 4
  max_size = 10
  min_size = 4
# After
  desired_capacity = 2
  max_size = 10
  min_size = 2
```
### Step-26-02: buildspec-dev.yml and buildspec-stag.yml
- Change them back to original state
```t
# Before
    #TF_COMMAND: "apply"
    TF_COMMAND: "destroy"   
# After
    TF_COMMAND: "apply"
    #TF_COMMAND: "destroy"     
```
### Step-26-03: Commit Changes via Git Repo
```t
# Verify Changes
git status

# Commit Changes to Local Repository
git add .
git commit -am "Fixed all the changes back to demo state"

# Push changes to Remote Repository
git push
```


## References
- [1:Backend configuration Dynamic](https://www.terraform.io/docs/cli/commands/init.html)
- [2:Backend configuration Dynamic](https://www.terraform.io/docs/language/settings/backends/configuration.html#partial-configuration)
- [AWS CodeBuild Builspe file reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html#build-spec.env)
