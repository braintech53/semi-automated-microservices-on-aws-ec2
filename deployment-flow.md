📘 Title: Deployment Workflow — Environment Branching and Manual Infra Rollout (No CI/CD Tools)

This document explains how deployments are managed in this architecture without using any CI/CD pipelines or DevOps tools. It relies entirely on GitHub branches, secure EC2 provisioning, and structured environments (test, stage, production) — all within AWS.

🌍 Environments
Environment	Purpose	Branch Used	EC2 Type
| Environment | Purpose                  | Branch Used | EC2 Type  |
| ----------- | ------------------------ | ----------- | --------- |
| test        | Feature validation       | test        | t2.micro  |
| stage       | UAT / Pre-production     | stage       | t3.medium |
| prod        | Live production services | prod        | t3.medium |


Each environment is mapped to a dedicated set of EC2 instances (1 per microservice)

Admin panel, MongoDB, and shared layers follow same environment separation

🔀 Git Branch-Based Control
Git Branch	Auto Pull Behavior
| Git Branch | Auto Pull Behavior                              |
| ---------- | ----------------------------------------------- |
| test       | Pulled during test EC2 creation (via user-data) |
| stage      | Manually deployed after testing is complete     |
| prod       | Manually deployed after staging approval        |


🔒 Secrets (e.g., API keys, Mongo URI) are never stored in the codebase.

🔁 Deployment Process
Developer merges feature into the appropriate environment branch (e.g., test)

AWS engineer spins up or replaces the EC2 instance for that service

EC2 boot sequence (via user-data) performs:

Pull source from GitHub branch

Load .env from AWS Secrets Manager

Install dependencies

Start service (via PM2, systemd, or native process)

Once validated in test:

Code is merged to stage branch

New EC2s are provisioned or replaced in the staging environment

Manual validation occurs

Once approved:

Code is merged to prod

Production EC2s are scaled/updated one-by-one to ensure zero downtime

🧠 Intelligent EC2 Boot Logic (User Data Script)
Each EC2 instance uses a launch template that runs a startup shell script (user-data), which:

Pulls correct GitHub branch (based on launch config)

Fetches .env securely from AWS Secrets Manager

Runs npm install, builds services (if needed)

Logs output to CloudWatch Logs or local file

Starts the server

Example command:

git clone -b stage https://github.com/your-org/service.git && cd service
aws secretsmanager get-secret-value --secret-id calendar_env --query SecretString --output text > .env
npm install && npm run start

🚫 What’s Not Used
No Jenkins, GitHub Actions, or DevOps pipelines

No ECS, EKS, or Fargate

No Lambda-based deployment logic

This project intentionally demonstrates how reliable, testable, and multi-environment infrastructure can be built using only EC2 and GitHub — a valuable model for teams transitioning toward DevOps maturity or operating with lean automation.