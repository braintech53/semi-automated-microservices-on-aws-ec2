Title: User Journey: AI-Powered Assistant Built on Modular, Scalable AWS EC2 Infrastructure

👤 Persona: SaaS End User
A modern professional subscribes to a personal AI assistant platform. After signing up on the landing page, they integrate their Gmail and Google Calendar accounts, connect their personal Telegram bot, and start using the service.

🌐 How the User Interacts
They sign up via the web dashboard

Configure their AI assistant preferences (OpenAI, Ollama, or OpenRouter)

Link their personal Telegram bot

Connect Gmail and Calendar

Start chatting with the assistant via Telegram

Example messages:

“Remind me to send the report at 9 PM”
“What meetings do I have tomorrow?”
“Summarize my last 5 emails”

⚙️ What Happens in the Background
Telegram Bot receives the message (via webhook)

Message is passed to MCP (Master Control Program) — the brain of the system

MCP uses the user’s preferred AI model to understand the request

Based on the request:

If it's email-related → MCP calls the Gmail Service

If it’s about events → MCP calls the Calendar Service

If it's a reminder → MCP calls the Reminder Service to schedule it

The relevant microservice responds, and MCP sends the result back via the Telegram Bot

MongoDB stores user data, preferences, and system prompts for future context

Reminder Service constantly checks upcoming reminders from calendar and triggers the Telegram Bot at the right time

🧱 Architecture Highlights (How It’s Built)
Each microservice (e.g., Auth, Gmail, Calendar, Reminder, Telegram, MCP, AI Proxy) runs on its own dedicated EC2 instance

Every environment (test, staging, production) has its own isolated infrastructure with unique URLs

Horizontal auto-scaling is implemented per microservice, so only the EC2 instance that receives high demand (like Gmail) will scale

Environment-based deployments use GitHub branches (test, stage, prod) — no CI/CD used in this setup

EC2 instances automatically pull source code and secrets from GitHub and AWS Secrets Manager during launch

Secure credential management is handled via AWS Secrets Manager — no hardcoded secrets

Deployment and scaling are done through Launch Templates, Auto Scaling Groups (ASG), and instance metadata without relying on Jenkins or GitHub Actions

📘 Why This Architecture Was Created
This setup demonstrates how to manage a modern SaaS backend using:

Modular microservices

Environment-based separation

Auto-scalable EC2 infrastructure

Secure, GitHub-based code delivery

Minimal automation with strong infrastructure control

It’s ideal for teams who want to understand how to structure microservices on AWS without immediately relying on full CI/CD pipelines or external DevOps tools.

The idea is not to replace CI/CD or orchestration platforms — but to provide a semi-automated, scalable approach that handles:

Deployment flexibility

Horizontal scaling

Microservice isolation

Secure configuration

Real-world AI-driven workflows