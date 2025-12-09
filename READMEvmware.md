# Guidance for Automated Setup of AWS Transform for VMware

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
7. [FAQ, known issues, additional considerations, and limitations](#faq-known-issues-additional-considerations-and-limitations)
   - [AWS Transform for .NET](#aws-transform-dotnet)
   - [Service Quotas](#service-quotas)
8. [Cleanup](#cleanup)
9. [Authors](#authors-optional)

## Overview

This Guidance provides an automated approach to deploy AWS Transform using infrastructure as code (IaC). It simplifies deployment by automatically provisioning AWS services and security controls required by AWS Transform. The guidance accelerates time-to-value for organizations migrating and modernizing their workloads while ensuring adherence to AWS best practices and security standards. AWS Transform is the first agentic AI service developed to accelerate enterprise modernization of workloads.

The journey begins with a thorough discovery and assessment of your on-premises VMware environment.

1. **AWS Transform for VMware** supports multiple discovery methods:

- RVTools for VMware inventory collection
- AWS Application Discovery Agent (Discovery Agent) for gathering network communication patterns between applications and servers
- Application Discovery Service Agentless Collector (Agentless Collector) for collecting communication data without installing an agent
  These tools help build a comprehensive view of application-to-application communications, server-to-server dependencies, and overall network topology.

2. **Inventory Discovery Agent** collects crucial data from your on-premises environment and stores it securely in both Amazon Simple Storage Service (Amazon S3) buckets within the AWS Discovery account and AWS Migration Hub. This data forms the foundation for informed migration planning and is further processed by Migration Hub and AWS Application Discovery Service. AWS Transform works together with these services to provide a single place to track migration progress and collect server inventory and dependency data, which is essential for successful application grouping and wave planning.

3. **Intelligent network conversion and wave planning**
   With a comprehensive understanding of your environment, AWS Transform for VMware moves to the next critical phase. The Network Conversion Agent automates the creation of AWS CloudFormation templates to set up the target network infrastructure. These templates make sure your cloud environment closely mirrors your on-premises setup, simplifying the setup for the migration.

Meanwhile, the Wave Planning Agent uses advanced graph neural networks to analyze application dependencies and plan optimal migration waves. This minimizes complex portfolio and application dependency analysis, and provides ready-to-migrate wave plans, resulting in smooth migrations.

4. **Enhanced security and compliance**
   Security remains paramount throughout the migration process. AWS Key Management Service (AWS KMS) provides robust encryption for stored data, conversation history, and artifacts when a customer managed key (CMK) is configured. AWS Organizations enables centralized management across multiple AWS accounts, and AWS CloudTrail captures and logs API calls for a complete audit trail.

Access control is managed through AWS Identity and Access Management (IAM), providing centralized access management across AWS accounts. Amazon CloudWatch continuously monitors AWS Transform service activities, resource utilization, and operational metrics within the management account, providing full visibility and control throughout the migration process.

5. **Orchestrated migration execution**
   When it’s time to execute the migration, the Migration Agent orchestrates the migration and cutover process. It works in tandem with AWS Application Migration Service to replicate source servers to Amazon Elastic Compute Cloud (Amazon EC2) instances based on the carefully planned waves and groupings.

The AWS Provisioning/Target Account serves as the production environment where your migrated applications will reside. This account contains the target infrastructure and will house your production workloads after migration is complete. S3 buckets in this account store the CloudFormation templates used for infrastructure deployment, providing a smooth, consistent, and reliable setup process.

6. **Flexible network configuration**
   AWS Transform for VMware offers two networking models to suit different requirements:

- Hub-and-spoke model – AWS Transit Gateway connects virtual private clouds (VPCs) through a central hub VPC with shared NAT gateways. This model is ideal for centralized management and shared services.
- Isolated model – Each VPC operates independently, connected directly by Transit Gateway. This approach offers greater isolation and is suitable for environments with strict separation requirements.

VPCs created by AWS Transform match your on-premises network segments, providing a seamless transition. NAT gateways provide outbound internet access for private subnets, maintaining security while enabling necessary connectivity. In hub-and-spoke deployments, shared NAT gateways are used in the central hub VPC, whereas in isolated deployments, individual NAT gateways are created for each VPC.

### Architecture

Below is the Reference architecture for the guidance showing the core and supporting AWS services:

![architecture](assets/aws_transform_vmware_ref-arch1.jpg)

1. Customer VMware environment hosts the workloads to be migrated. RVTools can be used along with optional import/export functionality for customers running VMware NSX.
2. AWS agent and agentless Discovery agents used (in addition to or instead of RVTools) to gather and collect data and dependencies for migration. AWS Replication Agent is used to migrate virtual machines to AWS.
3. AWS Transform for VMware discovery workspaces are available globally.
   A full list of supported AWS Regions can be found [here](https://docs.aws.amazon.com/transform/latest/userguide/regions.html).
4. AWS Transform for VMware helps optimize infrastructure and reduce operational overhead, giving you a more predictable, cost-efficient path to modernization.
5. The Inventory Discovery capability collects data from the on-premises environment and stores it in the Discovery account’s Amazon Simple Storage Service (S3) buckets.
6. As part of AWS Transform, the Wave Planning capability uses Graph Neural Networks to analyze application dependencies and plan migration waves.

![architecture2](assets/aws_transform_vmware_ref-arch2.jpg)

7. The AWS Migration Planning account hosts AWS Application Discovery Service (ADS) to collect, store, and process detailed infrastructure and application data for migration planning. The Discovery account provides secure isolation of collected infrastructure data and maintains separation of discovery and migration activities.
8. AWS Key Management Service (KMS) encrypts data using AWS managed keys by default or optional Customer Managed Keys (CMK)
9. AWS Organizations enables centralized management of AWS accounts through Organizational Units
10. Amazon CloudWatch monitors AWS Transform activities, resources, and metrics in the management account
11. AWS Identity and Access Management (IAM) Identity Center provides centralized access management across all AWS accounts.
12. Amazon S3 buckets in both the Planning and Discovery accounts store key migration artifacts including inventory data, dependency mappings, wave plans, and application groupings.
13. AWS CloudFormation automates resource provisioning across AWS accounts and regions for test and production environments.
14. AWS CloudTrail logs API activities in AWS accounts, while AWS Transform service tracks migration activities.
15. AWS ADS collects server inventory and dependencies to support application grouping and wave planning.
16. AWS KMS encrypts Discovery account S3 buckets that store source environment data.

![architecture3](assets/aws_transform_vmware_ref-arch3.jpg)

17. **NOTE**: For the most up-to-date information on supported Regions, please refer to [AWS Services by Region](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)
18. The AWS Target/Provisioning Account hosts migrated production workloads and applications.
19. The Network Migration capability converts on-premises networks to AWS using AWS CloudFormation and AWS Cloud Development Kit templates.
20. AWS Transform orchestrates end-to-end migration by coordinating across various AWS tools and service, including Server Migration/Rehost capability utilizing AWS Application Migration Service.
21. Amazon Elastic Compute Cloud (EC2) and Amazon Elastic Block Store (EBS) host migrated VMware virtual machines with recommended instance types
22. The network foundation of this migration architecture relies on Amazon Virtual Private Cloud (VPC) and Amazon Transit Gateway working in tandem, where VPC provides dedicated network isolation for migrated workloads while Transit Gateway acts as the central hub connecting these VPCs, with Amazon NAT Gateways enabling secure internet access for private subnet resources. AWS Application Migration Service handles the core migration execution by managing both the initial server replication process and orchestrating the test/cutover instance launches, being supported by a comprehensive set of AWS services (KMS, CloudWatch, CloudTrail, IAM permissions, CloudFormation, and AWS S3) that work together to maintain security, enable in-depth monitoring, and automate the infrastructure deployment through stored per-wave migration plans.

### Cost

While this guidance provides default configurations, customers are responsible for:

1. Configuring the guidance to their optimal settings based on their specific use case and requirements
2. Monitoring and managing the costs incurred from any supporting AWS services
3. Reviewing and optimizing storage usage for transformation artifacts

We recommend creating a [Budget](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) through [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this Guidance.

As of April 2024, the cost for running this Guidance with the default settings in the default AWS Region (US East - N. Virginia) us-east-1 is the following:

| AWS Service             | Dimensions                         | Cost [USD]              |
| ----------------------- | ---------------------------------- | ----------------------- |
| AWS Transform           | Unlimited transformations          | Free                    |
| AWS IAM Identity Center | Number of users                    | Free                    |
| AWS Organizations       | Number of accounts in organization | Free                    |
| AWS Lambda              | First 1M requests                  | Free                    |
| AWS Lambda              | Over 1M requests                   | $0.20 per 1M requests   |
| Amazon S3               | Standard Storage                   | $0.023 per GB per month |
| AWS KMS                 | Customer managed key (optional)    | $1.00 per key per month |

## Prerequisites

### AWS Account

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

Users should refer to the [AWS Transform security documentation](https://docs.aws.amazon.com/transform/latest/userguide/security.html) to understand the full scope of permissions that may be required for their particular use cases and transformation scenarios. Key security considerations include source code security (secure access to repositories, encryption of code during transformation, protection of transformation artifacts), identity and access management (role-based access control through IAM Identity Center, least privilege access principles, separation of duties between administrators and users), and data protection (encryption at rest for transformation artifacts, secure transmission of code and artifacts, optional customer-managed KMS keys for additional control).

## Deployment Steps

Clone Guidance repository

1. Log in to your AWS account on your CLI/shell through your preferred authentication provider.

2. Clone the repository:

   ```bash
   git clone https://github.com/aws-solutions-library-samples/guidance-for-automating-aws-transformations-vmware-deployment
   ```

Phase 1: Set up AWS Organizations

> Note : If you already have AWS Organizations enabled in your Management account, you can skip Phase 1.

1. Change directory to the source folder inside the guidance repository:

   ```bash
   cd guidance-for-automating-aws-transformations-vmware-deployment/source
   ```

2. Start by running the first shell script. This creates an AWS Organization with all features enabled.

   - STACK_NAME: {name of CloudFormation stack}.
     PLATE_PATH: {path to `phase2-idc.yaml`}.

   BASH:

   ```bash
   source % ./deploy-phase1.sh
   Enter stack name [aws-org-setup]: aws-org-setup
   Enter template path [/guidance-for-automating-aws-transformations-vmware-deployment/source/phase1-aws-organizations.yaml]:
   ```

   PowerShell:

   ```powershell
       PS C:\git\aws\guidance-for-automating-aws-transformations-vmware-deployment\source> .\deploy-phase1.ps1
       Enter stack name [aws-org-setup]:
       Enter template path [phase1-aws-organizations.yaml]:
   ```

   > Note : A Powershell script is available for Windows OS. Alternatively, the parameters can be manually added to the CloudFormation YAML.

3. After successful deployment, you will need to manually enable an organization instance of IAM Identity Center in the AWS Console (wait a few minutes for the changes to propagate), as shown below:

![enable_identity_center](assets/enable_identity_center.png)

Phase 2: Set up IAM Identity Center

1. After enabling IAM Identity Center manually and waiting for updates to propagate, run the second BASH script

   Pass in the following parameters using the bash script:

   STACK_NAME: {name of cloudformation stack}.
   TEMPLATE_PATH: {path to phase2 yaml}.
   ACCOUNT_NUMBER: {AWS account number}.
   IDENTITY_CENTER_ID: {AWS Identity Center ID}.
   ADMIN_EMAIL: {Email for admin user provisioned by script}.

   BASH:

   ```bash
       source % ./deploy-phase2.sh
       Enter stack name [aws-transform-setup]:
       Enter template path: [/guidance-for-automating-aws-transformations-vmware-deployment/source/phase2-idc.yaml]:
       Enter AWS account number: 1234567XXXXXX
       Enter admin email address: admin@amazon.com
       Enter Identity Center ID: ssoins-1234a123b1d5ab3f
       Retrieving Identity Store ID for IAM Identity Center instance ssoins-1234a252c3d5bd2f...
       Found Identity Store ID: d-40338374bc
   ```

   PowerShell:

   ```powershell
       PS C:\git\aws\guidance-for-automating-aws-transformations-vmware-deployment\source> .\deploy-phase2.ps1
       Enter stack name [aws-transform-setup]:
       Enter template path [phase2-idc.yaml]:
       Enter AWS account number: 1234567XXXXXXX
       Enter admin email address: admin@amazon.com
       Enter Identity Center ID: ssoins-1234a123b1d5ab3f
       Retrieving Identity Store ID for IAM Identity Center instance ssoins-1234a252c3d5bd2f...
       Found Identity Store ID: d-40338374bc
   ```

   This CloudFormation will:

- Create IAM Identity Center groups and users
- Set up the necessary IAM policies for AWS Transform for VMware for both groups
- Create an Admin user using lambda functions in Identity Center based on a provided email

> Note: The script/CloudFormation uses the deployed Lambda functions to add the provided email account as an Admin in the created AWS Transform Admin group in AWS IAM Identity Center. Subsequent admins and users can be added via the AWS console following best practices.

## Deployment Validation

1. Open CloudFormation in AWS console and verify the status of the stacks

![cfn_stack](assets/cfn_stack.png)

2. Open Identity Center and verify the created groups created:

![idc_group](assets/idc_group.png)

3. Open Identity Center and select Multi Account Permissions → AWS accounts. Select Assign Users or Groups

![idc_awsaccounts](assets/idc_awsaccounts.png)

4. Open Identity Center and select Multi Account Permissions → AWS accounts. Select Assign Users or Groups

![idc_users](assets/idc_group.png)

5. Select Admin IDC Group

![select_group](assets/select_group.png)

6. Select Admin Permission Set

![select_set](assets/select_set.png)

7. Submit. Repeat for User group/permission set.

![review_and_submit](assets/review_and_submit.png)

8. Review the admin group and verify created user

![admin_user](assets/admin_user.png)

9. Review the user group and verify created user

10. Make sure the groups can be added to AWS Transform

![transform_group](assets/transform_group.png)

11. Make sure the start URL can be accessed by Admin user

![transform start](assets/transform_start.png)

## FAQ, known issues, additional considerations, and limitations

### AWS Transform for .NET

For those looking to modernize their organizations .NET applications, view this [README](READMEdotnet.md) for further guidance.
AWS Transform for .NET is a service designed to accelerate the modernization of Windows-based .NET Framework applications to cross-platform .NET for Linux environments. It leverages generative AI to automate and streamline the process of porting applications, aiming to reduce the time and effort typically involved in such migrations. AWS Transform for .NET provides:

- Analysis of .NET Framework codebases from source control systems, including private NuGet support
- Automated transformation of legacy .NET Framework applications to cross-platform .NET
- Integration with source control platforms (BitBucket, GitHub, and GitLab)
- Validation of transformed code through unit tests

### Service Quotas

AWS Transform enforces specific service quotas, please review [AWS Transform Service Quotas](https://docs.aws.amazon.com/transform/latest/userguide/transform-limits.html) for more information.

## Cleanup

When you no longer need to use the guidance, you should delete the AWS resources deployed in order to prevent ongoing charges for their usage.

In the AWS Management Console, navigate to CloudFormation and locate the 2 guidance stacks deployed (typcally named aws-org-setup and aws-transform-setup as shown in the Deployment section above). Starting with the most recent stack (not including any nested stacks), select the stack and click Delete button:

![cleanupcfn](assets/cleanup_cfn.png)

When both stacks are successfully deleted, the corresponding AWS resources should be deleted as well.

## Authors

- Pranav Kumar, GenAI Labs Builder SA
- Ashish Bhatia, Sr. SSA, Microsoft - ISV
- Deepika Suresh, SA Tech Solns
- Daniel Zilberman, Sr. Specialist SA, AWS Solutions
