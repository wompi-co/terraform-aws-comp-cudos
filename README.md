---
title: "terraform-aws-comp-cudos"
generated_by: archai-docgen
generated_at: "2026-07-01T01:12:47.008Z"
repo_type: "Terraform"
primary_language: "Python"
framework: "boto3"
---

<div style="margin-bottom:15px">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://cdn.wompi.com/brand_wompi/logos/logo-secondary.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://cdn.wompi.com/brand_wompi/logos/logo-primary.svg">
  <img alt="Wompi Logo" src="https://cdn.wompi.com/brand_wompi/logos/logo-primary.svg" width="150">
</picture>
</div>

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![HCL](https://img.shields.io/badge/HCL-%23844FBA.svg?style=for-the-badge&logo=terraform&logoColor=white)

# terraform-aws-comp-cudos



This repository provides three Terraform modules for deploying AWS Cloud Intelligence Dashboards (CUDOS framework) in single or multi-account AWS Organizations environments. The modules automate Cost and Usage Report (CUR) setup with cross-account S3 replication and deploy pre-built QuickSight dashboards for cost optimization, KPI tracking, Trusted Advisor insights, and Compute Optimizer recommendations.

The `cur-setup-source` module creates CUR in payer accounts with S3 replication to a central aggregation bucket, `cur-setup-destination` provisions the aggregation bucket in the data collection account, and `cid-dashboards` deploys QuickSight dashboards via a CloudFormation wrapper that invokes the cid-cmd CLI tool through Lambda-backed custom resources.

## Key Features

- **Multi-Account CUR Aggregation**: S3 replication from multiple payer accounts to central bucket
- **Three Independent Modules**: Separate source, destination, and dashboard deployment modules
- **Pre-Built QuickSight Dashboards**: CUDOS v5, Cost Intelligence, KPI, TAO, and Compute Optimizer
- **Automated Athena Integration**: Glue crawler and database setup with daily metadata refresh
- **Secure S3 Configuration**: Versioning, public access blocking, TLS 1.2+ enforcement
- **CloudFormation Wrapper**: Lambda-based cid-cmd automation for dashboard lifecycle
- **KMS Encryption Support**: Optional customer-managed keys for CUR data (experimental)
- **Version Locking**: Git ref support for stable module deployment

## Usage

> [!NOTE]
> All modules require a `us-east-1` provider alias due to AWS CUR API requirements. The modules work independently but are designed to be deployed together for multi-account CUR aggregation.

### Basic Example - Destination Account

Provisions the aggregation S3 bucket in the data collection account that receives replicated CUR data from payer accounts.

```terraform
provider "aws" {
  region = "us-west-2"
  alias  = "data_collection"
}

provider "aws" {
  region = "us-east-1"
  alias  = "data_collection_useast1"
}

module "cur_destination" {
  source = "github.com/aws-samples/aws-cudos-framework-deployment//terraform-modules/cur-setup-destination?ref=0.2.14"

  source_account_ids = ["123456789012", "210987654321"]
  create_cur         = false

  providers = {
    aws         = aws.data_collection
    aws.useast1 = aws.data_collection_useast1
  }
}
```

### Basic Example - Source Account

Creates CUR in payer account with S3 replication to the destination bucket.

```terraform
provider "aws" {
  region = "us-west-2"
  alias  = "payer"
}

provider "aws" {
  region = "us-east-1"
  alias  = "payer_useast1"
}

module "cur_source" {
  source = "github.com/aws-samples/aws-cudos-framework-deployment//terraform-modules/cur-setup-source?ref=0.2.14"

  destination_bucket_arn = module.cur_destination.cur_bucket_arn

  providers = {
    aws         = aws.payer
    aws.useast1 = aws.payer_useast1
  }
}
```

### Basic Example - Dashboard Deployment

Deploys QuickSight dashboards via CloudFormation wrapper.

```terraform
module "cid_dashboards" {
  source = "github.com/aws-samples/aws-cudos-framework-deployment//terraform-modules/cid-dashboards?ref=0.2.14"

  stack_name      = "Cloud-Intelligence-Dashboards"
  template_bucket = "my-cfn-templates-bucket"

  stack_parameters = {
    "PrerequisitesQuickSight"            = "yes"
    "PrerequisitesQuickSightPermissions" = "yes"
    "QuickSightUser"                     = "admin/john-doe"
    "DeployCUDOSv5"                      = "yes"
    "DeployCostIntelligenceDashboard"    = "yes"
    "DeployKPIDashboard"                 = "yes"
  }
}
```

### With Custom Resource Prefix and S3 Logging

Configures custom naming and enables S3 access logging for the CUR bucket.

```terraform
module "cur_destination" {
  source = "github.com/aws-samples/aws-cudos-framework-deployment//terraform-modules/cur-setup-destination?ref=0.2.14"

  source_account_ids = ["123456789012"]
  create_cur         = true
  resource_prefix    = "myorg"
  cur_name_suffix    = "billing"

  s3_access_logging = {
    enabled = true
    bucket  = "my-logging-bucket"
    prefix  = "cur-destination-logs/"
  }

  providers = {
    aws         = aws.data_collection
    aws.useast1 = aws.data_collection_useast1
  }
}
```

### With SPICE Refresh Schedule and IAM Permissions Boundary

Enables scheduled QuickSight SPICE dataset refreshes with custom IAM configuration.

```terraform
module "cid_dashboards" {
  source = "github.com/aws-samples/aws-cudos-framework-deployment//terraform-modules/cid-dashboards?ref=0.2.14"

  stack_name      = "Cloud-Intelligence-Dashboards"
  template_bucket = "my-cfn-templates-bucket"

  stack_parameters = {
    "PrerequisitesQuickSight"            = "yes"
    "PrerequisitesQuickSightPermissions" = "yes"
    "QuickSightUser"                     = "admin/jane-smith"
    "QuickSightDataSetRefreshSchedule"   = "cron(0 6 * * ? *)"
    "DeployCUDOSv5"                      = "yes"
    "PermissionsBoundary"                = "arn:aws:iam::123456789012:policy/OrgBoundary"
    "RolePath"                           = "/analytics/"
  }

  stack_tags = {
    Environment = "production"
    CostCenter  = "finance"
  }
}
```



<!-- BEGIN_TF_DOCS -->
## Requirements

No requirements.

## Providers

No providers.

## Modules

No modules.

## Resources

No resources.

## Inputs

No inputs.

## Outputs

No outputs.
<!-- END_TF_DOCS -->
