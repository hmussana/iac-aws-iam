# iac-aws-iam

## Introduction
IaC AWS IAM is a set of example AWS CloudFormation templates for quickly setting up infrastructure pipelines for **Cross-Account Access** using **AWS CodePipeline** and GitHub repo as the pipeline trigger/source. Using this setup, IAM users(human users intended) only have to be created once in the jump account(manager account) and then can assume roles to other AWS accounts owned by the same owner(think AWS Organizations).
This repository contains cloudformation templates for setting up: **IAM Users**, **IAM Groups** and **IAM Roles** for those who:
* have multiple AWS accounts and want to manage & roll out all human users automatically (Continuous deployment).
* want to setup multiple AWS accounts from scratch and looking to implement cloudformation templates for AWS IAM users, roles and groups.


## Services
* **AWS IAM:** Identity and Access Management service for accessing AWS services
* **AWS CloudFormation:** Infrastructure as Code Service for creating AWS resources. Yaml as the templates language
* **AWS CodePipeline:**	Continuous Delivery Service by AWS, used here to automatically deploy any changes pushed to the source git repository.


## Usage

### Terminology
* **manager-account:** Main AWS account which only has IAM Groups and Users and acts as a jump account.
* **managed-accounts:** Team/Project & environment wise AWS accounts e.g. development-staging or androidApp-production.
* **jump account:** Another name for manager account.

### Login Workflow
User Login: ` Login to manager account -> Assume role to a managed account based upon group membership`

### Directory Structure
```
iac-aws-iam
│
├── README.md
├── managed-accounts
│   └── project1-production
│       ├── pipeline.yaml
│       └── roles.yaml
└── manager-account
    ├── groups-admin.yaml
    ├── pipeline.yaml
    └── users.yaml
```

### Setup Instructions
Following two types of pipelines will be setup using only the already defined resources in templates:

1. **IAM Roles Pipeline:** Should be setup first in the managed account(s) so the IAM groups referencing to `AssumeRole` of the relevant role will not point to a non-existent resource. Each AWS account should have its own pipeline for setting up roles.
2. **IAM Users and Groups Pipeline:** Has to be setup once in the main/manager account.

### Deployment Steps
1. Rename `project1-production` as desired and add all the aws accounts as directories under: `managed-accounts`
	- Also adjust the `TemplatePath` values accordingly in *pipeline.yaml*.
2. Replace the example AWS account ID `012345678901` in `project1-production` and subsequently for all managed accounts you want to use.
3. Create the pipeline stack in AWS CloudFormation console or via api/cli using `managed-accounts/project1-production/pipeline.yaml` which will setup the roles stack itself. See [this](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for how to do that.
4. Provide your GitHub token for the parameter value. Check [this guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/integrations-action-type.html#integrations-source
) for details on how to generate one.
5. Update/add the example account ID mappings in `groups-admin.yaml`. Create more group templates if desired using this file and name appropriately e.g. `groups-developer.yaml` or `groups-readonly.yaml`. Also change project names as desired and adjust `GroupName` and Output Export values accordingly.
6. Add more list items under `Action` under `Group-Stage` in the pipeline with correct `TemplatePath` for each new template added(assuming more group templates were created in above step).
7. Add more IAM users in the `users.yaml` as desired and adjust passwords and group memberships. 
	* **Important** The `GroupName` attribute is being imported from the IAM groups stack so the names must match. 
8. Login to manager account and create the pipeline stack using `manager-account/pipeline.yaml` and provide GuitHub access token again.

Once the pipelines finish, the users should be able to login using their one-time passwords and can then assume roles into various AWS accounts as defined by the IAM groups they are members of.


## Author
[hmussana](https://github.com/hmussana)

## References
Here are some documentation links which can be used as references:

    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-codepipeline-pipeline.html
    https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html
    https://docs.aws.amazon.com/codepipeline/latest/userguidereference-pipeline-structure.html#pipeline-requirements
