# IAM Setup Guide for Next.js Application

This guide will help you create an IAM user with the necessary permissions for your Next.js application to access DynamoDB and S3.

## Step 1: Create IAM User

1. **Log into AWS Console**
   - Go to https://console.aws.amazon.com/
   - Navigate to IAM service

2. **Create New User**
   - Click "Users" in the left sidebar
   - Click "Add users" button
   - User name: `agf-dashboard-app`
   - Select "Access key - Programmatic access"
   - Click "Next: Permissions"

## Step 2: Attach Policy

### Option A: Create Custom Policy (Recommended)

1. **Create Policy**
   - In IAM Console, click "Policies" in left sidebar
   - Click "Create policy"
   - Click "JSON" tab
   - Copy and paste the contents of `nextjs-app-iam-policy.json`
   - Click "Next: Tags"
   - Click "Next: Review"
   - Policy name: `AGFDashboardAppPolicy`
   - Description: "Policy for AGF Dashboard app to read DynamoDB tables and S3 objects"
   - Click "Create policy"

2. **Attach Policy to User**
   - Go back to the user creation flow
   - Select "Attach existing policies directly"
   - Search for `AGFDashboardAppPolicy`
   - Check the checkbox next to it
   - Click "Next: Tags"
   - Click "Next: Review"
   - Click "Create user"

### Option B: Quick Setup with AWS CLI

If you have AWS CLI configured, you can run these commands:

```bash
# Create the policy
aws iam create-policy \
  --policy-name AGFDashboardAppPolicy \
  --policy-document file://nextjs-app-iam-policy.json \
  --description "Policy for AGF Dashboard app to read DynamoDB tables and S3 objects"

# Create the user
aws iam create-user --user-name agf-dashboard-app

# Attach the policy to the user (replace ACCOUNT-ID with your AWS account ID)
aws iam attach-user-policy \
  --user-name agf-dashboard-app \
  --policy-arn arn:aws:iam::ACCOUNT-ID:policy/AGFDashboardAppPolicy

# Create access keys for the user
aws iam create-access-key --user-name agf-dashboard-app
```

## Step 3: Save Access Keys

After creating the user, you'll receive:
- **Access Key ID**: Something like `AKIA...`
- **Secret Access Key**: A long string (only shown once!)

**IMPORTANT**: Save these credentials securely. You won't be able to see the Secret Access Key again!

## Step 4: Add to AWS Amplify Environment Variables

1. **Go to AWS Amplify Console**
   - Navigate to your app
   - Click on "Environment variables" in the left sidebar

2. **Add the Following Variables**:
   ```
   AWS_ACCESS_KEY_ID = [Your Access Key ID]
   AWS_SECRET_ACCESS_KEY = [Your Secret Access Key]
   AWS_REGION = ap-southeast-2
   NEXT_PUBLIC_TABLE_ENV = dev
   ```

3. **Save and Redeploy**
   - Click "Save"
   - Trigger a new deployment (or it will auto-deploy on next git push)

## Step 5: (Optional) Local Development

For local development, add these to your `.env.local` file:

```bash
# AWS Credentials for DynamoDB/S3 Access
AWS_ACCESS_KEY_ID=your_access_key_here
AWS_SECRET_ACCESS_KEY=your_secret_key_here
AWS_REGION=ap-southeast-2
NEXT_PUBLIC_TABLE_ENV=dev
```

**Note**: Never commit `.env.local` to git!

## Permissions Granted

This IAM policy grants the following permissions:

### DynamoDB (Read-Only):
- GetItem - Read single items
- Query - Query with indexes
- Scan - Scan tables
- BatchGetItem - Read multiple items
- DescribeTable - Get table metadata

### S3 (Read-Only):
- GetObject - Download files
- GetObjectVersion - Get specific file versions
- ListBucket - List bucket contents
- GetBucketLocation - Get bucket region

### Tables Accessible:
- `agf-instruments-dev` (and all indexes)
- `agf-sync-runs-dev` (and all indexes)
- `agf-file-inventory-dev` (and all indexes)
- `agf-experiments-dev` (and all indexes)

### S3 Bucket:
- `agf-instrument-data` (read-only access)

## Security Best Practices

1. **Rotate Keys Regularly**
   - Consider rotating access keys every 90 days
   - Use AWS Secrets Manager for production

2. **Use IAM Roles Instead (Advanced)**
   - For production, consider using IAM roles with AWS Amplify
   - This eliminates the need to manage access keys

3. **Monitor Access**
   - Enable CloudTrail to log API calls
   - Set up CloudWatch alarms for unusual activity

4. **Principle of Least Privilege**
   - This policy only grants read access
   - No write, update, or delete permissions
   - Only specific tables and bucket are accessible

## Troubleshooting

### If you get "Access Denied" errors:
1. Check the AWS_REGION is set to `ap-southeast-2`
2. Verify the table names match (they should end with `-dev`)
3. Ensure the IAM policy was attached correctly
4. Check CloudTrail logs for the exact API call that failed

### If credentials aren't working in Amplify:
1. Make sure environment variables are set at the app level (not branch level)
2. Trigger a full redeploy after adding variables
3. Check Amplify build logs for any errors

## Next Steps

After setting up IAM:
1. The frontend error handling fixes will prevent crashes
2. The API will be able to access DynamoDB
3. File downloads from S3 will work

The application should now have proper access to all AWS resources!