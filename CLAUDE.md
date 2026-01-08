# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This submodule contains core infrastructure for the Australian Genome Foundry:
- **IAM roles and policies** for Lambda, Amplify, and frontend access
- **CloudWatch** log group configurations
- **Cross-cutting security** configurations

## Directory Structure

```
core/
├── iam/
│   └── policies/
│       ├── lambda-ingestion-role.json     # Data ingestion Lambda role
│       ├── lambda-zip-generator-role.json # Zip generator Lambda role
│       └── nextjs-app-policy.json         # Frontend DynamoDB access
├── cloudwatch/
│   └── (log group configurations)
└── IAM_SETUP_GUIDE.md                     # Frontend IAM setup guide
```

## IAM Roles

| Role | Service | Purpose |
|------|---------|---------|
| agf-ingestion-lambda-role-dev | Lambda | Process S3 metadata → DynamoDB |
| agf-zip-generator-role-dev | Lambda | Read/write S3 for zip generation |
| AmplifyRole-AGFDashboard | Amplify | CI/CD and hosting |
| AGF-CognitoPostConfirmation-role | Lambda | Cognito post-confirmation trigger |

## IAM Policies Summary

### Lambda Ingestion Role
- **AWSLambdaBasicExecutionRole** (managed)
- **DynamoDBAccess** (inline): PutItem, GetItem, UpdateItem, Query, Scan on 3 tables + indexes
- **S3ReadAccess** (inline): GetObject, ListBucket on agf-instrument-data

### Zip Generator Role
- **AWSLambdaBasicExecutionRole** (managed)
- **S3ReadWriteAccess** (inline): GetObject, PutObject on agf-instrument-data/*

### Frontend App Policy
- DynamoDB read-only access for dashboard queries
- S3 GetObject for presigned URL generation

## Common Commands

### View IAM Role
```bash
aws iam get-role --role-name agf-ingestion-lambda-role-dev
aws iam list-role-policies --role-name agf-ingestion-lambda-role-dev
```

### Update Inline Policy
```bash
aws iam put-role-policy --role-name agf-ingestion-lambda-role-dev --policy-name DynamoDBAccess --policy-document file://iam/policies/dynamodb-access.json
```
