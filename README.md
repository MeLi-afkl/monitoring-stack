📊 Infrastructure Monitoring Stack



A fully containerized monitoring and alerting stack built with Prometheus, Grafana, Node Exporter, Alertmanager, Docker Compose, and GitHub Actions.

The project collects system-level metrics, visualizes infrastructure performance through a custom Grafana dashboard, evaluates alerting rules, and sends Discord notifications when predefined conditions are triggered.

Grafana data sources and dashboards are automatically provisioned from configuration files, making the environment reproducible with a single Docker Compose deployment.

🎯 Project Overview

This project demonstrates a practical DevOps monitoring and observability workflow.

It includes:

Real-time infrastructure monitoring

Historical metric visualization

Custom Grafana dashboard

Prometheus alerting rules

Alertmanager routing

Discord alert notifications

Grafana provisioning as code

Persistent Docker volumes

Secret handling outside source control

Automated configuration validation with GitHub Actions

🏗️ Architecture

                   ┌─────────────────┐
                   │  Node Exporter  │
                   │      :9100      │
                   └────────┬────────┘
                            │
                            │ metrics
                            ▼
                   ┌─────────────────┐
                   │   Prometheus    │
                   │      :9090      │
                   └────────┬────────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 │ metrics             │ alerts
                 ▼                     ▼
        ┌────────────────┐    ┌─────────────────┐
        │    Grafana     │    │  Alertmanager   │
        │     :3000      │    │      :9093      │
        └────────────────┘    └────────┬────────┘
                                      │
                                      │ notifications
                                      ▼
                                ┌─────────────┐
                                │   Discord   │
                                └─────────────┘

Monitoring flow

Node Exporter exposes system-level metrics.

Prometheus scrapes and stores those metrics.

Grafana queries Prometheus and visualizes current and historical values.

Prometheus alert rules continuously evaluate infrastructure conditions.

When a rule enters a firing state, Prometheus forwards the alert to Alertmanager.

Alertmanager routes the notification to Discord.

🛠️ Tech Stack

Technology

Purpose

Docker

Containerization

Docker Compose

Multi-container orchestration

Prometheus

Metrics collection and time-series storage

PromQL

Metric queries and alert expressions

Node Exporter

System-level metric exporter

Grafana

Dashboards and visualization

Alertmanager

Alert processing and routing

Discord Webhooks

Alert notifications

GitHub Actions

Continuous Integration

YAML / JSON

Configuration and dashboard provisioning

📈 Monitoring Dashboard

The custom Grafana dashboard displays both current values and historical trends.

Metrics monitored

CPU utilization

Memory utilization

Disk utilization

Disk I/O

Network receive traffic

Network transmit traffic

Filesystem utilization

System uptime

Dashboard Preview



🚨 Alerting

Prometheus evaluates alert rules defined in alerts.yml.

The current rules include:

Alert

Condition

Severity

HighCPUUsage

CPU usage above 80% for 2 minutes

Warning

HighMemoryUsage

Memory usage above 90% for 2 minutes

Critical

LowDiskSpace

Available disk space below 15% for 5 minutes

Warning

TargetDown

Prometheus target unreachable for 1 minute

Critical

Example alert flow:

Node Exporter DOWN
        │
        ▼
Prometheus detects up == 0
        │
        ▼
TargetDown → PENDING
        │
        ▼
TargetDown → FIRING
        │
        ▼
Alertmanager
        │
        ▼
Discord notification

Prometheus Alert



🔔 Discord Notifications

Alertmanager routes firing and resolved alerts to Discord through a webhook.

Notifications include information such as:

Alert status

Alert name

Severity

Instance

Job

Summary

Description

Example:



The real Discord webhook URL is not committed to the repository.

It is stored locally in:

discord_webhook_url

and excluded through .gitignore.

Docker Compose mounts it into Alertmanager as a secret:

/run/secrets/discord_webhook

📊 Grafana Provisioning

Grafana is automatically configured when the stack starts.

The repository contains:

grafana/
├── dashboards/
│   └── node-exporter.json
│
└── provisioning/
    ├── dashboards/
    │   └── dashboard.yml
    │
    └── datasources/
        └── datasource.yml

This automatically provisions:

The Prometheus data source

The Infrastructure Monitoring dashboard

No manual data-source creation or dashboard import is required after deployment.

💾 Persistent Storage

Named Docker volumes are configured for:

prometheus-data
grafana-data
alertmanager-data

These volumes preserve service data when containers are restarted or recreated.

⚙️ Continuous Integration

The project includes a GitHub Actions workflow:

.github/workflows/ci.yml

The workflow runs automatically on pushes and pull requests targeting main.

It validates:

Docker Compose configuration

Prometheus configuration

Prometheus alert rules

Alertmanager configuration

Grafana dashboard JSON

A non-sensitive dummy Discord webhook file is created temporarily inside CI so the configuration can be validated without exposing the real webhook.

CI Pipeline



🚀 Getting Started

Prerequisites

Install:

Docker

Docker Compose

Git

1. Clone the repository

git clone https://github.com/MeLi-afkl/monitoring-stack.git
cd monitoring-stack

2. Configure the Discord webhook

Create a file named:

discord_webhook_url

in the project root and paste your Discord webhook URL into that file.

Do not commit this file. It is excluded through .gitignore.

3. Start the stack

docker compose up -d

4. Verify the containers

docker compose ps

Expected services:

prometheus
grafana
node-exporter
alertmanager

🌐 Service Access

Service

Address

Grafana

http://localhost:3000

Prometheus

http://localhost:9090

Alertmanager

http://localhost:9093

Node Exporter

http://localhost:9100/metrics

Default Grafana credentials:

Username: admin
Password: admin

These default credentials are intended only for local development and should be changed for any real deployment.

🧪 Testing the Alert Pipeline

A simple way to test the complete monitoring flow is to stop Node Exporter:

docker stop node-exporter

Prometheus detects the target as unavailable. After the configured delay, the TargetDown rule enters the FIRING state and Alertmanager forwards the notification to Discord.

Restart Node Exporter with:

docker start node-exporter

The target should recover and the alert should resolve.

📁 Project Structure

monitoring-stack/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── grafana/
│   ├── dashboards/
│   │   └── node-exporter.json
│   │
│   └── provisioning/
│       ├── dashboards/
│       │   └── dashboard.yml
│       │
│       └── datasources/
│           └── datasource.yml
│
├── screenshots/
│   ├── grafana-dashboard.png
│   ├── prometheus-alert.png
│   ├── discord-alert.png
│   └── github-actions-ci.png
│
├── .gitignore
├── alertmanager.yml
├── alerts.yml
├── docker-compose.yml
├── prometheus.yml
└── README.md

The local discord_webhook_url file is intentionally excluded from version control.

🧠 Key Learnings

This project provided hands-on experience with:

Building a multi-container monitoring environment

Collecting metrics with Prometheus and Node Exporter

Writing PromQL queries

Creating Prometheus alerting rules

Configuring Alertmanager routing

Integrating Discord webhook notifications

Building a custom Grafana dashboard

Provisioning Grafana dashboards and data sources as code

Using persistent Docker volumes

Keeping secrets outside source control

Validating monitoring configuration through GitHub Actions

Troubleshooting Docker networking and monitoring configuration

🔮 Possible Future Improvements

Potential extensions:

Deploy the stack to a cloud VM

Add Loki for centralized log aggregation

Monitor additional hosts and services

Add application-specific metrics

Add HTTPS and stronger authentication

Pin container image versions instead of using latest

👤 Author

Andreea Meli Cazan

Network Engineer transitioning into DevOps, with hands-on experience in networking, infrastructure monitoring, containerization, and automation.