AWS Multi-Account Integration Setup
This document explains the process of configuring AWS multi-account integration for Port's software catalog, including an example.

Understanding AWS Permissions Model
Before setting up, understand these key AWS components:

Policy: Defines allowed/denied actions on AWS resources.
Role: An identity with specific permissions.
AssumeRole: Allows a user/service to assume a role temporarily.
Trust Policy: Specifies principals that can assume a role.
Account: Container for AWS resources.
Root Account: The main account managing the AWS Organization.
Integration Account: Where the Port integration is installed.
Member Accounts: Accounts from which resources are fetched.
Setting Up Roles and Permissions
1. Integration Account Setup
Role Name: PortIntegrationRole

Policy: Attach a policy that grants read-only access to resources in the Integration Account. Example policy:
json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<ROOT_ACCOUNT_ID>:role/OrganizationalOceanRole"
    }
  ]
}
Trust Policy: Allow OrganizationalOceanRole from the Root Account to assume this role:

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<ROOT_ACCOUNT_ID>:role/OrganizationalOceanRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
2. Root Account Setup
Role Name: OrganizationalOceanRole

Policy: Attach a policy that grants permissions to list accounts:
json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "organizations:ListAccounts",
      "Resource": "*"
    }
  ]
}
Trust Policy: Allow PortIntegrationRole to assume this role:

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<INTEGRATION_ACCOUNT_ID>:role/PortIntegrationRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
3. Member Accounts Setup
Role Name: AccountReadRole

Policy: Attach a policy that grants read-only permissions for resources:
json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "s3:ListAllMyBuckets",
        "rds:DescribeDBInstances"
      ],
      "Resource": "*"
    }
  ]
}
Trust Policy: Allow PortIntegrationRole to assume this role:

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<INTEGRATION_ACCOUNT_ID>:role/PortIntegrationRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
Example Configuration
Integration Configuration Parameters
yaml
Copy
Edit
organizationRoleArn: arn:aws:iam::<ROOT_ACCOUNT_ID>:role/OrganizationalOceanRole
accountReadRoleName: AccountReadRole
Example Values
Integration Account ID: 123456789012
Root Account ID: 234567890123
Member Account ID: 345678901234
Example Resource ARN
Organization Role ARN: arn:aws:iam::234567890123:role/OrganizationalOceanRole
Integration Role ARN: arn:aws:iam::123456789012:role/PortIntegrationRole
Member Role ARN: arn:aws:iam::345678901234:role/AccountReadRole
