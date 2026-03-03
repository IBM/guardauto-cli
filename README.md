# Guardium Discovery CLI

Automated discovery tool that scans AWS accounts to find datastores and generates Terraform configurations for IBM Guardium Data Protection Universal Connector integration.

## Overview

The Guardium Discovery CLI provides a complete workflow for discovering, configuring, and deploying Guardium monitoring for AWS datastores. It supports multiple commands for different use cases, from simple discovery to full end-to-end deployment.

## Features

- 🔍 **Automated Discovery** - Scan AWS accounts for RDS, Aurora, DynamoDB, Redshift, and DocumentDB
- 📊 **Database Metrics** - View detailed information about discovered datastores (CPU, memory, storage)
- 🎯 **Selective Deployment** - Include/exclude specific databases with wildcard support
- 🚀 **End-to-End Workflow** - Complete deploy command from discovery to Terraform apply
- 📤 **Export Capabilities** - Export database inventory to JSON or CSV
- 🔄 **Interactive Mode** - Manual database selection with interactive menu
- 🏷️ **Tag Filtering** - Filter databases by AWS tags

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

**Note for macOS users:** After downloading, you may need to remove the quarantine attribute:
```bash
xattr -d com.apple.quarantine ./guardium-discovery
```

### 2. Quick Discovery

```bash
# Preview what will be discovered (dry run)
./guardium-discovery uc --region us-east-1 --dry-run

# Discover and generate Terraform configurations
./guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Review generated configuration
cat ./output/uc/main.tf
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

## Available Commands

```bash
./guardium-discovery --help

Available Commands:
  uc          Discover Universal Connector (UC) compatible datastores
  info        Display detailed information about discovered datastores
  deploy      Complete deployment workflow (discover + generate + apply)
  plan        Run terraform plan on discovered configurations
  apply       Apply terraform configuration
  help        Help about any command
```

## Command Usage

### 1. UC Discovery Command

Discover and generate Terraform configurations for UC-compatible datastores.

#### Basic Discovery
```bash
# Discover all databases in a region
./guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Dry run (preview only, no files generated)
./guardium-discovery uc --region us-east-1 --dry-run

# With verbose logging
./guardium-discovery uc --region us-east-1 --verbose
```

#### Interactive Selection
```bash
# Interactive menu to select specific databases
./guardium-discovery uc --region us-east-1 --interactive
```

#### Include/Exclude Databases
```bash
# Include only specific databases
./guardium-discovery uc --region us-east-1 \
  --include-databases "database-1,prod-aurora,redshift-cluster-1" \
  --output-dir ./output/uc-selected

# Exclude test/dev databases (supports wildcards)
./guardium-discovery uc --region us-east-1 \
  --exclude-databases "test-*,dev-*,Automation*" \
  --output-dir ./output/uc-prod
```

**UC Options:**
- `--region` - AWS region to scan (required)
- `--output-dir` - Output directory for generated files (default: ./output/uc)
- `--interactive` - Interactive database selection menu
- `--include-databases` - Comma-separated list of database identifiers to include (supports wildcards)
- `--exclude-databases` - Comma-separated list of database identifiers to exclude (supports wildcards)
- `--postgres-session-only` - Only include PostgreSQL session modules (default: true)
- `--include-dynamodb` - Include DynamoDB tables (default: true)
- `--include-documentdb` - Include DocumentDB clusters (default: true)
- `--include-redshift` - Include Redshift clusters (default: true)
- `--filter-tags` - Filter by tags (format: Key=Value)
- `--dry-run` - Preview what would be discovered without generating files
- `--verbose` - Enable verbose logging

### 2. Info Command

View detailed information about datastores with metrics (CPU, memory, storage, etc.).

#### Basic Usage
```bash
# View all databases in a region with metrics
./guardium-discovery info --region us-east-1

# With verbose logging
./guardium-discovery info --region us-east-1 -v
```

#### Filter Specific Databases
```bash
# View only specific databases
./guardium-discovery info --region us-east-1 \
  --databases prod-postgres-1,prod-mysql-2

# View databases matching a pattern
./guardium-discovery info --region us-east-1 \
  --databases prod-*
```

#### Export to Different Formats
```bash
# Export to JSON (for automation/scripts)
./guardium-discovery info --region us-east-1 \
  --format json \
  --output databases-metrics.json

# Export to CSV (for Excel/spreadsheet analysis)
./guardium-discovery info --region us-east-1 \
  --format csv \
  --output databases-metrics.csv

# JSON to stdout (pipe to jq for filtering)
./guardium-discovery info --region us-east-1 --format json | \
  jq '.[] | select(.Status == "available")'
```

**Info Options:**
- `--region` - AWS region to scan (required)
- `--databases` - Comma-separated list of database identifiers to view
- `--format` - Output format: table (default), json, csv
- `--output` - Output file path (stdout if not specified)

### 3. Deploy Command

Complete end-to-end deployment workflow: discover → generate → apply.

```bash
# Interactive deployment with Guardium credentials
./guardium-discovery deploy \
  --region us-east-1 \
  --gdp-server guardium.example.com \
  --gdp-username admin \
  --gdp-password your-password \
  --gdp-client-id client1 \
  --gdp-client-secret your-secret \
  --interactive

# Non-interactive deployment (all databases)
./guardium-discovery deploy \
  --region us-east-1 \
  --gdp-server guardium.example.com \
  --gdp-username admin \
  --gdp-password your-password \
  --gdp-client-id client1 \
  --gdp-client-secret your-secret \
  --output-dir ./output/uc
```

**Deploy Options:**
- All UC discovery options (--region, --include-databases, etc.)
- `--gdp-server` - Guardium server hostname (required)
- `--gdp-username` - Guardium username (required)
- `--gdp-password` - Guardium password (required)
- `--gdp-client-id` - Guardium client ID (required)
- `--gdp-client-secret` - Guardium client secret (required)
- `--gdp-port` - Guardium port (default: 8443)
- `--auto-approve` - Skip terraform apply confirmation

### 4. Plan Command

Run terraform plan on discovered configurations:

```bash
./guardium-discovery plan --output-dir ./output/uc
```

### 5. Apply Command

Apply terraform configuration:

```bash
./guardium-discovery apply --output-dir ./output/uc
```

## Common Use Cases

### Use Case 1: Quick Discovery and Review
```bash
# 1. Dry run to see what will be discovered
./guardium-discovery uc --region us-east-1 --dry-run

# 2. View detailed metrics for all databases
./guardium-discovery info --region us-east-1

# 3. Generate configurations
./guardium-discovery uc --region us-east-1 --output-dir ./output/uc
```

### Use Case 2: Production-Only Deployment
```bash
# Exclude test/dev databases and deploy
./guardium-discovery deploy \
  --region us-east-1 \
  --exclude-databases "test-*,dev-*,sandbox-*" \
  --gdp-server guardium.example.com \
  --gdp-username admin \
  --gdp-password your-password \
  --gdp-client-id client1 \
  --gdp-client-secret your-secret \
  --interactive
```

### Use Case 3: Specific Databases Only
```bash
# Select specific critical databases
./guardium-discovery uc \
  --region us-east-1 \
  --include-databases "prod-postgres,prod-mysql,critical-aurora" \
  --output-dir ./output/uc-critical
```

### Use Case 4: Export Database Inventory
```bash
# Export all database info to CSV for reporting
./guardium-discovery info \
  --region us-east-1 \
  --format csv \
  --output database-inventory.csv

# Export to JSON for automation
./guardium-discovery info \
  --region us-east-1 \
  --format json \
  --output database-inventory.json
```

### Use Case 5: Multi-Region Discovery
```bash
# Discover across multiple regions
for region in us-east-1 us-west-2 eu-west-1; do
  ./guardium-discovery uc \
    --region $region \
    --output-dir ./output/uc-$region
done

# Consolidate metrics from all regions
for region in us-east-1 us-west-2 eu-west-1; do
  ./guardium-discovery info \
    --region $region \
    --format json >> all-regions-inventory.json
done
```
## Advanced Usage

### Filter by Tags

```bash
# Discover only datastores with specific tags
./guardium-discovery uc \
  --region us-east-1 \
  --filter-tags "Environment=Production,ManagedBy=Terraform"
```

### Wildcard Patterns

```bash
# Include all production databases
./guardium-discovery uc \
  --region us-east-1 \
  --include-databases "prod-*,production-*"

# Exclude multiple patterns
./guardium-discovery uc \
  --region us-east-1 \
  --exclude-databases "test-*,dev-*,temp-*,Automation*"
```

### Combining Filters

```bash
# Complex filtering: production databases, exclude specific ones
./guardium-discovery uc \
  --region us-east-1 \
  --filter-tags "Environment=Production" \
  --exclude-databases "prod-test-*,prod-sandbox-*" \
  --interactive
```

### Pipeline Integration

```bash
# CI/CD pipeline example
#!/bin/bash
set -e

# Discover and validate
./guardium-discovery uc --region us-east-1 --dry-run

# Generate configurations
./guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Plan (review changes)
cd ./output/uc
terraform init
terraform plan -out=uc.tfplan

# Apply (in production pipeline)
terraform apply uc.tfplan
```


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