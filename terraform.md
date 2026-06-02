# Terraform Cheatsheet

## Commands

```bash
terraform init      # download providers, init backend
terraform plan      # preview changes
terraform apply     # apply changes
terraform destroy   # delete all resources

terraform fmt       # format files
terraform validate  # check syntax
terraform output    # show outputs
```

## Basic Structure

```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}
```

## Variables

```hcl
# variables.tf
variable "region" {
  type    = string
  default = "us-east-1"
}

variable "environment" {
  type = string
}

variable "tags" {
  type = map(string)
  default = {}
}
```

```bash
# Pass variables
terraform apply -var="environment=prod"

# Or use .tfvars
terraform apply -var-file="prod.tfvars"
```

## Outputs

```hcl
output "bucket_name" {
  value = aws_s3_bucket.main.id
}

output "api_url" {
  value = aws_apigatewayv2_api.main.api_endpoint
}
```

## S3 Bucket

```hcl
resource "aws_s3_bucket" "main" {
  bucket = "${var.project}-${var.environment}"

  tags = {
    Environment = var.environment
  }
}
```

## Lambda Function

```hcl
resource "aws_lambda_function" "main" {
  function_name = "${var.project}-handler"
  runtime       = "python3.12"
  handler       = "main.handler"
  role          = aws_iam_role.lambda.arn

  filename         = "lambda.zip"
  source_code_hash = filebase64sha256("lambda.zip")

  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.main.name
    }
  }
}
```

## IAM Role for Lambda

```hcl
resource "aws_iam_role" "lambda" {
  name = "${var.project}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}
```

## DynamoDB Table

```hcl
resource "aws_dynamodb_table" "main" {
  name         = "${var.project}-table"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "PK"
  range_key    = "SK"

  attribute {
    name = "PK"
    type = "S"
  }

  attribute {
    name = "SK"
    type = "S"
  }

  attribute {
    name = "GSI1PK"
    type = "S"
  }

  global_secondary_index {
    name            = "GSI1"
    hash_key        = "GSI1PK"
    projection_type = "ALL"
  }
}
```

## Referencing Resources

```hcl
# Use attributes from other resources
bucket_name = aws_s3_bucket.main.id
table_arn   = aws_dynamodb_table.main.arn
role_arn    = aws_iam_role.lambda.arn
```

## State Backend (S3)

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "project/terraform.tfstate"
    region = "us-east-1"
  }
}
```

## Notes

- Run `terraform init` after adding providers or backend
- Use `-auto-approve` to skip confirmation (CI/CD only)
- `terraform plan -out=plan.tfplan` then `terraform apply plan.tfplan` for safety
- State file contains secrets - never commit to git

## Docs

- https://registry.terraform.io/providers/hashicorp/aws/latest/docs
