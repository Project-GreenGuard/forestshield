# ForestShield - Staging Environment and CI/CD Pipeline Guide

## Purpose

This document outlines the recommended approach for setting up a staging environment and continuous integration/continuous deployment (CI/CD) pipeline for the ForestShield project. This guide is intended for future implementation when the project is ready for automated deployment workflows.

## Overview

A staging environment and CI/CD pipeline will enable:

- Automated testing before production deployment
- Consistent deployment processes
- Reduced manual errors
- Faster iteration cycles
- Better collaboration between team members

## Staging Environment Architecture

### Environment Structure

The project will use three distinct environments:

1. **Development Environment**

   - Local development using Docker Compose
   - Individual developer machines
   - No AWS resources required

2. **Staging Environment**

   - AWS infrastructure mirroring production
   - Used for integration testing and validation
   - Separate AWS account or resources with staging prefix

3. **Production Environment**
   - Live AWS infrastructure
   - Production data and services
   - Highest availability and security requirements

### Staging Environment Components

**AWS Resources:**

- **IoT Core**: Staging IoT Core endpoint for device testing
- **Lambda Functions**: Separate staging Lambda functions with staging suffix
- **DynamoDB**: Staging table (e.g., `WildfireSensorData-Staging`)
- **API Gateway**: Staging API Gateway with separate stage
- **IAM Roles**: Staging-specific IAM roles with limited permissions

**Naming Conventions:**

- Resource names should include `-staging` suffix
- Example: `wildfire-process-sensor-data-staging`
- DynamoDB table: `WildfireSensorData-Staging`
- API Gateway stage: `staging`

**Configuration Management:**

- Use environment variables for environment-specific settings
- Terraform variables for environment selection
- Separate Terraform workspaces or state files per environment

## CI/CD Pipeline Architecture

### Pipeline Stages

The CI/CD pipeline will consist of the following stages:

1. **Source Control**

   - Git repository (GitHub)
   - Branch protection rules
   - Pull request requirements

2. **Build Stage**

   - Package Lambda functions
   - Build frontend application
   - Run unit tests
   - Generate deployment artifacts

3. **Test Stage**

   - Run integration tests
   - Execute linting and code quality checks
   - Security scanning
   - Performance testing

4. **Deploy to Staging**

   - Deploy infrastructure using Terraform
   - Deploy Lambda functions
   - Deploy frontend to staging environment
   - Run smoke tests

5. **Staging Validation**

   - Manual testing and approval
   - Automated end-to-end tests
   - Performance validation
   - Security review

6. **Deploy to Production**
   - Deploy to production environment
   - Run production smoke tests
   - Monitor deployment health

### CI/CD Tool Options

**Option 1: GitHub Actions (Recommended)**

- Native GitHub integration
- Free for public repositories
- YAML-based configuration
- Built-in secrets management

**Option 2: AWS CodePipeline**

- Native AWS integration
- Integrated with other AWS services
- Visual pipeline editor
- Cost-effective for AWS-heavy projects

**Option 3: GitLab CI/CD**

- Integrated with GitLab
- Comprehensive CI/CD features
- Self-hosted or cloud options

## Implementation Plan

### Phase 1: Staging Environment Setup

1. **Create Staging Terraform Configuration**

   - Duplicate production Terraform files
   - Add environment-specific variables
   - Configure staging resource names
   - Set up separate Terraform state backend

2. **Configure Staging AWS Resources**

   - Create staging DynamoDB table
   - Deploy staging Lambda functions
   - Set up staging API Gateway
   - Configure staging IoT Core

3. **Environment-Specific Configuration**
   - Create staging environment variables
   - Configure staging API endpoints
   - Set up staging database
   - Configure staging monitoring

### Phase 2: CI/CD Pipeline Setup

1. **GitHub Actions Workflow (Recommended)**

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Staging

on:
  push:
    branches: [develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          cd forestshield-backend
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest tests/

      - name: Package Lambda functions
        run: |
          cd forestshield-backend/lambda-processing
          zip -r lambda-processing.zip .
          cd ../api-gateway-lambda
          zip -r api-gateway-lambda.zip .

  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    steps:
      - uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_STAGING }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_STAGING }}
          aws-region: us-east-1

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: |
          cd forestshield-infrastructure
          terraform init

      - name: Terraform Plan
        run: |
          cd forestshield-infrastructure
          terraform plan -var="environment=staging"

      - name: Terraform Apply
        run: |
          cd forestshield-infrastructure
          terraform apply -auto-approve -var="environment=staging"
```

2. **Frontend Deployment Workflow**

```yaml
name: Deploy Frontend

on:
  push:
    branches: [develop, main]

jobs:
  build-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install dependencies
        run: |
          cd forestshield-frontend
          npm install

      - name: Build
        run: |
          cd forestshield-frontend
          npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.STAGING_API_URL }}

      - name: Deploy to S3
        if: github.ref == 'refs/heads/develop'
        run: |
          aws s3 sync forestshield-frontend/build s3://forestshield-staging-frontend
```

### Phase 3: Testing Strategy

1. **Unit Tests**

   - Lambda function unit tests
   - Frontend component tests
   - API handler tests

2. **Integration Tests**

   - End-to-end API tests
   - Database integration tests
   - IoT message flow tests

3. **Smoke Tests**
   - Health check endpoints
   - Basic functionality verification
   - Deployment validation

## Branch Strategy

### Recommended Git Workflow

1. **Main Branch**

   - Production-ready code
   - Protected branch
   - Requires pull request approval
   - Triggers production deployment

2. **Develop Branch**

   - Integration branch for features
   - Triggers staging deployment
   - Used for testing before production

3. **Feature Branches**

   - Individual feature development
   - Created from develop
   - Merged back to develop via pull request

4. **Hotfix Branches**
   - Critical production fixes
   - Created from main
   - Merged to both main and develop

## Environment Variables Management

### Staging Environment Variables

Store in GitHub Secrets or AWS Systems Manager Parameter Store:

- `AWS_ACCESS_KEY_ID_STAGING`
- `AWS_SECRET_ACCESS_KEY_STAGING`
- `STAGING_API_URL`
- `STAGING_DYNAMODB_TABLE`
- `STAGING_IOT_ENDPOINT`

### Terraform Variables

Create `terraform.tfvars.staging`:

```hcl
environment = "staging"
aws_region  = "us-east-1"

# Staging-specific overrides
dynamodb_table_name = "WildfireSensorData-Staging"
api_gateway_stage   = "staging"

# Resource naming
resource_prefix = "forestshield-staging"
```

## Monitoring and Alerts

### Staging Environment Monitoring

- CloudWatch alarms for Lambda errors
- API Gateway metrics and logs
- DynamoDB performance metrics
- Cost monitoring and alerts

### Deployment Notifications

- Slack notifications for deployments
- Email alerts for failed deployments
- GitHub status checks for pipeline results

## Security Considerations

### Staging Environment Security

- Separate AWS IAM roles for staging
- Limited permissions following least privilege
- No production data in staging
- Secure secrets management
- Regular security scanning

### CI/CD Security

- Secure credential storage (GitHub Secrets, AWS Secrets Manager)
- Code signing for deployments
- Dependency vulnerability scanning
- Infrastructure as Code security scanning

## Cost Management

### Staging Environment Costs

- Use smaller instance sizes where possible
- Implement auto-scaling with lower thresholds
- Use on-demand pricing for DynamoDB
- Monitor and set budget alerts
- Consider shutting down staging during off-hours

### CI/CD Costs

- GitHub Actions: Free for public repos, limited minutes for private
- AWS CodePipeline: Pay per pipeline execution
- Consider cost optimization strategies

## Rollback Procedures

### Automated Rollback

- Terraform state management for infrastructure rollback
- Lambda function versioning for code rollback
- API Gateway deployment rollback capabilities
- Frontend deployment versioning

### Manual Rollback Steps

1. Identify the deployment to roll back to
2. Revert code changes in Git
3. Trigger deployment pipeline
4. Verify rollback success
5. Document rollback reason

## Future Enhancements

### Advanced CI/CD Features

- Blue-green deployments
- Canary deployments
- Automated performance testing
- Load testing in staging
- Automated security scanning
- Infrastructure drift detection

### Monitoring and Observability

- Application performance monitoring (APM)
- Distributed tracing
- Log aggregation and analysis
- Real-time alerting
- Dashboard for deployment metrics

## Implementation Checklist

### Staging Environment

- [ ] Create staging Terraform configuration
- [ ] Set up staging AWS resources
- [ ] Configure staging environment variables
- [ ] Test staging deployment manually
- [ ] Document staging access procedures

### CI/CD Pipeline

- [ ] Set up GitHub Actions workflows
- [ ] Configure AWS credentials for CI/CD
- [ ] Create build and test jobs
- [ ] Implement deployment automation
- [ ] Set up notifications
- [ ] Test pipeline end-to-end

### Testing

- [ ] Write unit tests for all components
- [ ] Create integration test suite
- [ ] Implement smoke tests
- [ ] Set up test data management
- [ ] Configure test reporting

### Documentation

- [ ] Document deployment procedures
- [ ] Create runbooks for common issues
- [ ] Document rollback procedures
- [ ] Update team on new processes

## References

- AWS CodePipeline Documentation: https://docs.aws.amazon.com/codepipeline/
- GitHub Actions Documentation: https://docs.github.com/en/actions
- Terraform Workspaces: https://www.terraform.io/docs/language/state/workspaces.html
- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/

---

**Document Version:** 1.0  
**Last Updated:** Current Semester  
**Status:** Future Implementation  
**Project:** Project GreenGuard - ForestShield
