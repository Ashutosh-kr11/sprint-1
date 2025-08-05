<img src="https://github.com/user-attachments/assets/2dd05944-d7a3-44a6-91a5-23db8bf8454c" alt="Image" width="100%" />


# Employee API – Detailed Setup & Deployment Documentation

---

## Author Information

| **Author**   | **Created on** | **Version** | **Last updated on** | **Level** | **Reviewer**  |
|--------------|----------------|-------------|---------------------|-----------|---------------|
|Ashutosh Kumar| 2025-08-04     | 1.0         | 2025-08-04          | Internal  |Siddharth Pawar / Sahil Gupta|

---

## Table of Contents

- [Introduction](#introduction)
- [Purpose](#purpose)
- [Pre-requisites](#pre-requisites)
- [System Requirements](#system-requirements)
- [Dependencies](#dependencies)
- [Important Ports](#important-ports)
- [Architecture](#architecture)
- [Dataflow Diagram](#dataflow-diagram)
- [Step-by-step Installation](#step-by-step-installation)
- [Monitoring](#monitoring)
- [Health Check](#health-check)
- [Logging](#logging)
- [Disaster Recovery](#disaster-recovery)
- [High Availability](#high-availability)
- [Troubleshooting](#troubleshooting)
- [FAQs](#faqs)
- [Contact Information](#contact-information)
- [References](#references)

---

## Introduction

The Employee API is a microservice designed to provide a scalable, secure, and unified RESTful interface for managing employee records in modern enterprise environments. Developed using GoLang, it serves as a backend for HR, payroll, and IT integrations, supporting CRUD operations, advanced search, and robust security for employee data. The API leverages Redis for high-performance caching and ScyllaDB for resilient, high-throughput NoSQL storage.

---

## Purpose

This documentation aims to provide a comprehensive, step-by-step guide to installing, configuring, and maintaining the Employee API. The Employee API addresses common enterprise challenges such as data consistency, secure access, automated workflows, and integration with other business systems, offering a standardized and efficient way to manage employee information across the organization.

---

## Pre-requisites

Before proceeding with deployment, ensure the following hardware, software, and security prerequisites are satisfied.

---

## System Requirements

| Hardware Specifications | Minimum Recommendation         |
|------------------------|-------------------------------|
| Processor              | Dual-core                     |
| RAM                    | 4GB (20GB+ for ScyllaDB node) |
| Disk                   | 20GB                          |
| OS                     | Ubuntu 22.04 LTS              |

---

## Dependencies

### Build Time

| Name    | Version   | Description                  |
|---------|-----------|------------------------------|
| GoLang  | v1.20+    | Application language runtime |
| Make    | Latest    | Automation of build tasks    |
| Git     | Latest    | Source code management       |

### Runtime

| Name        | Version   | Description                  |
|-------------|-----------|------------------------------|
| Redis       | Latest    | In-memory caching            |
| ScyllaDB    | v6.1+     | NoSQL database               |

### Other

| Name      | Version   | Description                    |
|-----------|-----------|--------------------------------|
| Prometheus| Latest    | Monitoring and metrics         |
| Swagger UI| Latest    | API documentation & testing    |
| jq        | Latest    | Command-line JSON processor    |

---

## Important Ports

| Port | Service      | Description                              |
|------|--------------|------------------------------------------|
| 22   | SSH          | Server access                            |
| 80   | HTTP         | Web traffic                              |
| 443  | HTTPS        | Secure web traffic                       |
| 8080 | Employee API | API & Swagger UI                         |
| 6379 | Redis        | Caching                                  |
| 9042 | ScyllaDB     | NoSQL database (CQL native transport)    |

---

## Architecture

The Employee API uses a microservices architecture, decoupling the API layer from the data storage. The service communicates with Redis for fast-access caching and ScyllaDB for distributed NoSQL data. Authentication is handled via JWT tokens, and observability is supported through Prometheus metrics endpoints.

---

## Dataflow Diagram

![Employee API Architecture and Dataflow](https://github.com/user-attachments/assets/6ea12e87-00d7-4576-adea-99594d2729cf)

**Logical Data Flow:**
1. The API first checks Redis cache for the requested data.
2. On a cache miss, it queries ScyllaDB.
3. Data fetched from ScyllaDB is cached in Redis for subsequent requests.
4. All migrations (schema/data changes) are applied directly to ScyllaDB.
5. Responses are served to the end-user or calling service.

---

## Step-by-step Installation

### Step 1: Install Build and Runtime Dependencies

**Update and install essentials**
```bash
sudo apt update
sudo apt install -y git make jq
```
**Install GoLang**
```bash
wget https://go.dev/dl/go1.20.6.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.6.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
go version
```
**Install Redis**
```bash
sudo apt install -y redis-server
sudo systemctl enable --now redis-server
```
**Install ScyllaDB**
Refer to [ScyllaDB official docs](https://www.scylladb.com/doc/latest/getting-started/install-linux/) for up-to-date installation steps.

### Step 2: Clone and Configure the Application

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api
cd employee-api/
```
- Configure cache and database connection settings in `config.yaml`
- Add initial seed data in `migration.json` (if needed)

### Step 3: Database Preparation

**ScyllaDB**
- Create required keyspace and tables using CQLSH, e.g.:
```cql
CREATE KEYSPACE employee_db WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
USE employee_db;
CREATE TABLE employee (id UUID PRIMARY KEY, name TEXT, designation TEXT, location TEXT);
```

### Step 4: Build and Run the API

```bash
make build
./employee-api
```

### Step 5: Verify Deployment

- Access Swagger UI: `http://<server-ip>:8080/swagger/index.html`
- Run health checks and test endpoints using Swagger or curl.

---

## Monitoring

Employee API exposes Prometheus-style metrics and uses application logs for observability.

### Key Metrics

| Parameter           | Description                                  | Priority | Threshold                    |
|---------------------|----------------------------------------------|----------|------------------------------|
| Disk Utilization    | Percentage of disk space used                | High     | >90%                         |
| Availability        | Application uptime percentage                | High     | ≥ 99.9%                      |
| Memory Utilization  | Memory used by the application               | Medium   | >80%                         |
| CPU Utilization     | CPU time used                                | Medium   | >70%                         |
| Network Traffic     | Network usage                                | Medium   | Varies per use case          |
| Latency             | Response time per request                    | High     | <300ms                       |
| Errors              | Number of errors per minute                  | High     | >5/min                       |
| Throughput          | Requests handled per minute                  | High     | >1000/min                    |
| Security            | Auth, encryption, vulnerability monitoring   | High     | Continuous monitoring        |

---

## Health Check

| Name        | Type            | InitialDelaySeconds | PeriodSeconds | TimeoutSeconds | SuccessThreshold | FailureThreshold |
|-------------|-----------------|--------------------|---------------|---------------|------------------|------------------|
| EmployeeAPI | ReadinessProbe  | 10                 | 10            | 5             | 1                | 3                |
| EmployeeAPI | LivenessProbe   | 10                 | 10            | -             | 5                | 1                |

- **ReadinessProbe:** Checks if the service is ready to receive traffic.
- **LivenessProbe:** Checks if the service is running/responding.
- **InitialDelaySeconds:** Wait before first check.
- **PeriodSeconds:** Frequency of checks.
- **TimeoutSeconds:** Max response time before considered unhealthy.
- **Success/FailureThreshold:** Number of successes/failures before changing health status.

---

## Logging

| Log Type                       | Path/Location       | Description                                              |
|--------------------------------|---------------------|----------------------------------------------------------|
| Event Logs                     | logs/event.log      | Tracks API events, login attempts, and usage data        |
| Auth Logs                      | logs/access.log     | Tracks authentication, authorization, user actions       |
| Server Logs                    | logs/server.log     | Tracks server-level actions and errors                   |
| Threat Logs                    | logs/threat.log     | Records security events, suspicious traffic, etc.        |

---

## Disaster Recovery

- **Backups:** Automated, scheduled backups for ScyllaDB.
- **Restore Plan:** Documented DB and config restoration process.
- **Failover:** Secondary standby instances for critical services.
- **Periodic Testing:** Disaster recovery drills to ensure readiness.

---

## High Availability

- **Load Balancer:** Distributes traffic across multiple API instances.
- **Redundant Infrastructure:** Multiple DB and cache nodes.
- **Automated Health Checks:** Services auto-restart on failure.
- **Replication:** Data replicated across zones/nodes for resilience.

---

## Troubleshooting

| Issue                        | Description / Cause                    | Solution / Fix                                  |
|------------------------------|----------------------------------------|-------------------------------------------------|
| Port conflict on 8080        | Another process using port             | Use `lsof -i :8080` and stop conflicting process|
| DB connection failure        | ScyllaDB not running, wrong credentials| Verify DB service and credentials in config     |
| Cache connection failure     | Redis not running or misconfigured     | Check Redis service status and config           |
| Build errors                 | Dependency/version mismatch            | Check Go version, run `make clean build`        |
| API returns 401 Unauthorized | Invalid/missing JWT token              | Ensure valid JWT in request headers             |

---

## Contact Information

| Name   | Email                      |
|--------|----------------------------|
| Ashutosh Kumar  | [ashutosh.kumar.snaatak@mygurukulam.co](mailto:ashutosh.kumar.snaatak@mygurukulam.co) |

---

## References

| Link                                                                 | Description                                   |
|----------------------------------------------------------------------|-----------------------------------------------|
| https://github.com/OT-MICROSERVICES/employee-api                     | Employee API Source & Releases                |
| https://www.scylladb.com/doc/latest/getting-started/install-linux/   | ScyllaDB Installation Guide                   |
| https://redis.io/docs/getting-started/installation/                  | Redis Official Installation                   |

---
