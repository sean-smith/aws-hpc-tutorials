+++
title = "- Build an IAM Policy with Cloudformation"
date = 2019-09-18T10:46:30-04:00
weight = 57
tags = ["tutorial", "IAM", "ParallelCluster", "Serverless"]
+++

Now you will create your custom IAM policy. To facilitate its creation, a CloudFormation template has been prepared for download and you will examine the different permissions that will be granted to your HPC cluster.

1. Download the CloudFormation template containing the custom IAM policy from the S3 bucket **aws-hpc-workshops** using the command `aws s3` as follows:
    ```bash
    aws s3 cp s3://aws-hpc-workshops/serverless-template.yaml .
    ```
    For reference, the **serverless-template.yaml** file content is provided below. It contains the CloudFormation template that will create the IAM policy which allows cluster instances to connect to the SSM endpoints and access to the S3 bucket created in the previous steps.


    {{%expand "See the content of serverless-template.yaml (click to expand)" %}}
    AWSTemplateFormatVersion: 2010-09-09
    Description: This template deploys the ParallelCluster additional policies required to use SSM.
    Parameters:
      S3Bucket:
        Description: Choose an existing S3 bucket used to store the input/output data from jobs and save the    output of SSM execution commands.
        Type: String
    Resources:
      pclusterSSM:
        Type: 'AWS::IAM::ManagedPolicy'
        Properties:
          ManagedPolicyName: pclusterSSM
          Path: /
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - 'ssm:DescribeAssociation'
                  - 'ssm:GetDeployablePatchSnapshotForInstance'
                  - 'ssm:GetDocument'
                  - 'ssm:DescribeDocument'
                  - 'ssm:GetManifest'
                  - 'ssm:GetParameter'
                  - 'ssm:GetParameters'
                  - 'ssm:ListAssociations'
                  - 'ssm:ListInstanceAssociations'
                  - 'ssm:PutInventory'
                  - 'ssm:PutComplianceItems'
                  - 'ssm:PutConfigurePackageResult'
                  - 'ssm:UpdateAssociationStatus'
                  - 'ssm:UpdateInstanceAssociationStatus'
                  - 'ssm:UpdateInstanceInformation'
                  - 'ssmmessages:CreateControlChannel'
                  - 'ssmmessages:CreateDataChannel'
                  - 'ssmmessages:OpenControlChannel'
                  - 'ssmmessages:OpenDataChannel'
                  - 'ec2messages:AcknowledgeMessage'
                  - 'ec2messages:DeleteMessage'
                  - 'ec2messages:FailMessage'
                  - 'ec2messages:GetEndpoint'
                  - 'ec2messages:GetMessages'
                  - 'ec2messages:SendReply'
                Resource: '*'
              - Effect: Allow
                Action:
                  - 's3:*'
                Resource: !Sub 'arn:aws:s3:::${S3Bucket}/*'
    Outputs:
      PclusterPolicy:
        Description: PclusterPolicy
        Value: !Sub ${pclusterSSM}
      S3Bucket:
        Description: S3 Bucket name
        Value: !Sub ${S3Bucket}
    {{% /expand%}}

2. Deploy the CloudFormation template and create the IAM policy
    {{% notice info %}}
The command above creates a CloudFormation stack as based on the template `serverless-template.yaml`. The policy name is specified in the template file. The `--stack-name` argument takes a unique name that will be associated with the stack on your account. The `--parameters` option specify the input parameters for the stack (here we pass `S3Bucket` as the key and name of the S3 Bucket as the Value). Furthermore, since you are creating IAM resources you must explicitly acknowledge that your stack template contains `--capabilities`. You could use the *AWS Console* to create this stack too.
{{% /notice %}}

    ```bash
    aws cloudformation create-stack --stack-name pc-serverless-policy --parameters ParameterKey=S3Bucket,ParameterValue=serverless-${BUCKET_POSTFIX} --template-body file://serverless-template.yaml --capabilities CAPABILITY_NAMED_IAM
    ```
    {{% notice note %}}
The IAM policy enables access to the S3 bucket created to store the job data and SSM commands. This is provided as `ParameterValue`. Make sure to provide the correct S3 bucket name
{{% /notice %}}

3. Once the AWS CloudFormation stack creation completes, you can confirm the IAM Policy is created by running the command below in your Cloud9 Terminal to query the policy `pclusterSSM` you just created

   ```bash
   aws iam list-policies --query 'Policies[?PolicyName==`pclusterSSM`]'
   ```

   The expected command line output should look like:
![Lambda Basic Settings](/images/serverless/iam-policy-result.png)

In the next section, you will change your AWS ParallelCluster configuration file to add the `pclusterSSM` IAM policy and will apply the modified configuration to your cluster.


<!-- {{% notice tip %}}
You don't actually need to download the Cloudformation template to your Cloud9 instance. Indeed, it is possible to run Cloudformation templates that are already stored on S3. For that matter you could replace the `--template-file file://serverless-template.yaml` by the argument `--template-url https://aws-hpc-workshops.s3.amazonaws.com/serverless-template.yaml` and it will work as well.
{{% /notice %}} -->