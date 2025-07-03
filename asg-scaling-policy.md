📘 Title: Auto Scaling Policy – Per-Microservice Horizontal Scaling with Traffic-Aware Strategy

This document outlines the auto scaling group (ASG) strategy used to horizontally scale microservices based on workload, without involving traditional DevOps tools like ECS, CI/CD pipelines, or service meshes. Each microservice is deployed on its own EC2 instance and auto-scaled independently based on metrics like CPU utilization or request count.

🎯 Goal of This Strategy
✅ Optimize cost by scaling only the services under load
✅ Maintain performance by ensuring services like Gmail or AI Proxy don’t bottleneck
✅ Keep deployment simple: no CI/CD, no Kubernetes — only EC2 + Launch Template + ASG

🧩 Per-Service Auto Scaling Groups (ASGs)
Each microservice runs in its own Auto Scaling Group with its own metrics and Launch Template.

| Service       | Min Instances | Max Instances | Metric           | Trigger Threshold  |
| ------------- | ------------- | ------------- | ---------------- | ------------------ |
| Auth Service  | 1             | 2             | CPU Utilization  | > 70% (5 mins)     |
| Gmail Service | 1             | 5             | Network In + CPU | > 65% or > 200 RPS |
| Calendar      | 1             | 3             | Network In       | > 60%              |
| AI Proxy      | 1             | 6             | CPU Utilization  | > 70% (2 mins)     |
| MCP           | 1             | 1             | —                | Fixed              |
| Telegram Bot  | 1             | 2             | Network In       | > 50%              |
| Reminder      | 1             | 2             | Memory           | > 400MB (optional) |
| Admin Panel   | 1             | 1             | —                | Fixed              |


🛑 MCP and Admin Panel are not auto-scaled due to:

Critical coordination logic (MCP)

Low load expectations (Admin Panel)

⚙️ Components Used
✅ Launch Templates with:

GitHub branch boot logic

Secrets pull from AWS Secrets Manager

✅ Auto Scaling Group

Warm-up time: 60–90s

Grace period: 300s for boot + service up

✅ CloudWatch Alarms

CPU Utilization, NetworkIn, RequestCount

✅ SNS Alerts (Optional)

Notify when new instance added or terminated

🔁 Deployment Refresh Strategy
When new code is merged into a branch (stage or prod):

ASG is updated to use new launch template version

Existing EC2s are terminated one-by-one (rolling replace)

Each new instance boots with new source code automatically

Secrets and configuration are pulled fresh each time

🛡️ Scaling Cooldowns & Anti-Flap Control
Scale-out cooldown: 300 seconds

Scale-in cooldown: 600 seconds

Warm-up period defined in CloudWatch metric math to avoid premature scale-ins

🧪 Testing Auto Scaling
Use Apache Benchmark (ab) or Locust to simulate API traffic on individual services and observe CloudWatch metric graphs and instance count behavior.

Example:

ab -n 1000 -c 50 http://your-api.com/gmail/emails
