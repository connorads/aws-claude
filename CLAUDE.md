# Claude Code Configuration for AWS CLI with Granted.dev

## Project Overview
This project uses AWS CLI commands with granted.dev for credential management and role assumption. Claude Code should help with AWS operations while respecting security best practices.

## Authentication Setup
- **Tool**: granted.dev is used for AWS credential management
- **Command**: Use `assume <role-name>` to switch between AWS roles/accounts
- **Profile Format**: Profiles are in the format `account-name/RoleName` (e.g., `mycompany-dev/ReadOnlyAccess`)
- **Credential Usage**: After assuming a role, use `--profile <profile-name>` with AWS CLI commands
- **Verification**: Use `aws sts get-caller-identity --profile <profile-name>` to verify credentials

## AWS CLI Guidelines

### Pre-execution Checks
Before running any AWS commands:
1. Check current AWS credentials: `aws sts get-caller-identity --profile <profile-name>`
2. If wrong role/account, use `assume <profile-name>` to switch
3. Note: granted.dev configures profiles but doesn't export credentials to environment variables by default

### Common AWS Operations
- **S3 operations**: List buckets, upload/download files, sync directories
- **EC2 management**: List instances, start/stop instances, security groups
- **Lambda functions**: Deploy, invoke, view logs
- **CloudFormation**: Deploy stacks, check status, view events
- **IAM**: List users, roles, policies (read-only operations preferred)
- **AppConfig**: Manage feature flags and dynamic configuration

### Security Best Practices
- Never hardcode credentials or sensitive information
- Always verify you're in the correct AWS account before destructive operations
- Use least-privilege principle when suggesting IAM policies
- Prompt for confirmation before destructive operations (delete, terminate, etc.)
- Prefer read-only commands when exploring/debugging

### Error Handling
- If AWS commands fail due to credentials, suggest using `assume <role-name>`
- Check for region-specific resources if commands return empty results
- Provide clear explanations for AWS CLI error messages

### File Verification Best Practices
When creating or modifying files (especially configuration files):
- Always use `diff` to compare changes before applying them
- Review the differences to ensure only intended changes are present
- Check for unexpected additions at the end of files (e.g., shell artifacts like `EOF`)
- Validate file formats after modification (e.g., use `jq` for JSON validation)
- **IMPORTANT**: When creating files, use the Write tool instead of bash heredocs to avoid EOF markers being included in the file
- Example workflow:
  ```bash
  # AVOID: Using heredocs with cat can add EOF to the file
  # cat > config.json << 'EOF'
  # { "data": "value" }
  # EOF
  
  # PREFER: Use the Write tool for clean file creation
  # Or if using bash, use echo or printf
  
  # Always verify and compare
  diff -u original-config.json updated-config.json
  
  # Validate JSON format
  jq . updated-config.json > /dev/null && echo "Valid JSON"
  
  # Only proceed if changes look correct
  ```

## Example Workflows

### Initial Setup Check
```bash
# Check available profiles
cat ~/.aws/config | grep -E '^\[profile|^\[default'

# Assume a role
assume <profile-name>

# Check current identity (must use profile)
aws sts get-caller-identity --profile <profile-name>
```

### Finding Available Roles
```bash
# Check AWS configuration files for available profiles
cat ~/.aws/config | grep -E '^\[profile|^\[default'

# List IAM roles in current AWS account (requires appropriate permissions)
aws iam list-roles --profile <profile-name> --query 'Roles[].{RoleName:RoleName,Arn:Arn}'

# Note: 'granted ls' shows help menu, not profiles
# Note: 'assume --list' is not a valid command
```

### Role Switching
```bash
# Switch to specific role (use full profile name)
assume production-readonly

# Verify switch worked (must use --profile flag)
aws sts get-caller-identity --profile production-readonly
```

### Safe Exploration
```bash
# List resources safely (always include --profile)
aws s3 ls --profile <profile-name>
aws ec2 describe-instances --profile <profile-name> --query 'Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType}'
aws lambda list-functions --profile <profile-name> --query 'Functions[].{Name:FunctionName,Runtime:Runtime}'
```

## File Structure
- AWS CLI configuration typically in `~/.aws/config` and `~/.aws/credentials`
- Granted.dev configuration in `~/.granted/config`
- Project-specific configurations should be documented here

## Notes for Claude Code
- Always confirm the AWS account/role before executing commands
- Always use `--profile <profile-name>` with AWS CLI commands after assuming a role
- Provide explanations for complex AWS CLI queries
- Suggest using `--dry-run` flag when available for testing
- Use `--query` parameter to format output for better readability
- Remember that granted.dev configures AWS profiles but doesn't export credentials to environment by default

## Common Commands Reference
```bash
# Identity and region
aws sts get-caller-identity --profile <profile-name>
aws configure get region --profile <profile-name>

# S3
aws s3 ls --profile <profile-name>
aws s3 cp file.txt s3://bucket/ --profile <profile-name>
aws s3 sync ./local-folder s3://bucket/folder/ --profile <profile-name>

# EC2
aws ec2 describe-instances --profile <profile-name>
aws ec2 describe-security-groups --profile <profile-name>
aws ec2 start-instances --instance-ids i-1234567890abcdef0 --profile <profile-name>

# Lambda
aws lambda list-functions --profile <profile-name>
aws lambda invoke --function-name my-function output.txt --profile <profile-name>

# CloudFormation
aws cloudformation list-stacks --profile <profile-name>
aws cloudformation describe-stacks --stack-name my-stack --profile <profile-name>

# AppConfig
aws appconfig list-applications --profile <profile-name> --region <region>
aws appconfig list-environments --application-id <app-id> --profile <profile-name> --region <region>
aws appconfig list-configuration-profiles --application-id <app-id> --profile <profile-name> --region <region>
aws appconfig list-deployments --application-id <app-id> --environment-id <env-id> --profile <profile-name> --region <region>
aws appconfig get-hosted-configuration-version --application-id <app-id> --configuration-profile-id <profile-id> --version-number <version> --profile <profile-name> --region <region> output.json
```

## Environment Variables
When using granted.dev in this setup:
- Environment variables are NOT automatically exported to the shell
- Credentials are stored in AWS config/credentials files
- Use `--profile <profile-name>` flag with all AWS CLI commands
- To export credentials to environment, you may need to use additional granted.dev flags (e.g., `--export-all-env-vars`)