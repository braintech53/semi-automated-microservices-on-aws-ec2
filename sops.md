# Standard Operating Procedures

This document outlines the operational procedures for managing the EC2-hosted microservice architecture, including deployment protocols, incident response, maintenance operations, and system validation procedures.

## Deployment Procedures

### Service Deployment Protocol

1. Pre-deployment Validation:
   - Verify branch alignment with environment
   - Validate AWS credentials and permissions
   - Confirm target ASG configuration

2. Deployment Sequence:
   ```bash
   # Update Launch Template
   aws ec2 create-launch-template-version \
     --launch-template-id lt-123 \
     --version-description "Deploy v1.2.3" \
     --source-version 1

   # Update ASG
   aws autoscaling update-auto-scaling-group \
     --auto-scaling-group-name service-asg \
     --launch-template LaunchTemplateId=lt-123,Version='$Latest'
   ```

3. Post-deployment Validation:
   - Monitor instance health checks
   - Verify service endpoints
   - Validate metrics and logs

## Incident Response

### Service Recovery Protocol

1. Failure Detection:
   - Monitor CloudWatch alarms
   - Check service health endpoints
   - Review error logs

2. Recovery Actions:
   ```bash
   # Revert to previous Launch Template version
   aws autoscaling update-auto-scaling-group \
     --auto-scaling-group-name service-asg \
     --launch-template LaunchTemplateId=lt-123,Version='1'

   # Force instance refresh
   aws autoscaling start-instance-refresh \
     --auto-scaling-group-name service-asg \
     --strategy Rolling
   ```

## Maintenance Operations

### Secret Rotation Procedure

1. AWS Secrets Manager:
   ```bash
   # Create new secret version
   aws secretsmanager update-secret \
     --secret-id service_env_prod \
     --secret-string file://new-secret.json

   # Verify secret access
   aws secretsmanager get-secret-value \
     --secret-id service_env_prod \
     --version-stage AWSCURRENT
   ```

2. Service Redeployment:
   - Update Launch Template version
   - Initiate rolling instance refresh

### Auto Scaling Validation

1. Scale-out Testing:
   ```bash
   # Simulate high CPU load
   stress --cpu 8 --timeout 300s

   # Monitor ASG events
   aws autoscaling describe-scaling-activities \
     --auto-scaling-group-name service-asg
   ```

2. Scale-in Verification:
   - Reduce load simulation
   - Confirm instance termination
   - Verify service stability

## System Maintenance

### MongoDB Disk Management

1. Space Monitoring:
   ```bash
   # Check disk usage
   df -h /data/mongodb

   # Monitor database size
   mongo --eval "db.stats()"
   ```

2. Cleanup Operations:
   - Archive old data
   - Compact collections
   - Verify replication status

## Environment Promotion

### Promotion Workflow

1. Stage to Production:
   - Tag stage branch commit
   - Create production merge PR
   - Update Launch Template
   - Deploy to production ASG

2. Validation Steps:
   - Verify service health
   - Check metrics baseline
   - Confirm data integrity

## Debug Procedures

### Service Debugging

1. Log Analysis:
   ```bash
   # Fetch service logs
   journalctl -u service-name -n 1000 --no-pager

   # Check nginx access logs
   tail -f /var/log/nginx/access.log
   ```

2. Performance Analysis:
   - Monitor CPU/memory usage
   - Check network connectivity
   - Verify DNS resolution

## Backup and Restore

### Backup Protocol

1. MongoDB Backup:
   ```bash
   # Create backup
   mongodump --uri mongodb://localhost:27017/db \
     --out /backup/$(date +%Y%m%d)

   # Verify backup
   ls -lh /backup/$(date +%Y%m%d)
   ```

2. Restore Procedure:
   - Stop affected service
   - Restore from backup
   - Verify data integrity