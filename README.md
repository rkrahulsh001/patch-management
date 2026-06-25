# Patch Management POC

## Jenkins + Ansible Linux Package Patching Automation

### Overview

This Proof of Concept (POC) demonstrates an automated Linux package patching solution using Jenkins and Ansible.

The objective of this project is to simulate an enterprise patch management workflow where package upgrades and downgrades can be executed in a controlled manner across Development, Staging, and Production environments.

The solution includes:

* Parameterized Jenkins Pipeline
* Ansible-based package management
* Dry Run capability
* Environment-wise deployment
* Production Approval Gate
* Patch Reporting
* Version Verification

---

## Use Case

In enterprise environments, infrastructure teams regularly patch packages such as:

* Nginx
* Apache
* OpenSSH
* MySQL

Manual patching is time-consuming and error-prone.

This POC demonstrates how patching can be automated through Jenkins and Ansible while maintaining deployment control and auditability.

---

## Features

* Parameterized Jenkins Pipeline
* Ansible Role-Based Architecture
* SSH Key-Based Authentication
* Dev → Staging → Production Rollout
* Dry Run Validation
* Production Approval Workflow
* Package Version Tracking
* Automated Patch Reports
* Upgrade and Downgrade Support
* Multiple Package Selection

---

## Architecture

```text
                Jenkins

                    │

                    ▼

          Ansible Control Node

                    │

        ┌───────────┼───────────┐

        ▼           ▼           ▼

   Dev Server   Staging Server   Prod Server
   172.17.0.3   172.17.0.4      172.17.0.5
```

---

## Pipeline Parameters

| Parameter       | Description            |
| --------------- | ---------------------- |
| TARGET_ENV      | dev / staging / prod   |
| ACTION          | upgrade / downgrade    |
| PATCH_NGINX     | Patch Nginx            |
| PATCH_APACHE    | Patch Apache           |
| PATCH_OPENSSH   | Patch OpenSSH          |
| PATCH_MYSQL     | Patch MySQL            |
| NGINX_VERSION   | Target Nginx Version   |
| APACHE_VERSION  | Target Apache Version  |
| OPENSSH_VERSION | Target OpenSSH Version |
| MYSQL_VERSION   | Target MySQL Version   |
| DRY_RUN         | Simulate execution     |

---

## Project Structure

```text
patch-management/

├── Jenkinsfile

├── inventory/
│   └── hosts.ini

├── playbooks/
│   ├── patch.yml
│   └── rollback.yml

├── roles/
│   ├── nginx/
│   ├── apache/
│   ├── openssh/
│   └── mysql/

├── reports/

└── README.md
```

---

## Pipeline Workflow

### Stage 1 – Pre-Flight Validation

Validate all user-provided parameters before execution.

---

### Stage 2 – Connectivity Verification

Verify Ansible connectivity to target servers.

```bash
ansible -i inventory/hosts.ini dev -m ping
```

---

### Stage 3 – Dry Run Execution

Simulate package changes without modifying the system.

```bash
ansible-playbook playbooks/patch.yml --check --diff
```

---

### Stage 4 – Production Approval

Manual approval is required before Production deployment.

```text
Approve Production Patching?
```

---

### Stage 5 – Package Patching

Execute package upgrade or downgrade.

```bash
ansible-playbook playbooks/patch.yml \
-e "package=nginx version=1.26.0-1~jammy"
```

---

### Stage 6 – Version Verification

Verify installed package version after patching.

```bash
dpkg -l | grep nginx
```

---

### Stage 7 – Patch Report Collection

Collect generated reports from target servers.

```bash
ansible -m fetch \
-a "src=/tmp/patch_report.txt dest=reports/"
```

---

## Inventory Example

```ini
[dev]
172.17.0.3

[staging]
172.17.0.4

[prod]
172.17.0.5

[all:vars]
ansible_user=ansible
ansible_ssh_private_key_file=/var/lib/jenkins/.ssh/ansible_key
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

## Security Considerations

* SSH Key-Based Authentication
* No Passwords Stored in Jenkins Pipeline
* Controlled Production Approval
* Least Privilege Access Model
* Passwordless Sudo Used Only for POC

---

## Prerequisites

* Jenkins
* Ansible
* Docker
* Ubuntu 22.04
* SSH Key Configuration

---

## Sample Execution

### Development Environment

```text
TARGET_ENV    = dev
PATCH_NGINX   = true
ACTION        = upgrade
DRY_RUN       = false
```

### Production Environment

```text
TARGET_ENV    = prod
PATCH_NGINX   = true
ACTION        = upgrade
DRY_RUN       = false

Manual Approval Required
```

---

## POC Environment

| Component       | Details           |
| --------------- | ----------------- |
| Jenkins         | localhost:8080    |
| Target Servers  | Docker Containers |
| OS              | Ubuntu 22.04      |
| Automation Tool | Ansible           |
| SCM             | Git               |

---

## Technologies Used

* Jenkins
* Ansible
* Groovy
* Ubuntu 22.04
* OpenSSH
* APT Package Manager
* Docker

---

## Learning Outcome

This project demonstrates:

* Jenkins Pipeline Development
* Infrastructure Automation
* Linux Patch Management
* Ansible Role Design
* Controlled Production Deployments
* CI/CD Integration for Operations Workflows

---

## Author

Rahul Sharma

