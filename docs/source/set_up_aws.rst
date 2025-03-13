=====================================
Setup and Configure AWS Batch
=====================================

The Nextflow workflows can use AWS Batch as a compute provider. This document describes how to setup and configure AWS Batch to run these Nextflow workflows.

AWS Batch is a managed service that allows you to run batch computing workloads on the AWS Cloud. AWS Batch automatically provisions compute resources and optimizes the workload distribution based on the quantity and scale of the workloads. AWS Batch plans, schedules, and executes the Nextflow workflows across the full range of AWS compute services and features, such as Amazon EC2 and Spot Instances.

.. important:: These instructions assume that you are familiar and comfortable using and managing AWS Accounts and Services. If you are not familiar with AWS, we recommend that you contact us to assist with the setup and configuration of AWS Batch.


Before you begin
================
Before you begin, you will need the following to be configured and available:

* An AWS account: https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html#sign-up-for-aws
* An AWS Identity and Access Management (IAM) user with administrator permissions: https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html#create-an-admin
* Choose the Region where you want to run the Nextflow workflows *(We recommend choosing a region geographically closest to you to reduce latency and costs)*.
* Optional: Install the AWS Command Line Interface (AWS CLI): https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html#install-aws-cli


Setup the Required AWS Services and resources
=============================================
The following AWS services and resources are required to run Nextflow workflows on AWS Batch.

Create a key pair
-----------------
AWS uses public-key cryptography to secure the login information for your instance. A Linux instance, such as an AWS Batch compute environment container instance, has no password to use for SSH access. You use a key pair to log in to your instance securely. You specify the name of the key pair when you create your compute environment, then provide the private key when you log in using SSH

To create the key pair, follow the instructions at https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html#create-a-key-pair.

Create a VPC
------------
With Amazon Virtual Private Cloud (Amazon VPC), you can launch AWS resources into a virtual network that you've defined.

If you have an existing VPC, you can use that and skip to the next step. Otherwise, follow the instructions at https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html#create-a-vpc to create a VPC.



Create a security group
-----------------------
Security groups act as a firewall for associated compute environment container instances, controlling both inbound and outbound traffic at the container instance level. A security group can be used only in the VPC for which it is created

Follow the instructions at https://docs.aws.amazon.com/batch/latest/userguide/get-set-up-for-aws-batch.html#create-a-security-group to create a security group(s) required for AWS Batch.


Create an Custom AMI
=====================================
A custom AMI is required to run Nextflow workflows on AWS Batch. The custom AMI is based on the Amazon ECS-optimized Amazon Linux 2 AMI. The custom AMI requires the AWScli to be installed. To create the custom AMI, you will need to

1. Create an EC2 instance using the latest version of the "Amazon ECS-optimized Amazon Linux 2" AMI which you be found in the AWS Marketplace.
2. Connect to the EC2 instance using SSH
3. install the AWScli, and then create an AMI from the EC2 instance.

    .. code-block:: bash

        sudo yum update
        sudo yum install unzip
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install


Create IAM role for compute instances
=====================================
AWS Batch compute environments are populated with Amazon ECS container instances. They run the Amazon ECS container agent locally. The Amazon ECS container agent makes calls to various AWS API operations on your behalf. Therefore, container instances that run the agent require an IAM policy and role for these services to recognize that the agent belongs to you.

Follow the instructions at https://docs.aws.amazon.com/batch/latest/userguide/instance_IAM_role.html check if the role already exists. If it does not, then follow the instructions to create the IAM role. Call the IAM role `ecsInstanceRole`.


Create IAM role for Spot instances
====================================
MacCoss Lab used Amazon EC2 Spot Instances to run the Nextflow workflows. This reduces the cost by 50-70%. To use Spot Instances, you must create an IAM role that allows AWS Batch to launch Spot Instances on your behalf.

Follow the instructions at https://docs.aws.amazon.com/batch/latest/userguide/spot_fleet_IAM_role.html to create the IAM role.


Create the Compute Environment and the Job Queue to run Nextflow workflows
==========================================================================
We will create a new compute environment which uses Amazon Elastic Compute Cloud(Amazon EC2) instances. The compute environment is where the Nextflow workflows will run.

To create the compute environment, see *Create a compute environment* in https://docs.aws.amazon.com/batch/latest/userguide/getting-started-ec2.html in the AWS Batch User Guide. Refer to the following table to determine what options to select for creating the Compute Environment.

=====================================  ============
Option                                 Value
=====================================  ============
Instance Role                          Use the instance role, you created, named `ecsInstanceRole`
Add a Tag                              We recommend creating at least 1 Tag to help track the usage. For example, use Key='Project', Value='Nextflow'
Use EC2 Spot instances                 Enable
Spot Fleet Role                        Use the Spot Fleet role you created above
Minimum vCPUS                          Use `0`. This will allow AWS Batch to scale the compute environment to 0 instances when there are no jobs to run. This will help reduce costs.
Desired vCPUs                          Use same value as the Minimum vCPUs
Maximum vCPUS                          MacCoss Lab used `640`. This will allow AWS Batch to scale the compute environment to use up to 640 vCPUs when there are jobs to run
Allowed instance types                 MacCoss Lab used the following instance types: `"r6a.large", "r6a.xlarge", "c6a.8xlarge", "r6a.4xlarge"`
EC2 key pair                           Use the key pair you created above
Allocation strategy                    Use `BEST_FIT_PROGRESSIVE`
EC2 configuration - Image type         Use `ID` for the AMI you created above
EC2 configuration - Image ID override  Use `Amazon Linux 2` also called `ECS_AL2`
Network - VPC Configuration            Use the VPC you created above
Network - Security groups              Use the Security group you created above
=====================================  ============

On the "Create a job queue" page, refer to the following table to determine what options to select for creating the Job Queue.

Job Queue settings
=====================================  ============
Option                                 Value
=====================================  ============
Name                                   Use the same name as the compute environment
Priority                               MacCoss Lab used `50`
=====================================  ============

On the Create a Job Definition page, select all the defaults and click `Next`.

On the Create a Job page, select all the defaults and click `Next`.

Click on the `Create` button to create the compute environment and the job queue.

**Note**: The compute environment will take a few minutes to be created. You can check the status of the compute environment in the AWS Batch console. The compute environment is ready when the status is `VALID`.



Enable CloudWatch Logs with AWS Batch
=====================================
You can configure your AWS Batch jobs (ie Nextflow workflows) to send detailed log information and metrics to CloudWatch Logs. Doing this, you can view different logs from your jobs in one convenient location.

Add a CloudWatch Logs IAM policy
--------------------------------
Follow the instructions at https://docs.aws.amazon.com/batch/latest/userguide/using_cloudwatch_logs.html#cwl_iam_policy to add a CloudWatch Logs IAM policy.

In the instructions, you will be asked to add the new policy to the IAM role used by AWS Batch. This is the IAM role created in the *IAM role for compute instances* section above.

Once you have created the IAM policy you should also go to CloudWatch and create a Log Group that will be needed for the configuration of the workflow run.

Make S3 bucket
==============
Create a new S3 bucket to store the Nextflow workflow files and results. To create the S3 bucket, see *Creating a bucket* in https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html. Refer to the following table to determine what options to select.

=====================================  ============
Option                                 Value
=====================================  ============
Bucket name                            Specify the name of the bucket. For example, `nextflow-dia` or `<your-lab-name>-nextflow-dia`
Region                                 Use the same region as the VPC, created above
Object Ownership                       Choose the default setting of "Bucket owner enforced"
Block Public Access                    Keep the default settings. Public access is not required for Nextflow workflows
Default encryption                     Enable and use `Amazon S3 managed key (SSE-S3)`
Tags                                    We recommend creating at least 1 Tag to help track the usage. For example, use Key='Project', Value='Nextflow'
=====================================  ============


IAM Policy to enable read/write access to the S3 bucket
-------------------------------------------------------
Create a new IAM policy to allow read/write access to the S3 bucket. This policy will be used by the AWS Batch IAM roles and by the IAM users submitting Nextflow workflows. This policy also includes permissions to read secret values from AWS Secrets Manager.

An example policy is below

.. code::

    {
        "Statement": [
            {
                "Action": [
                    "s3:ListAllMyBuckets"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:s3:::*"
                ]
            },
            {
                "Action": [
                    "s3:ListBucket",
                    "s3:GetBucketLocation",
                    "s3:GetBucketACL",
                    "s3:ListBucketMultipartUploads"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:s3:::<bucket-name>"
                ]
            },
            {
                "Action": [
                    "s3:PutObject",
                    "s3:PutObjectAcl",
                    "s3:GetObject",
                    "s3:GetObjectAcl",
                    "s3:DeleteObject",
                    "s3:AbortMultipartUpload",
                    "s3:ListMultipartUploadParts"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:s3:::<bucket-name>",
                    "arn:aws:s3:::<bucket-name>/*"
                ]
            }
        ],
        "Version": "2012-10-17"
    }

- where `<bucket-name>` is the name of the S3 bucket you created above.

Add the new policy to the IAM role used by AWS Batch. This is the IAM role created in the *IAM role for compute instances* section above.


Enable using secrets in AWS Secret Manager
==========================================
The Nextflow workflows may use the AWS Secrets Manager to store and retrieve secrets. To enable this, you will need to create two IAM policies. These will allow for the creation/management of secrets and for reading the secret value.

Create IAM Policy to enable reading the secrets
-----------------------------------------------
Create a new IAM policy to allow the AWS Batch instances to read the secrets created when running the Nextflow workflows.

An example policy is below

.. code::

    {
        "Statement": [
            {
                "Effect":"Allow",
                "Action": "secretsmanager:GetSecretValue",
                "Resource": "arn:aws:secretsmanager:<aws-region>:<aws-account>:secret:NF_*"
            }
        ],
        "Version": "2012-10-17"
    }

where

- `<aws-region>` is the name of the aws-region where your AWS resources are located
- `<aws-account>` is the AWS account number

Add the new policy to the IAM role used by AWS Batch. This is the IAM role created in the *IAM role for compute instances* section above.


Create IAM Policy to enable creating and managing the secrets
-------------------------------------------------------------
Create a new IAM policy to allow IAM users to create and manage secrets in AWS Secrets Manager.

An example policy is below

.. code::

    {
        "Statement": [
            {
                "Sid": "EnableAccessToSecrets",
                "Effect": "Allow",
                "Action": [
                    "secretsmanager:GetSecretValue",
                    "secretsmanager:CreateSecret",
                    "secretsmanager:DeleteSecret",
                    "secretsmanager:DescribeSecret",
                    "secretsmanager:PutSecretValue",
                    "secretsmanager:UpdateSecret"
                ],
                "Resource": "arn:aws:secretsmanager:<aws-region>:<aws-account>:secret:NF_${aws:username}_*"
            },
            {
                "Sid": "EnableListingSecrets",
                "Effect": "Allow",
                "Action": [
                    "secretsmanager:ListSecrets"
                ],
                "Resource": "*"
            }
        ],
        "Version": "2012-10-17"
    }

where

- `<aws-region>` is the name of the aws-region where your AWS resources are located
- `<aws-account>` is the AWS account number


Create IAM Users for users submitting Nextflow workflows
========================================================

Create IAM Policy to enable running AWS Batch jobs
--------------------------------------------------
Create a new IAM policy to allow IAM users to submit AWS Batch jobs.

An example policy is below:

.. code::

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "batch:DescribeJobQueues",
                    "batch:CancelJob",
                    "batch:SubmitJob",
                    "batch:ListJobs",
                    "batch:DescribeComputeEnvironments",
                    "batch:TerminateJob",
                    "batch:DescribeJobs",
                    "batch:RegisterJobDefinition",
                    "batch:DescribeJobDefinitions",
                    "batch:TagResource"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ecs:DescribeTasks",
                    "ec2:DescribeInstances",
                    "ec2:DescribeInstanceTypes",
                    "ec2:DescribeInstanceAttribute",
                    "ecs:DescribeContainerInstances",
                    "ec2:DescribeInstanceStatus"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ecr:GetAuthorizationToken",
                    "ecr:BatchCheckLayerAvailability",
                    "ecr:GetDownloadUrlForLayer",
                    "ecr:GetRepositoryPolicy",
                    "ecr:DescribeRepositories",
                    "ecr:ListImages",
                    "ecr:DescribeImages",
                    "ecr:BatchGetImage",
                    "ecr:GetLifecyclePolicy",
                    "ecr:GetLifecyclePolicyPreview",
                    "ecr:ListTagsForResource",
                    "ecr:DescribeImageScanFindings",
                    "logs:GetLogEvents"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }


Create IAM Users
----------------
Create a new IAM user for each user who will be submitting Nextflow workflows. If the users already have an IAM user, you can use that IAM user and skip the instructions for creating the user. Ideally the IAM user should have both programmatic and console access.

Follow the instructions to create an IAM user at https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html

When creating the IAM user, you will be asked to add permissions:

- Add the IAM policy created in the *Create IAM Policy to enable running AWS Batch jobs* section above
- Add the IAM policy created in the *IAM Policy to enable read/write access to the S3 bucket* section above
- Add the IAM policy created in the *Create IAM Policy to enable creating and managing the secrets* section above

Next steps
==========
Set up of AWS Batch is complete. Next see our :doc:`configure_workflow` document for how to configure and run the workflow using AWS Batch.
