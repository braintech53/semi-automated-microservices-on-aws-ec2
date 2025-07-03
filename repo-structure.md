📘 Title: Repository Structure – Infrastructure Reference for AWS EC2-Based Microservices Deployment (No Code or Templates Included)

—

🧭 Overview

This repository (semi-automated-microservices-on-aws-ec2) is a DevOps documentation and infrastructure design reference for horizontally scaled, environment-aware microservices running on AWS EC2.

It does not contain:

Any microservice source code

Any EC2 launch template JSONs

Any user-data bootstrap scripts

Any actual infrastructure-as-code (e.g. Terraform, CloudFormation)

Instead, this repo serves as a knowledge base and architectural showcase for how a fully functional, scalable, secure, and testable EC2-based deployment can be structured and operated manually or semi-automatically.

—

📦 Purpose of This Repository

Present a real-world microservices deployment blueprint

Document all major DevOps aspects: environments, scaling, secrets, backups

Showcase portfolio-ready expertise in architecture and cloud engineering

Provide reusable ideas and structure for future automation (Terraform, Jenkins, etc.)

—

🗂️ Directory & File Structure

semi-automated-microservices-on-aws-ec2/
├── README.md # Summary of project purpose, architecture intent, and features
├── architecture.md # EC2 infrastructure, VPC layout, service routing design
├── architecture.png # Visual architecture diagram
├── drawio-architecture.xml # Editable draw.io source for architecture diagram
├── security.md # VPC, subnets, SGs, IAM roles, and least-privilege strategies
├── environment-config.md # Config separation for test, stage, and production
├── cost-model.md # High-level cost estimate of infrastructure (by environment & usage)
├── monitoring-observability.md # Prometheus, Grafana, Node Exporter, Blackbox strategies
├── sops.md # Standard Operating Procedures: deployments, failovers, secrets
├── repo-structure.md # (This file)
└── future-roadmap.md # Optional: Future automation plans, Jenkins/GitHub Actions, IaC strategy

—

🧾 What This Repository Does Not Include

No source code for any microservice (auth, calendar, ai-proxy, etc.)

No EC2 launch templates or shell scripts

No .env files, Dockerfiles, or build configs

No production secrets or environment tokens

No Terraform, Ansible, or CloudFormation templates (yet)

This is a concept-level repository designed to demonstrate DevOps architecture capability.

—

🎯 Naming & Branching Conventions (External to This Repo)

Repositories: One per microservice (e.g., calendar-service, auth-service)

Branches: test, stage, prod (per repo)

Secrets Manager Keys: {service}env{env}

EC2 Tags:

Name: auth-service-prod

ENV: prod

OWNER: manthan

—

📘 Usage Guidelines

You can use this repo as:

A public proof of infrastructure design knowledge

A DevOps portfolio piece without exposing source code

A knowledge base or onboarding reference for teammates or recruiters

A starter blueprint to build fully automated Terraform-based infrastructure later