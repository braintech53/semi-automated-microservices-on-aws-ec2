# EC2 Instance Initialization Architecture

This document details the automated instance provisioning process implemented via EC2 user-data scripts. The architecture enables stateless, self-configuring instances that automatically pull code, retrieve secrets, and initialize services without manual intervention.

## Initialization Architecture

### Bootstrap Sequence

1. System Preparation:
   - Package repository updates
   - Core dependency installation
   - Runtime environment setup

2. Application Deployment:
   - GitHub repository cloning
   - Environment configuration retrieval
   - Service dependency installation
   - Process management configuration

## Implementation Example

### Node.js Service Bootstrap

```bash
#!/bin/bash
set -e

# System initialization
apt-get update
apt-get install -y nodejs npm git awscli

# Application directory setup
mkdir -p /opt/service
cd /opt/service

# Clone service repository
git clone https://github.com/org/service.git .
git checkout ${ENVIRONMENT_BRANCH}

# Retrieve environment configuration
aws secretsmanager get-secret-value \
  --secret-id ${SERVICE_ENV_SECRET} \
  --query SecretString \
  --output text > .env

# Install dependencies
npm ci --production

# Configure process management
cat > /etc/systemd/system/service.service << EOF
[Unit]
Description=Node.js Service
After=network.target

[Service]
Type=simple
User=nodejs
WorkingDirectory=/opt/service
ExecStart=/usr/bin/npm start
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Start service
systemctl daemon-reload
systemctl enable service
systemctl start service
```

## Security Implementation

### IAM Configuration

1. Instance Profile Setup:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "secretsmanager:GetSecretValue",
         "Resource": "arn:aws:secretsmanager:*:*:secret:service_env_*"
       }
     ]
   }
   ```

2. Security Considerations:
   - No hardcoded credentials
   - Environment-specific IAM roles
   - Least privilege access

## Service-Specific Implementations

### Docker-based Services

```bash
#!/bin/bash
set -e

# Install Docker
apt-get update
apt-get install -y docker.io
systemctl enable docker

# Configure Docker daemon
cat > /etc/docker/daemon.json << EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
EOF

# Start application container
docker run -d \
  --name service \
  --restart always \
  --env-file /opt/service/.env \
  org/service:latest
```

### Python Services

```bash
#!/bin/bash
set -e

# Install Python environment
apt-get update
apt-get install -y python3-venv python3-pip

# Create virtual environment
python3 -m venv /opt/service/venv
source /opt/service/venv/bin/activate

# Install application
pip install -r requirements.txt

# Configure Gunicorn service
cat > /etc/systemd/system/service.service << EOF
[Unit]
Description=Python Service
After=network.target

[Service]
Type=simple
User=python
WorkingDirectory=/opt/service
EnvironmentFile=/opt/service/.env
ExecStart=/opt/service/venv/bin/gunicorn app:app
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```