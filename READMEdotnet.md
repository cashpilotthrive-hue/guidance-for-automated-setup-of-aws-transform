# Guidance for Automated Setup of AWS Transform for .NET

## Table of Contents

1. [Overview](#overview)
   - [Architecture](#architecture)
   - [Cost](#cost)
2. [Prerequisites](#prerequisites)
   - [AWS Account](#aws-account)
   - [Supported Regions](#supported-regions)
   - [Software](#software)
3. [Security](#security)
4. [Deployment Steps](#deployment-steps)
5. [Deployment Validation](#deployment-validation)
6. [Running the Guidance](#running-the-guidance)
7. [Troubleshooting](#troubleshooting)
8. [FAQ, known issues, additional considerations, and limitations](#faq-known-issues-additional-considerations-and-limitations)
   - [Service Quotas](#service-quotas)
9. [Cleanup](#cleanup)
10. [Authors](#authors)

## Overview

This Guidance provides an automated approach to deploying AWS Transform for .NET resources using Infrastructure as Code (IaC). It streamlines the setup process by automating the provisioning of required AWS services and security controls. The guidance accelerates time-to-value for organizations modernizing .NET applications while ensuring adherence to AWS best practices and security standards.

### Capabilities and Key Features

AWS Transform for .NET is a service designed to accelerate the modernization of Windows-based .NET Framework applications to cross-platform .NET for Linux environments. It leverages generative AI to automate and streamline the process of porting applications, aiming to reduce the time and effort typically involved in such migrations. AWS Transform for .NET provides:

- Analysis of .NET Framework codebases from source control systems, including private NuGet support
- Automated transformation of legacy .NET Framework applications to cross-platform .NET
- Integration with source control platforms (BitBucket, GitHub, and GitLab)
- Validation of transformed code through unit tests

.NET modernization can be initiated from either the Visual Studio IDE or the AWS Transform Web Experience.
The process and underlying systems are the same in either modality.
If initiated from the AWS Transform web experience the developer is required to select an account (that they
have access to) and from the account the existing Amazon Code Connections to connect to GitHub, GitLab or
Bitbucket. Once connected, they can select a repository for transformation. Once selected, that repo is
uploaded to the transformation system.
In the IDE developers can transform .NET projects using the AWS Plugin for Visual Studio.

### Benefits

- Accelerate .NET application porting from Windows to Linux up to 4 times faster
- AWS Transform accelerates large-scale .NET modernization by up to 4x. By using an autonomous AI-powered .NET transformation agent under human supervision, modernization teams can collaboratively execute larger and more complex projects with consistency, reduce operating costs by up to 40%, and enhance quality and performance using Generative AI.
- As an innovator in this space, AWS Transform for .NET is still the only generally available agent-based upgrade tool for .NET. Our tools gives users the power of an intelligent LLM along with the structure of purpose-built prompts, ensuring an efficient and reliable upgrade process.
- AWS Transform is also designed to meet our customers where they are at. When working with technical decision makers, you can emphasize the unique web-based experience for transforming apps at scale. And when speaking with developers, you can highlight the IDE integration for diving deep into the details of a complex upgrade.
- .NET Linux workloads are eligible to run on Graviton and can see up to 60% better price/performance versus Windows
- Lastly, AWS has years of experience and strong incentives when it comes to helping customers modernize apps and break free from expensive and restrictive licenses. Be sure to highlight our programs and tools in this area, including App Modernization Labs and Experience-Based Acceleration, both of which are aimed at helping customers move seamlessly towards modernization and digital transformation.

### Supported Versions and Project Types

AWS Transform supports transformation for:

- .NET Framework 3.5+
- .NET Core 3.1
- .NET 5.x+
- .NET 8.0

Supported project types include:

- Libraries
- Console applications
- Web API (ASP.NET)
- MVC applications including Razor views
- WCF services
- Unit test projects

### Architecture

You can modernize your .NET code by using either the AWS Transform web application or the AWS Toolkit for Visual Studio.

Below is the Reference architecture for the guidance showing the core and supporting AWS services:

![Figure 1: Standard .NET Transform process](assets/aws_transform_dotnet_ref-arch1.png)

**Figure 1. Automated Setup of AWS Transform for .NET - Standard .NET Transform process**

1. The user authenticates through the AWS Identity and Access Management (AWS IAM) Identity Center.
2. The user selects a solution or project to transform. AWS Transform for .NET builds code locally to verify if it is buildable and configured correctly for transformation.
3. The Specialized Agent in AWS Transform for .NET requests a unique transformation job ID, which creates an association at the AWS Transform for .NET service account securing the job ID to the user who requested the transformation.
4. AWS Transform for .NET then uploads code to a Amazon Simple Storage Service (Amazon S3) bucket. The bucket is sorted by account ID and job ID. When a job reads from the bucket, access is limited to the code relevant to that job. Code from other jobs, even for the same customer, remains inaccessible to the running job process.
5. Transformed code is saved in Amazon S3 under the same the job ID
6. Use the Amazon Q Developer extension in the Developer IDE to download the code directly from AWS Transform .Net.
7. AWS Transform's specialized agent analyzes incompatibilities, generates and replaces code to automatically port applications from outdated C# to Linux-compatible versions, upgrading .NET Framework to cross-platform .NET, and updating NuGet packages and APIs.

![Figure 2: Web Experience Specific](assets/aws_transform_dotnet_ref-arch2.png)

**Figure 2. Automated Setup of AWS Transform for .NET - Web Experience Specific**

1. For web portal users, AWS CodeConnections provides secure access to authorized source code repositories that AWS Transform for .NET can access.
2. The Amazon Elastic Compute Cloud (Amazon EC2) instance that hosts the Sandbox environment clones the repository and processes transformations in isolation, with one sandbox per job to prevent cross-contamination.
3. After completing the transformation, the changes are committed to the repository in a new branch.

![Figure 3: Supporting services](assets/aws_transform_dotnet_ref-arch3.png)

**Figure 3. Automated Setup of AWS Transform for .NET - Supporting services**

1. AWS Transform for .NET validates Amazon S3 bucket access by matching job ID with the initiating user's saved code. The service removes code from the Amazon S3 bucket twenty-four hours after job completion
2. AWS Transform for .NET deploys ephemeral agents for both web and IDE experiences, which perform the code transformation tasks and automatically terminate after job completion
3. AWS Transform for .NET processes selected repositories in isolated sandboxes, with one sandbox per job.

### Cost

While this implementation guide provides default configurations, customers are responsible for:

1. Configuring the guidance to their optimal settings based on their specific use case and requirements
2. Monitoring and managing the costs incurred from any supporting AWS services
3. Reviewing and optimizing storage usage for transformation artifacts

We recommend creating a [Budget](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) through [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this Guidance.

#### Sample Cost Table

As of April 2024, the cost for running this Guidance with the default settings in the default AWS Region (US East - N. Virginia) us-east-1 is minimal:

| AWS Service             | Dimensions                         | Cost [USD]              |
| ----------------------- | ---------------------------------- | ----------------------- |
| AWS Transform           | Unlimited transformations          | Free                    |
| AWS IAM Identity Center | Number of users                    | Free                    |
| AWS Organizations       | Number of accounts in organization | Free                    |
| AWS Lambda              | First 1M requests                  | Free                    |
| AWS Lambda              | Over 1M requests                   | $0.20 per 1M requests   |
| Amazon S3               | Standard Storage                   | $0.023 per GB per month |
| AWS CloudTrail          | Management events                  | First trail free        |
| AWS KMS                 | Customer managed key (optional)    | $1.00 per key per month |

Note: This cost estimate assumes typical usage patterns and does not include costs that might be incurred from extensive use of supporting services. Actual costs may vary based on your specific usage and configuration.

## Prerequisites

### AWS account requirements

An AWS Account with [IAM admin access](https://docs.aws.amazon.com/streams/latest/dev/setting-up.html) is required to run the scripts that will enable AWS organization and the Identity Center groups.

### Supported Regions

For the full list of supported regions view [AWS Transform Regions](https://docs.aws.amazon.com/transform/latest/userguide/regions.html).
Currently, the following regions are supported by default:

- US East (N. Virginia)
- Europe (Frankfurt)
- Asia Pacific (Mumbai)
- Asia Pacific (Sydney)
- Asia Pacific (Tokyo)
- Europe (London)
- Asia Pacific (Seoul)
- Canada (Central)

### Software

- The machine from which this guidance is deployed needs to support Linux BASH or PowerShell scripts. Alternatively, the parameters can be manually added to the CloudFormation YAML files.
- Source code repository (GitHub, GitLab, or Bitbucket)
- Visual Studio IDE (for IDE integration)

## Security

When you build systems on AWS infrastructure, security responsibilities are shared between you and AWS. This [shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/) reduces your operational burden because AWS operates, manages, and controls the components including the host operating system, the virtualization layer, and the physical security of the facilities in which the services operate. For more information about AWS security, visit [AWS Cloud Security](https://aws.amazon.com/security/).

Upon successful completion of both deployment phases, you will have established two IAM Identity Center permission sets and corresponding IDC groups with minimal baseline permissions derived from the official AWS Transform for .NET documentation. These permission sets provide only the essential access required for basic AWS Transform operations, following the principle of least privilege.

Organizations will need to customize and expand these permissions based on their specific .NET modernization requirements, target AWS services, and operational needs. The barebones permissions serve as a secure foundation that should be augmented with additional policies for services like AWS CodeConnections for source code repository access, Amazon S3 for transformation artifact storage, AWS KMS for encryption key management, AWS CloudWatch for monitoring and logging, and other AWS resources that will be utilized during the actual .NET application transformation process.

Users should refer to the [AWS Transform security documentation](https://docs.aws.amazon.com/transform/latest/userguide/security.html) to understand the full scope of permissions that may be required for their particular use cases and transformation scenarios. Key security considerations include source code security (secure access to repositories, encryption of code during transformation, protection of transformation artifacts), identity and access management (role-based access control through IAM Identity Center, least privilege access principles, separation of duties between administrators and users), and data protection (encryption at rest for transformation artifacts, secure transmission of code and artifacts, optional customer-managed KMS keys for additional control).

By adhering to these security principles and leveraging AWS's robust security features, organizations can ensure a secure .NET modernization process using AWS Transform.

## Deployment Steps

### Clone Guidance repository

1. Log in to your AWS account on your CLI/shell through your preferred authentication provider.
2. Clone the repository:

   ```bash
   git clone https://github.com/aws-solutions-library-samples/guidance-for-automated-setup-of-aws-transform
   ```

### Phase 1: Set up AWS Organizations

> Note : If you already have AWS Organizations enabled in your Management account, you can skip Phase 1.

1. Change directory to the source folder inside the guidance repository:

   ```bash
   cd guidance-for-automated-setup-of-aws-transform/source
   ```

2. Start by running the first shell script. This creates an AWS Organization with all features enabled.

   - STACK_NAME: {name of CloudFormation stack}.
   - TEMPLATE_PATH: {path to `phase2-idc.yaml`}.

   BASH:

   ```bash
   source % ./deploy-phase1.sh
   Enter stack name [aws-org-setup]: aws-org-setup
   Enter template path [/guidance-for-automated-setup-of-aws-transform/source/phase1-aws-organizations.yaml]:
   ```

   PowerShell:

   ```powershell
       PS C:\git\aws\guidance-for-automated-setup-of-aws-transform\source> .\deploy-phase1.ps1
       Enter stack name [aws-org-setup]:
       Enter template path [phase1-aws-organizations.yaml]:
   ```

   > Note : A Powershell script is available for Windows OS. Alternatively, the parameters can be manually added to the CloudFormation YAML.

3. After successful deployment, you will need to manually enable an organization instance of IAM Identity Center in the AWS Console (wait a few minutes for the changes to propagate), as shown below:

![Enable IAM Identity Center](assets/enable_identity_center.png)

**Figure 4. Enable an Organization instance of IAM Identity Center**

### Phase 2: Set up IAM Identity Center

1. After enabling IAM Identity Center manually and waiting for updates to propagate, run the second BASH script

   Pass in the following parameters using the bash script:

   - STACK_NAME: {name of cloudformation stack}.
   - TEMPLATE_PATH: {path to phase2 yaml}.
   - ACCOUNT_NUMBER: {AWS account number}.
   - IDENTITY_CENTER_ID: {AWS Identity Center ID}.
   - ADMIN_EMAIL: {Email for admin user provisioned by script}.

   BASH:

   ```bash
       source % ./deploy-phase2.sh
       Enter stack name [aws-transform-setup]:
       Enter template path: [/guidance-for-automated-setup-of-aws-transform/source/phase2-idc.yaml]:
       Enter AWS account number: 1234567XXXXXX
       Enter admin email address: admin@amazon.com
       Enter Identity Center ID: ssoins-1234a123b1d5ab3f
       Retrieving Identity Store ID for IAM Identity Center instance ssoins-1234a252c3d5bd2f...
       Found Identity Store ID: d-40338374bc
   ```

   PowerShell:

   ```powershell
       PS C:\git\aws\guidance-for-automated-setup-of-aws-transform\source> .\deploy-phase2.ps1
       Enter stack name [aws-transform-setup]:
       Enter template path [phase2-idc.yaml]:
       Enter AWS account number: 1234567XXXXXXX
       Enter admin email address: admin@amazon.com
       Enter Identity Center ID: ssoins-1234a123b1d5ab3f
       Retrieving Identity Store ID for IAM Identity Center instance ssoins-1234a252c3d5bd2f...
       Found Identity Store ID: d-40338374bc
   ```

   This script will:

   - Create IAM Identity Center groups and users
   - Set up the necessary IAM policies for AWS Transform for both groups
   - Create an Admin user using lambda functions in Identity Center based on a provided email

> Note : The script uses the deployed Lambda functions to add the provided email account as an Admin in the created AWS Transform Admin group in AWS IAM Identity Center. Subsequent admins and users can be added via the AWS console following best practices.

## Deployment Validation

- Open CloudFormation in AWS console and verify the status of the stacks

![CloudFormation Stack Status](assets/cfn_stack.png)

**Figure 5. Guidance Cloud Formation Stack Deployment Status**

- Open Identity Center and verify the created groups:

![Identity Center Groups](assets/idc_group.png)

**Figure 6. Verify Identity Center Groups**

- Open Identity Center and select Multi Account Permissions → AWS accounts. Select Assign Users or Groups

![IDC AWS Accounts](assets/idc_awsaccounts.png)

**Figure 7. IDC AWS Accounts Select Assign Users or Groups**

- Select Admin IDC Group

![Select Group](assets/select_group.png)

**Figure 8. Select Admin IDC Group**

- Select Admin Permission Set

![Select Permission Set](assets/select_set.png)

**Figure 9. Select Admin IDC Permission Set**

- Submit. Repeat for User group/permission set.

![Review and Submit](assets/review_and_submit.png)

**Figure 10. Submit. Repeat for User IDC Group and Permission Set**

- Review the admin group and verify created user

![Admin User](assets/admin_user.png)

**Figure 11. View the Administrators Group and Verify Created User**

- Make sure the groups can be added to AWS Transform

![Transform Group](assets/transform_group.png)

**Figure 12. Verify that IDC groups can be added to AWS Transform**

- Make sure the start URL can be accessed by Admin user

![Transform Start](assets/transform_start.png)

**Figure 13. Verify that Start URL can be accessed by Administrator User**

At this point all of the pre-requisites are complete and you are ready now to use AWS Transform for .NET. Follow the steps [here](https://docs.aws.amazon.com/transform/latest/userguide/what-is-transform.html) to continue

## Running the Guidance

Please see the [AWS official documentation](https://docs.aws.amazon.com/transform/latest/userguide/what-is-transform.html) of AWS Transform for .NET for details of using the Service.

Before starting your own .NET modernization project, we recommend completing the [AWS Transform for .NET workshop](https://catalog.workshops.aws/transform-dotnet). In this workshop, gain hands-on experience porting a .NET Framework application to cross-platform .NET. Use AWS Transform for .NET, a modernization service powered by generative AI that significantly reduces the time to migrate applications from Windows Server to Linux. Learn how migrating to Linux helps you avoid additional licensing fees and gain performance and security benefits.

Please feel free to explore our [self-guided demo](https://aws.storylane.io/demo/5xy6f98m17hm) to learn how AWS Transform for .NET accelerates large-scale modernization from .NET Framework to cross-platform .NET.

### Troubleshooting

In case when either of the guidance deployment phases described above fail, you should start troubleshooting their deployment from the CloudFormation console that would show the failed step in the "Events" tab as illustrated in the example below:

![Phase 2 Deployment Failed](assets/phase2_stack_creation_failed.jpg)

**Figure 14. Example of failed phase 2 Cloud Formation deployment**

To determine the root cause, follow the specified log group in the Cloud Watch service area from the error message and locate an event that contains an Error log message as illustrated below:

![Phase 2 CloudWatch Log](assets/phase2_stack_creation_failed_cloudwatch_log_details.jpg)

**Figure 15. Example of CloudWatch log with a message for failed phase 2 Cloud Formation deployment**

Then you can examine the detailed message and determine the root cause of an error:

```
"Data": {
        "Error": "An error occurred (AccessDeniedException) when calling the CreateGroupMembership operation: User: arn:aws:sts::1234567XXXXXXX:assumed-role/aws-transform-setup-IdentityCenterLambdaRole-WriIgQEV4Wx9/aws-transform-setup-AddUserToGroupFunction-Z1ugUVrdHWr7 is not authorized to perform: identitystore:CreateGroupMembership on resource: arn:aws:identitystore:::group/6468d408-50b1-7045-2426-80200c9a324f because no identity-based policy allows the identitystore:CreateGroupMembership action, User: arn:aws:sts::354918380621:assumed-role/aws-transform-setup-IdentityCenterLambdaRole-WriIgQEV4Wx9/aws-transform-setup-AddUserToGroupFunction-Z1ugUVrdHWr7 is not authorized to perform: identitystore:CreateGroupMembership on resource: arn:aws:identitystore:::user/f458b4c8-f081-7031-262c-791e8a173a98 because no identity-based policy allows the identitystore:CreateGroupMembership action"
    }
```

and make necessary updates. In this example the issue is resolved by adding necessary IAM permission policy.

## FAQ, known issues, additional considerations, and limitations

### Service Quotas

AWS Transform enforces specific service quotas, please review [AWS Transform Service Quotas](https://docs.aws.amazon.com/transform/latest/userguide/transform-limits.html) for more information.

## Cleanup

When you no longer need to use the guidance, you should delete the AWS resources deployed in order to prevent ongoing charges for their usage.

In the AWS Management Console, navigate to CloudFormation and locate the 2 guidance stacks deployed (typically named aws-org-setup and aws-transform-setup as shown in the Deployment section above). Starting with the most recent stack (not including any nested stacks), select the stack and click Delete button:

![Delete CloudFormation Stack](assets/cleanup_cfn.png)

**Figure 16. Deleting Guidance Cloud Formation Stacks**

When both stacks are successfully deleted, the corresponding AWS resources should be deleted as well.

## Authors

Pranav Kumar, GenAI Labs Builder SA  
Ashish Bhatia, Sr. SSA, Microsoft - ISV  
Deepika Suresh, SA Tech Solns  
Daniel Zilberman, Sr. Specialist SA, AWS Solutions
