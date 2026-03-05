# Guardium Discovery for Universal Connector

Automated discovery tool that scans AWS accounts to find datastores and generates Terraform configurations for IBM Guardium Data Protection Universal Connector integration.

## Quick Start

### 1. Configure AWS Credentials

```bash
# Option 1: AWS CLI
aws configure

# Option 2: Environment variables
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
export AWS_REGION=us-east-1

# Option 3: AWS Profile
export AWS_PROFILE=your-profile
```

### 2. Run Discovery

```bash
# Discover all UC-compatible datastores
guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Review generated configuration
cat ./output/uc/main.tf
```

**Note for macOS users:** After downloading, you may need to remove the quarantine attribute:
```bash
xattr -d com.apple.quarantine ./guardium-discovery
```

### 3. Configure and Deploy

```bash
cd ./output/uc

# Copy and edit configuration
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your Guardium credentials

# Deploy
terraform init
terraform plan
terraform apply
```

## What Gets Discovered?

The tool automatically discovers and configures:

- ✅ **AWS Aurora PostgreSQL** (Session audit only)
- ✅ **AWS RDS PostgreSQL** (Session audit only)
- ✅ **AWS Aurora MySQL**
- ✅ **AWS RDS MariaDB**
- ✅ **AWS RDS MySQL**
- ✅ **AWS DynamoDB**
- ✅ **AWS DocumentDB**
- ✅ **AWS Redshift**

## Command Options

```bash
guardium-discovery uc \
  --region us-east-1 \
  --output-dir ./output/uc \
  --postgres-session-only \
  --include-dynamodb \
  --include-documentdb \
  --include-redshift \
  --verbose
```

**Available Options:**
- `--region` - AWS region to scan (required)
- `--output-dir` - Output directory for generated files (default: ./output/uc)
- `--postgres-session-only` - Only include PostgreSQL session modules (default: true)
- `--include-dynamodb` - Include DynamoDB tables (default: true)
- `--include-documentdb` - Include DocumentDB clusters (default: true)
- `--include-redshift` - Include Redshift clusters (default: true)
- `--dry-run` - Preview what would be discovered without generating files
- `--verbose` - Enable verbose logging

## Prerequisites

1. **AWS Credentials** - Valid AWS credentials with read permissions for:
   - RDS (DescribeDBInstances, DescribeDBClusters)
   - DynamoDB (ListTables, DescribeTable)
   - Redshift (DescribeClusters)
   - DocumentDB (DescribeDBClusters)

2. **Terraform** - Version 1.0.0 or later

3. **Guardium Data Protection** - Running GDP instance with API access

## Required AWS Permissions

The discovery tool needs read-only permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances",
        "rds:DescribeDBClusters",
        "dynamodb:ListTables",
        "dynamodb:DescribeTable",
        "redshift:DescribeClusters",
        "docdb:DescribeDBClusters"
      ],
      "Resource": "*"
    }
  ]
}
```

## Support

For issues and questions:
- Create an issue in this repository
- Contact the maintainers

## Related Projects

- [terraform-guardium-datastore-audit](https://github.com/IBM/terraform-guardium-datastore-audit) - Audit/UC modules
- [terraform-guardium-gdp](https://github.com/IBM/terraform-guardium-gdp) - GDP integration modules
- [terraform-provider-guardium-data-protection](https://github.com/IBM/terraform-provider-guardium-data-protection) - Guardium Terraform provider