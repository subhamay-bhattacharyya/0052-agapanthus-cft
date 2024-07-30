![](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/last-commit/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/release-date/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/repo-size/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/actions/workflow/status/subhamay-bhattacharyya/0052-agapanthus-cft/deploy-stack.yaml)&nbsp;![](https://img.shields.io/github/issues/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/languages/top/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya/0052-agapanthus-cft)&nbsp;![](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/b21a193b71079cfd71034ff303e7cae9/raw/0052-agapanthus-cft.json?)

![Project Agapanthus - Design Diagram](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/architecture-diagram.jpg)


# Project Agapanthus: Working with Glue Data Catalog and Running the Glue Crawler On Demand.

This project demonstrates how the AWS services - S3, Glue and Athena can be used to run analytics on data stored in S3 bucket in a secured environment. Security is enforced using AWS custom VPC, private subnets, VPC Endpoints, Identity and Access Management, Resource policy and Key Management service. The S3 bucket is strictly a private bucket which can be accessed only through VPC Gateway endpoint while Glue Crawlers runs in private subnets in a custom VPC. The S3 bucket policy prevents any operation from public internet and only allows upload of objects which are encrypted at rest using customer managed KMS Keys. Also the bucket policy ensures HTTPS is enforced for in-flight data transfer. Glue and KMS being public services are only accessible through VPC interface endpoints. Glue crawler is triggered manually to generate the metadata of CSV and JSON files stored in the raw S3 bucket. Then Athena is used to query the data only by the priviledged users having access to the custom WorkGroup. The entire stack is created using CloudFormation. User login credential is stored using AWS Secrets Manager.

## Description

A custom VPC houses the entire stack in a region of AWS Cloud. For this project, I am using US East 1 - North Virginia region, but any region can be used to create the resources. Two private subnets are created to ensure high availabiity. A S3 bucket is used to store the raw data which is only accessible from the private subnets enforced using S3 bucket policy. Glue crawler created to crawl the raw data to generate the metadata. The Glue Crawler and executed using manual run once the raw data is uploaded by the users. The S3 objects are encrypted using customer managed KMS key and preventing upload of unencrypted objects. Access of the bucket is restricted to only the private subnets and one custom IAM role and admin user. The Athena user is also grant read only access to the bucket in order to run analytics report by running SQL queries from Athena console..



### Services Used

```mermaid
mindmap
  root((CloudFormation ))
    AWS VPC
            Private Subnets
            Network Access Control List
            Security Group
            Route Table
            VPC Gateway Endpoint for S3

   AWS Key Management Service

   AWS Glue
          Database
          Crawler

   AWS Athena
          WorkGroup

   AWS IAM
          User
          Policy
          Role
```
# ***Prerequisites :***

## 1. Create a KMS Key or use an existing Key and update the Key policy by replacing the ***AWS Account Id***:
```json
{
    "Version": "2012-10-17",
    "Id": "key-consolepolicy-3",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::<AWS Account Id>:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::<AWS Account Id>:user/iamadmin",
                    "arn:aws:iam::<AWS Account Id>:role/OrganizationAccountAccessRole"
                ]
            },
            "Action": [
                "kms:Create*",
                "kms:Describe*",
                "kms:Enable*",
                "kms:List*",
                "kms:Put*",
                "kms:Update*",
                "kms:Revoke*",
                "kms:Disable*",
                "kms:Get*",
                "kms:Delete*",
                "kms:TagResource",
                "kms:UntagResource",
                "kms:ScheduleKeyDeletion",
                "kms:CancelKeyDeletion",
                "kms:RotateKeyOnDemand"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::<AWS Account Id>:user/iamadmin",
                    "arn:aws:iam::<AWS Account Id>:role/OrganizationAccountAccessRole",
                    "arn:aws:iam::<AWS Account Id>:role/agapanthus-workgroup-role"
                ]
            },
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow attachment of persistent resources",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::<AWS Account Id>:user/iamadmin",
                    "arn:aws:iam::<AWS Account Id>:role/OrganizationAccountAccessRole"
                ]
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": "true"
                }
            }
        },
        {
            "Sid": "Allowed services to use the encryption key",
            "Effect": "Allow",
            "Principal": {
                "Service": "s3.amazonaws.com"
            },
            "Action": [
                "kms:GenerateDataKey",
                "kms:Decrypt"
            ],
            "Resource": "*"
        }
    ]
}
```
## 2. Create a secret to store the IAM admin user password, which will be usd to create the athena user.

![Project Agapanthus -  AWS Secrets](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/secrets-manager.jpg)


## Root stack launching the child stacks to create the necessary resources:

```mermaid
---
title: Root and Nested Stacks
---
flowchart LR
    id1(Root Stack) -->|Nested Stack| id2(Network Stack)
    id2(Network Stack)-->|Nested Stack| id2.1(Custom VPC)
    id2(Network Stack)-->|Nested Stack| id2.2(Private Subnet 1)
    id2(Network Stack)-->|Nested Stack| id2.3(Private Subnet 2)
    id2(Network Stack)-->|Nested Stack| id2.4(Custom NACL)
    id2(Network Stack)-->id2.5(Default Security Group)
    id2(Network Stack)-->id2.6(Default Security Group Self Ref Ingress)
    id2(Network Stack)-->id2.7(Glue VPC Interface Endpoint)
    id2(Network Stack)-->id2.8(KMS VPC Interface Endpoint)
    id2(Network Stack)-->id2.9(S3 Gateway Endpoint)
    id2(Network Stack)-->id2.10(Private Route Table)
    id2(Network Stack)-->id2.11(Private Route Table Subnet 1 Association)
    id2(Network Stack)-->id2.12(Private Route Table Subnet 2 Association)
    id1(Root Stack) -->|Nested Stack| id3(S3 Bucket Stack)
    id1(Root Stack) -->|Nested Stack| id4(IAM Role Stack)
    id1(Root Stack) -->|Nested Stack| id5(Glue Database Stack)
    id1(Root Stack) -->|Nested Stack| id6(Glue Crawler Stack)
    id1(Root Stack) -->|Nested Stack| id7(Athena WorkGroup Stack)
    id2.1(Custom VPC) --> id2.1.1(Custom VPC)
    id2.2(Private Subnet 1) --> id2.2.1(Private Subnet)
    id2.2(Private Subnet 1) --> id2.2.2(Subnet ACL Association)
    id2.3(Private Subnet 2) --> id2.3.1(Private Subnet)
    id2.3(Private Subnet 2) --> id2.3.2(Subnet ACL Association)
    id2.4(Custom NACL) --> id2.4.1(Network ACL)
    id2.4(Custom NACL) --> id2.4.2(NACL Inbound Rule)
    id2.4(Custom NACL) --> id2.4.3(NACL Outbound Rule)
    id3(S3 Bucket Stack) --> id3.1(S3 Bucket)
    id3(S3 Bucket Stack) --> id3.2(S3 Bucket Policy)
    id4(IAM Role Stack) --> id4.1(Glue Role)
    id4(IAM Role Stack) --> id4.2(Athena Policy)
    id4(IAM Role Stack) --> id4.3(Athena Role)
    id4(IAM Role Stack) --> id4.4(Athena IAM User)
    id5(Glue Database Stack) --> id5.1(Glue Database)
    id6(Glue Crawler Stack) --> id6.1(Glue Crawler)
    id7(Athena WorkGroup Stack) --> id7.1(Athena WorkGroup)


```


# ***Resource Walkthrough :***

## 1. S3 Bucket and Bucket Policy 

```jsonc
{
    "Version": "2012-10-17",
    "Statement": [
       // Bucket policy statement to enforce AWS KMS encryption
        {
            "Sid": "Deny-unless-SSE-KMS-encrypiton",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1/*",
            "Condition": {
                "StringNotEqualsIfExists": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                },
                "Null": {
                    "s3:x-amz-server-side-encryption": "false"
                }
            }
        },
        // Bucket policy statement to enforce using a particular KMS key 
        {
            "Sid": "Deny-unless-SSE-KMS-KeyId",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1/*",
            "Condition": {
                "StringNotEqualsIfExists": {
                    "s3:x-amz-server-side-encryption-aws-kms-key-id": "arn:aws:kms:us-east-1:637423502513:key/494509e4-3bc5-44b8-9c4d-12449900d395"
                }
            }
        },
        // Bucket policy statement to enforce HTTPS for inflight data transfer
        {
            "Sid": "EnforceInFlightObjectEncryption",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1/*",
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        },
        // Bucket policy statement to enforce the bucket to be accessable via S3 Gateway Endpoint Id only
        // Condition contains the Canonical Id of the allowed user and roles which can access the bucket from S3 console
        // CLI Command reference:
        // 1. IAM User
        // aws iam get-user --user-name <User Name>
        // aws iam get-role --role-name <Role Name>
        {
            "Sid": "S3-Security-Deny-unless-VPC-endpoint",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1/*",
            "Condition": {
                "StringNotLike": {
                    "aws:userId": [
                        "AROAZI2LGYSYVERBCCHGO:*",
                        "AROAZI2LGYSY3MGOOZD75:*",
                        "AROAZI2LGYSYVKL4XJGIN:*",
                        "AIDAZI2LGYSY7WDKOVKYW"
                    ]
                },
                "StringNotEquals": {
                    "aws:sourceVpce": "vpce-033a0b15c3e7ae69f"
                }
            }
        }
    ]
}

```

## 2. Glue Database 

![Project Agapanthus - Glue Database](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/glue-database.jpg)


## 3. Glue Crawler

![Project Agapanthus - Glue Crawler](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/glue-crawler.jpg)

## 4. Glue IAM Role

![Project Agapanthus - Glue IAM Role](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/glue-iam-role.jpg)

```jsonc
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1/*",
            "Effect": "Allow"
        },
        {
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:GenerateDataKey*"
            ],
            "Resource": "arn:aws:kms:us-east-1:637423502513:key/494509e4-3bc5-44b8-9c4d-12449900d395",
            "Effect": "Allow"
        }
    ]
}
```

## 5. Athena WorkGroup

![Project Agapanthus - Athena WorkGroup](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/athena-workgroup.jpg)

## 5. Athena IAM User

![Project Agapanthus - Athena IAM User](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/athena-iam-user.jpg)

## 6. Athena IAM User Assume Role

![Project Agapanthus - Athena IAM User Assume Role](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/athena-assume-role.jpg)

```jsonc
// Athena assume role policy grants required access to the IAM user "athena"
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "athena:ListWorkGroups",
                "athena:GetWorkGroup"
            ],
            "Resource": "*",
            "Effect": "Allow",
            "Sid": "WorkGroups"
        },
        {
            "Action": [
                "athena:StartQueryExecution",
                "athena:StopQueryExecution",
                "athena:BatchGetQueryExecution",
                "athena:GetQueryExecution",
                "athena:ListQueryExecutions",
                "athena:GetQueryResults",
                "athena:GetQueryResultsStream",
                "athena:CreateNamedQuery",
                "athena:GetNamedQuery",
                "athena:BatchGetNamedQuery",
                "athena:ListNamedQueries",
                "athena:DeleteNamedQuery",
                "athena:CreatePreparedStatement",
                "athena:GetPreparedStatement",
                "athena:ListPreparedStatements",
                "athena:UpdatePreparedStatement",
                "athena:DeletePreparedStatement"
            ],
            "Resource": "arn:aws:athena:us-east-1:637423502513:workgroup/agapanthus-workgroup-devl-us-east-1",
            "Effect": "Allow",
            "Sid": "Athena"
        },
        {
            "Action": [
                "glue:GetDatabases",
                "glue:GetTable*"
            ],
            "Resource": "*",
            "Effect": "Allow",
            "Sid": "Glue"
        },
        {
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:ListMultipartUploadParts",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1",
                "arn:aws:s3:::agapanthus-landing-zone-devl-us-east-1/*"
            ],
            "Effect": "Allow",
            "Sid": "S3"
        }
    ]
}
```

# ***Running the Glue Crawler and querying the data in Athena :***

## 1. Upload the source data to the S3 landing zone bucket.

![Project Agapanthus - Upload S3 objects](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/upload-source-object-to-s3.jpg)



## 2. Run the Glue crawler on-demand.


![Project Agapanthus - Run Crawler](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/run-glue-crawler.jpg)


## 3. Metadata generated for the source csv and json objects.

![Project Agapanthus - CSV file metadata](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/csv-file-metadata.jpg)

![Project Agapanthus - JSON file metadata](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/json-file-metadata.jpg)

## 4. Login as "athena" IAM user, who don't have any access by default.


![Project Agapanthus - JSON file metadata](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/login-as-athena-user.jpg)

## 5. Switch role and execute the query in Athena.

![Project Agapanthus - Switch role and run query](https://subhamay-github-readme-md-images.s3.amazonaws.com/0052-agapanthus-cft/switch-role-execute-query.jpg)

## Help

:email: Subhamay Bhattacharyya - [subhamay.aws@gmail.com]

## Authors

Contributors names and contact info

Subhamay Bhattacharyya - [subhamay.aws@gmail.com]

## Version History

- 0.1
  - Initial Release

## License

This project is licensed under Subhamay Bhattacharyya. All Rights Reserved.

## Acknowledgments
