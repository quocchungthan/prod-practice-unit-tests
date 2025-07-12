# Production Deployment Guide

This directory contains the production deployment configuration for the PDF-to-EPUB application using AWS ECR and Docker Compose.

## Prerequisites

1. **AWS Account** with ECR permissions
2. **Docker** installed and running
3. **Git** installed
4. **AWS CLI** configured with proper credentials

## AWS Configuration

### Step 1: Install and Configure AWS CLI

Install and configure AWS CLI:

**Linux/Mac:**
```bash
# Install AWS CLI (if not already installed)
# Linux/Mac: pip install awscli

# Configure AWS credentials
aws configure
```

**Windows:**
```cmd
# Install AWS CLI (if not already installed)
# Download from https://aws.amazon.com/cli/
# Run the installer and follow the prompts

# Configure AWS credentials
aws configure
```

Enter your:
- AWS Access Key ID
- AWS Secret Access Key  
- Default region (e.g., us-east-1)
- Default output format (json)

### Step 2: Verify AWS Authentication

Test your AWS configuration:

**Linux/Mac:**
```bash
# Test AWS CLI configuration
aws sts get-caller-identity

# Test ECR access
aws ecr describe-repositories --region us-east-1
```

**Windows:**
```cmd
# Test AWS CLI configuration
aws sts get-caller-identity

# Test ECR access
aws ecr describe-repositories --region us-east-1
```

### Step 3: Prepare Production Server

Install required software on your production server:

**Linux:**
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Git
sudo apt-get update
sudo apt-get install git

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Windows:**
```cmd
# Install Docker Desktop
# Download from https://www.docker.com/products/docker-desktop/
# Run the installer and follow the prompts

# Install Git
# Download from https://git-scm.com/download/win
# Run the installer and follow the prompts

# Install AWS CLI
# Download from https://aws.amazon.com/cli/
# Run the installer and follow the prompts
```

## Initial Setup

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd prod-practice-unit-tests
```

### 2. Configure Environment

Copy the environment example file and configure it for production:

```bash
# Copy environment template
cp ../practice-unit-tests/env.example .env

# Edit the environment file
nano .env
```

### 3. Required Environment Variables

Update your `.env` file with the following production values:

```bash
# AWS Configuration
AWS_ACCOUNT_ID=123456789012
AWS_REGION=us-east-1

# Application Configuration
APP_ADDRESS=your-domain.com
MAX_FILE_SIZE_KB=10240

# OpenAI Configuration
OPENAI_APIKEY=your-openai-api-key
OPENAI_PROJ_ID=your-openai-project-id
OPENAI_ORG_ID=your-openai-org-id
OPENAI_MODEL_NAME=gpt-4

# Gmail Configuration
GMAIL_USER=your-gmail@gmail.com
GMAIL_PASS=your-gmail-app-password
```

## Deployment Options

### Option 1: Deploy with Version Detection (Recommended)

### Option 2: Manual Deployment

Deploy with a specific version:

```bash
# Deploy specific version
VERSION=1.2.3 docker-compose up -d

# Deploy latest version
VERSION=latest docker-compose up -d

# Deploy without specifying version (uses latest)
docker-compose up -d
```

### Option 3: Deploy from Windows

```cmd
# Navigate to scripts directory
cd ..\practice-unit-tests\script

# Deploy with automatic version detection
deploy_with_version.bat
```

## Service Architecture

The production deployment includes 8 services:

1. **application** - React frontend with Nginx (Ports: 80, 443)
2. **bff** - Backend for Frontend API (Port: 3001)
3. **pdf2markdown** - PDF to Markdown conversion
4. **translationservice** - OpenAI translation service
5. **reconstructionservice** - EPUB reconstruction
6. **emailservice** - Email notifications
7. **contentbreakdown** - Content analysis
8. **composingservice** - EPUB composition

## Monitoring and Management

### Check Service Status

```bash
# View all running services
docker-compose ps

# View service logs
docker-compose logs

# View specific service logs
docker-compose logs application
docker-compose logs bff
```

### Scale Services

```bash
# Scale translation service to 3 instances
docker-compose up -d --scale translationservice=3
```

### Stop Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

## Troubleshooting

### Common Issues

1. **Environment Variables Missing**
   ```bash
   # Check if .env file exists and has all required variables
   cat .env
   ```

2. **ECR Images Not Found**
   ```bash
   # Verify images exist in ECR
   aws ecr describe-images --repository-name application --region us-east-1
   ```

3. **Service Not Starting**
   ```bash
   # Check service logs
   docker-compose logs <service-name>
   ```

4. **Port Conflicts**
   ```bash
   # Check what's using the ports
   netstat -tulpn | grep :80
   netstat -tulpn | grep :443
   ```

5. **ECR Login Failed**
   ```bash
   # Reconfigure AWS credentials
   aws configure
   aws ecr get-login-password --region us-east-1
   ```

### Logs and Debugging

```bash
# Follow logs in real-time
docker-compose logs -f

# View logs for specific service
docker-compose logs -f application

# Check container status
docker ps
docker ps -a
```

## Security Considerations

1. **Environment Variables**: Never commit `.env` files to version control
2. **ECR Access**: Use IAM roles instead of access keys when possible
3. **Network Security**: Configure firewalls and security groups
4. **SSL/TLS**: Set up HTTPS for production deployments

## Backup and Recovery

### Backup Data

```bash
# Create backup directory
mkdir -p backups/$(date +%Y%m%d_%H%M%S)

# Backup storage directory
cp -r storage backups/$(date +%Y%m%d_%H%M%S)/

# Backup environment file
cp .env backups/$(date +%Y%m%d_%H%M%S)/
```

### Restore Data

```bash
# Restore from backup
cp -r backups/20241201_143022/storage ./
cp backups/20241201_143022/.env ./
```

## Complete Deployment Workflow

### 1. Initial Setup (One-time)
```bash
# Clone repository
git clone <your-repo-url>
cd prod-practice-unit-tests

# Configure environment
cp ../practice-unit-tests/env.example .env
nano .env  # Edit with production values
```

### 2. Build and Push Images (Development Machine)
```bash
# Navigate to development repository
cd ../practice-unit-tests

# Build and push with version detection
cd script
./build_and_push_to_ecr.sh
```

### 3. Deploy (Production Server)
```bash
# Navigate to production directory
cd prod-practice-unit-tests

# Deploy with version detection
cd ../practice-unit-tests/script
./deploy_with_version.sh
```

### 4. Verify Deployment
```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs

# Test application
curl http://localhost
```

## Support

For additional deployment options and advanced configurations, see:
- `../practice-unit-tests/AWS_ECS_EFS_DEPLOYMENT.md` - ECS deployment with shared storage
- `../practice-unit-tests/script/README.md` - Additional utility scripts
