# AWS CloudFormation CI/CD Pipeline

This repository contains an Infrastructure as Code (IaC) solution using AWS CloudFormation with a complete CI/CD pipeline. The pipeline automatically validates and deploys infrastructure changes when templates are updated.

## Architecture

### Infrastructure Components (Nested Stack)
- VPC with public and private subnets
- Network components (Internet Gateway, NAT Gateway, Route Tables)
- Security Groups for EC2 instances
- EC2 instances in public and private subnets

### CI/CD Pipeline Components
- GitHub: Source control
- CodeBuild: Template validation (cfn-lint + taskcat)
- CloudFormation: Infrastructure deployment

## Prerequisites

1. AWS CLI installed and configured
2. An AWS account with appropriate permissions
3. Git installed locally
4. An EC2 key pair in your AWS account
5. A GitHub account and repository
6. A GitHub Personal Access Token with repo access

## Setup Instructions

### 1. Create GitHub Repository

1. Create a new repository on GitHub
2. Name it `infrastructure-repo` (or your preferred name)
3. Make it private or public as per your requirements
4. Don't initialize it with any files

### 2. Generate GitHub Personal Access Token

1. Go to GitHub Settings -> Developer Settings -> Personal Access Tokens -> Tokens (classic)
2. Generate a new token with the following permissions:
   - `repo` (Full control of private repositories)
   - `admin:repo_hook` (Full control of repository hooks)
3. Copy the token and keep it secure (you'll need it for the pipeline setup)

### 3. Clone and Set Up Repository

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/infrastructure-repo.git
cd infrastructure-repo

# Copy your infrastructure files to the repository
mkdir Task2
cp -r /path/to/your/templates/* Task2/

# Initial commit
git add .
git commit -m "Initial commit with infrastructure templates"
git push
```

### 4. Deploy the Pipeline

```bash
# Get your current IP address for the AllowedIP parameter
curl https://checkip.amazonaws.com

# Deploy the pipeline stack
aws cloudformation create-stack \
  --stack-name infrastructure-pipeline \
  --template-body file://Task2/pipeline-stack.yaml \
  --parameters \
    ParameterKey=GitHubOwner,ParameterValue=YOUR_GITHUB_USERNAME \
    ParameterKey=GitHubRepo,ParameterValue=infrastructure-repo \
    ParameterKey=GitHubBranch,ParameterValue=main \
    ParameterKey=GitHubToken,ParameterValue=YOUR_GITHUB_TOKEN \
    ParameterKey=AllowedIP,ParameterValue=$(curl -s https://checkip.amazonaws.com)/32 \
    ParameterKey=KeyName,ParameterValue=your-key-pair-name \
  --capabilities CAPABILITY_IAM

# Monitor the stack creation
aws cloudformation describe-stacks --stack-name infrastructure-pipeline
```

## Pipeline Workflow

1. **Source Stage**:
   - Monitors changes in the GitHub repository
   - Triggers pipeline when changes are pushed

2. **Validation Stage**:
   - Runs `cfn-lint` on all templates
   - Executes `taskcat` tests
   - Fails the pipeline if validation errors are found

3. **Deploy Stage**:
   - Deploys the nested stack using CloudFormation
   - Creates/updates all infrastructure components

## Making Changes

1. Clone the repository if you haven't already:
```bash
git clone https://github.com/YOUR_USERNAME/infrastructure-repo.git
cd infrastructure-repo
```

2. Make changes to templates in the `Task2` folder:
   - `root-stack.yaml`: Main template
   - `modules/*.yaml`: Individual component templates

3. Commit and push changes:
```bash
git add .
git commit -m "Description of changes"
git push
```

4. The pipeline will automatically:
   - Detect the changes
   - Validate the templates
   - Deploy the updates if validation passes

## Monitoring

### Pipeline Status
```bash
# Get pipeline execution status
aws codepipeline get-pipeline-state --name infrastructure-pipeline

# Get CloudFormation stack status
aws cloudformation describe-stacks --stack-name infrastructure-stack
```

### Logs and Troubleshooting
- Pipeline execution details: AWS Console -> CodePipeline
- Build logs: AWS Console -> CodeBuild
- Deployment logs: AWS Console -> CloudFormation

## Cleanup

To delete all resources:

1. Delete the infrastructure stack:
```bash
aws cloudformation delete-stack --stack-name infrastructure-stack
```

2. Delete the pipeline stack:
```bash
aws cloudformation delete-stack --stack-name infrastructure-pipeline
```

3. Delete the GitHub repository:
   - Go to GitHub repository settings
   - Scroll to the bottom to the "Danger Zone"
   - Click "Delete this repository"
   - Follow the confirmation steps

## Security Notes

- The pipeline uses IAM roles with necessary permissions
- EC2 instances are secured with security groups
- SSH access is restricted to your IP address
- Private instances are only accessible through the public instance
- GitHub token is stored securely in AWS Systems Manager Parameter Store

## Template Structure

```
Task2/
├── pipeline-stack.yaml     # CI/CD pipeline definition
├── root-stack.yaml        # Main infrastructure template
├── .taskcat.yml          # Taskcat configuration
└── modules/
    ├── vpc-module.yaml
    ├── network-module.yaml
    ├── security-module.yaml
    └── compute-module.yaml
``` 