# Guardium Discovery for Universal Connector

Terraform-based discovery tool that automatically scans AWS accounts to find datastores and generates Terraform configurations for IBM Guardium Data Protection Universal Connector integration.

## Overview

This repository provides automated discovery and configuration generation for **Universal Connector (UC)**: One-shot discovery that creates a massive main.tf with all discovered datastores configured for audit logging.

## Features

- **Automated AWS Discovery**: Scans AWS accounts for RDS instances, Aurora clusters, DynamoDB tables, Redshift clusters, and DocumentDB
- **UC One-Shot Generation**: Creates a single main.tf with all UC-compatible datastores configured
- **PostgreSQL Session-Only**: Automatically filters to use only PostgreSQL session modules (no object-level)
- **Terraform Plan Ready**: Generated configurations are ready for `terraform plan` and `terraform apply`
- **Go-Based**: Fast, compiled binary with no runtime dependencies
- **Variable-Based Identifiers**: Database identifiers stored in terraform.tfvars for easy modification

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│              Guardium Discovery Tool                            │
│                                                                 │
│  Scans AWS Account → Generates Terraform Configurations         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │
                              ▼
                  ┌───────────────────────┐
                  │  UC Discovery         │
                  │  (One-Shot)           │
                  │                       │
                  │  Creates:             │
                  │  - Single main.tf     │
                  │  - All UC modules     │
                  │  - Postgres session   │
                  │    only               │
                  │  - Variable-based IDs │
                  └───────────────────────┘
                              │
                              ▼
                  ┌─────────────────────────┐
                  │  Output Directory       │
                  │                         │
                  │  - main.tf              │
                  │  - variables.tf         │
                  │  - terraform.tfvars     │
                  │  - versions.tf          │
                  │  - provider.tf          │
                  │  - README.md            │
                  └─────────────────────────┘
```

## Supported Datastores

### Universal Connector (UC)
- AWS Aurora PostgreSQL (Session only)
- AWS RDS PostgreSQL (Session only)
- AWS RDS MariaDB
- AWS RDS MySQL
- AWS Neptune
- AWS DynamoDB
- AWS DocumentDB
- AWS Redshift

## Prerequisites

1. **AWS Credentials**: Valid AWS credentials with read permissions for:
   - RDS (DescribeDBInstances, DescribeDBClusters)
   - DynamoDB (ListTables, DescribeTable)
   - Redshift (DescribeClusters)
   - DocumentDB (DescribeDBClusters)

2. **Terraform**: Version 1.0.0 or later

3. **Go**: Version 1.19 or later (for building from source, or use pre-built binaries)

4. **Guardium Data Protection**: Running GDP instance with API access

## Quick Start

### 1. Install the Discovery Tool

#### Option A: Download Pre-built Binary

```bash
# Download latest release
curl -LO https://github.com/IBM/terraform-guardium-discovery/releases/latest/download/guardium-discovery-linux-amd64
chmod +x guardium-discovery-linux-amd64
sudo mv guardium-discovery-linux-amd64 /usr/local/bin/guardium-discovery
```

#### Option B: Build from Source

```bash
git clone https://github.com/IBM/terraform-guardium-discovery.git
cd terraform-guardium-discovery
go build -o bin/guardium-discovery ./cmd/guardium-discovery
sudo mv bin/guardium-discovery /usr/local/bin/
```

### 2. Configure AWS Credentials

```bash
export AWS_REGION=us-east-1
export AWS_PROFILE=your-profile
# or
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
```

### 3. Run UC Discovery

```bash
# Discover all UC-compatible datastores and generate single main.tf
guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Review generated configuration
cat ./output/uc/main.tf

# Configure your Guardium credentials
cd ./output/uc
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your Guardium credentials

# Initialize and plan
terraform init
terraform plan
```

## Usage

### Available Commands

```bash
guardium-discovery --help

Available Commands:
  uc          Discover Universal Connector (UC) compatible datastores
  info        Display detailed information about discovered datastores
  deploy      Complete deployment workflow (discover + generate + apply)
  plan        Run terraform plan on discovered configurations
  apply       Apply terraform configuration
  help        Help about any command
```

### 1. UC Discovery Command

Discover and generate Terraform configurations for UC-compatible datastores.

#### Basic Discovery
```bash
# Discover all databases in a region
guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Dry run (preview only, no files generated)
guardium-discovery uc --region us-east-1 --dry-run

# With verbose logging
guardium-discovery uc --region us-east-1 --verbose
```

#### Interactive Selection
```bash
# Interactive menu to select specific databases
guardium-discovery uc --region us-east-1 --interactive
```

#### Include/Exclude Databases
```bash
# Include only specific databases
guardium-discovery uc --region us-east-1 \
  --include-databases "database-1,gat-aurora-psql,redshift-cluster-1" \
  --output-dir ./output/uc-selected

# Exclude test/dev databases (supports wildcards)
guardium-discovery uc --region us-east-1 \
  --exclude-databases "test-*,dev-*,Automation*" \
  --output-dir ./output/uc-prod
```

**UC Options:**
- `--region`: AWS region to scan (required)
- `--output-dir`: Output directory for generated files (default: ./output/uc)
- `--interactive`: Interactive database selection menu
- `--include-databases`: Comma-separated list of database identifiers to include (supports wildcards)
- `--exclude-databases`: Comma-separated list of database identifiers to exclude (supports wildcards)
- `--postgres-session-only`: Only include PostgreSQL session modules (default: true)
- `--include-dynamodb`: Include DynamoDB tables (default: true)
- `--include-documentdb`: Include DocumentDB clusters (default: true)
- `--include-redshift`: Include Redshift clusters (default: true)
- `--filter-tags`: Filter by tags (format: Key=Value)
- `--dry-run`: Preview what would be discovered without generating files
- `--verbose`: Enable verbose logging

### 2. Info Command

View detailed information about datastores with metrics (CPU, memory, storage, etc.).

#### Basic Usage
```bash
# View all databases in a region with metrics
guardium-discovery info --region us-east-1

# With verbose logging
guardium-discovery info --region us-east-1 -v
```

#### Filter Specific Databases
```bash
# View only specific databases
guardium-discovery info --region us-east-1 \
  --databases prod-postgres-1,prod-mysql-2

# View databases matching a pattern
guardium-discovery info --region us-east-1 \
  --databases np-aurora-mysql-instance-*
```

#### Export to Different Formats
```bash
# Export to JSON (for automation/scripts)
guardium-discovery info --region us-east-1 \
  --format json \
  --output databases-metrics.json

# Export to CSV (for Excel/spreadsheet analysis)
guardium-discovery info --region us-east-1 \
  --format csv \
  --output databases-metrics.csv

# JSON to stdout (pipe to jq for filtering)
guardium-discovery info --region us-east-1 --format json | \
  jq '.[] | select(.Status == "available")'
```

**Info Options:**
- `--region`: AWS region to scan (required)
- `--databases`: Comma-separated list of database identifiers to view
- `--format`: Output format: table (default), json, csv
- `--output`: Output file path (stdout if not specified)

### 3. Deploy Command

Complete end-to-end deployment workflow: discover → generate → apply.

```bash
# Interactive deployment with Guardium credentials
guardium-discovery deploy \
  --region us-east-1 \
  --gdp-server guardium.example.com \
  --gdp-username admin \
  --gdp-password your-password \
  --gdp-client-id client1 \
  --gdp-client-secret your-secret \
  --interactive

# Non-interactive deployment (all databases)
guardium-discovery deploy \
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
- `--gdp-server`: Guardium server hostname (required)
- `--gdp-username`: Guardium username (required)
- `--gdp-password`: Guardium password (required)
- `--gdp-client-id`: Guardium client ID (required)
- `--gdp-client-secret`: Guardium client secret (required)
- `--gdp-port`: Guardium port (default: 8443)
- `--auto-approve`: Skip terraform apply confirmation

### 4. Plan Command

Run terraform plan on discovered configurations:

```bash
guardium-discovery plan --output-dir ./output/uc
```

### 5. Apply Command

Apply terraform configuration:

```bash
guardium-discovery apply --output-dir ./output/uc
```

## Generated Structure

### UC Output Structure

```
output/uc/
├── main.tf                      # Single file with all UC modules
├── variables.tf                 # All required variables
├── terraform.tfvars.example     # Example configuration with discovered databases
├── versions.tf                  # Terraform and provider versions
├── provider.tf                  # AWS and Guardium providers
└── README.md                    # Usage instructions
```

### Example terraform.tfvars.example

```hcl
# Discovered Database Identifiers
# Map of sanitized keys to actual AWS resource identifiers
discovered_databases = {
  database_1                     = "database-1"
  aurora_postgres_cluster        = "aurora-postgres-cluster"
  mysql_instance                 = "mysql-instance"
  # ... all discovered databases
}

# Guardium Data Protection Configuration
gdp_server             = "guardium.example.com"
gdp_port               = "8443"
gdp_username           = "admin"
gdp_password           = "your-guardium-password"
gdp_client_id          = "your-client-id"
gdp_client_secret      = "your-client-secret"
gdp_ssh_username       = "guardium"
gdp_ssh_privatekeypath = "/path/to/private/key"
gdp_mu_host            = "guardium-mu.example.com"

# Universal Connector Configuration
udc_aws_credential = "aws-credentials-name"

# AWS Configuration
aws_region = "us-east-1"

# Tags
tags = {
  Environment = "Production"
  ManagedBy   = "Terraform"
  Purpose     = "Guardium-UC-Discovery"
}
```

## Configuration

### UC Configuration

After discovery, edit `output/uc/terraform.tfvars`:

```hcl
# 1. Review discovered_databases map (auto-populated)
discovered_databases = {
  database_1 = "database-1"
  # ... other databases
}

# 2. Configure Guardium credentials
gdp_server             = "guardium.example.com"
gdp_port               = "8443"
gdp_username           = "admin"
gdp_password           = "your-password"
gdp_client_id          = "your-client-id"
gdp_client_secret      = "your-client-secret"
gdp_ssh_username       = "guardium"
gdp_ssh_privatekeypath = "/path/to/private/key"
gdp_mu_host            = "guardium-mu.example.com"

# 3. Configure Universal Connector
udc_aws_credential     = "aws-credentials-name"
```

## Terraform Plan and Apply

After configuration, run terraform plan:

```bash
cd output/uc
terraform init
terraform plan -out=uc.tfplan
terraform apply uc.tfplan
```

## Common Use Cases

### Use Case 1: Quick Discovery and Review
```bash
# 1. Dry run to see what will be discovered
guardium-discovery uc --region us-east-1 --dry-run

# 2. View detailed metrics for all databases
guardium-discovery info --region us-east-1

# 3. Generate configurations
guardium-discovery uc --region us-east-1 --output-dir ./output/uc
```

### Use Case 2: Production-Only Deployment
```bash
# Exclude test/dev databases and deploy
guardium-discovery deploy \
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
guardium-discovery uc \
  --region us-east-1 \
  --include-databases "prod-postgres,prod-mysql,critical-aurora" \
  --output-dir ./output/uc-critical
```

### Use Case 4: Export Database Inventory
```bash
# Export all database info to CSV for reporting
guardium-discovery info \
  --region us-east-1 \
  --format csv \
  --output database-inventory.csv

# Export to JSON for automation
guardium-discovery info \
  --region us-east-1 \
  --format json \
  --output database-inventory.json
```

### Use Case 5: Multi-Region Discovery
```bash
# Discover across multiple regions
for region in us-east-1 us-west-2 eu-west-1; do
  guardium-discovery uc \
    --region $region \
    --output-dir ./output/uc-$region
done

# Consolidate metrics from all regions
for region in us-east-1 us-west-2 eu-west-1; do
  guardium-discovery info \
    --region $region \
    --format json >> all-regions-inventory.json
done
```

## Advanced Usage

### Filter by Tags

```bash
# Discover only datastores with specific tags
guardium-discovery uc \
  --region us-east-1 \
  --filter-tags "Environment=Production,ManagedBy=Terraform"
```

### Wildcard Patterns

```bash
# Include all production databases
guardium-discovery uc \
  --region us-east-1 \
  --include-databases "prod-*,production-*"

# Exclude multiple patterns
guardium-discovery uc \
  --region us-east-1 \
  --exclude-databases "test-*,dev-*,temp-*,Automation*"
```

### Combining Filters

```bash
# Complex filtering: production databases, exclude specific ones
guardium-discovery uc \
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
guardium-discovery uc --region us-east-1 --dry-run

# Generate configurations
guardium-discovery uc --region us-east-1 --output-dir ./output/uc

# Plan (review changes)
cd ./output/uc
terraform init
terraform plan -out=uc.tfplan

# Apply (in production pipeline)
terraform apply uc.tfplan
```

## Security Considerations

- **Passwords**: Never commit passwords to version control
- **Secrets Management**: Use AWS Secrets Manager or HashiCorp Vault for sensitive data
- **IAM Permissions**: Use least-privilege IAM roles for discovery
- **Network Access**: Ensure Lambda functions can access databases in VPC
- **Encryption**: Use encrypted connections for database access

## Troubleshooting

### Discovery Issues

**Problem**: No datastores discovered
```bash
# Check AWS credentials
aws sts get-caller-identity

# Check region
aws rds describe-db-instances --region us-east-1

# Enable verbose logging
guardium-discovery uc --region us-east-1 --verbose
```

**Problem**: Permission denied
```bash
# Verify IAM permissions
aws iam get-user
aws iam list-attached-user-policies --user-name your-user
```

### Terraform Issues

**Problem**: Module not found
```bash
# Ensure module paths are correct
terraform init -upgrade
```

**Problem**: Provider authentication failed
```bash
# Check Guardium connectivity
curl -k https://guardium.example.com:8443/restAPI/online
```

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

```text
#
# Copyright IBM Corp. 2025
# SPDX-License-Identifier: Apache-2.0
#
```

## Support

For issues and questions:
- Create an issue in this repository
- Contact the maintainers

## Related Projects

- [terraform-guardium-datastore-audit](https://github.com/IBM/terraform-guardium-datastore-audit) - Audit/UC modules
- [terraform-guardium-gdp](https://github.com/IBM/terraform-guardium-gdp) - GDP integration modules
- [terraform-provider-guardium-data-protection](https://github.com/IBM/terraform-provider-guardium-data-protection) - Guardium Terraform provider
