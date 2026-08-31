# 100 Days of DevOps

A hands-on DevOps learning journey focused on building practical infrastructure skills, solving real-world engineering problems, and documenting my progress toward DevOps and DevSecOps engineering.

Rather than only completing labs, I am using each challenge to understand the underlying concepts, verify my work, document troubleshooting, and gradually combine related tasks into larger portfolio projects.

## Goals

* Build strong Linux and systems administration foundations
* Develop practical DevOps troubleshooting skills
* Learn infrastructure automation and CI/CD
* Gain hands-on experience with containers and Kubernetes
* Apply cloud and Infrastructure as Code practices
* Introduce security throughout the software delivery lifecycle
* Build portfolio projects that demonstrate practical engineering ability

## Learning Approach

For each challenge, I follow this process:

1. Understand the requirement
2. Research unfamiliar concepts
3. Implement the solution
4. Verify that the configuration behaves correctly
5. Document what I learned
6. Extend related challenges into larger real-world projects where appropriate

The goal is not simply to complete 100 tasks, but to understand **why** each solution works and how the same concepts are used in production environments.

## Progress

**Completed: 4 / 100**

| Day                                                         | Topic                                       | Key Skills                                                                       | Status     |
| ----------------------------------------------------------- | ------------------------------------------- | -------------------------------------------------------------------------------- | ---------- |
| [Day 01](daily-notes/day-001-linux-non-interactive-user.md) | Linux User Setup with Non-Interactive Shell | Linux users, service accounts, `useradd`, `/etc/passwd`, `nologin`, verification | ✅ Complete |
| [Day 02](daily-notes/day-002-temporary-user-expiry.md) | Temporary Linux User with Expiry Date | `useradd`, account expiry, `chage`, access lifecycle | ✅ Complete |
| [Day 03](daily-notes/day-003-disable-root-ssh-login.md) | Disable Direct Root SSH Login | SSH hardening, `sshd_config`, `PermitRootLogin`, configuration validation | ✅ Complete |
| [Day 04](daily-notes/day-004-linux-file-permissions.md) | Linux File Permissions for Executable Script | `chmod`, symbolic permissions, numeric modes, least privilege | ✅ Complete |

## Areas Covered

As the challenge progresses, this repository will include practical work involving:

* Linux administration
* Shell scripting
* Git
* Networking
* Web and application servers
* Docker
* CI/CD
* Infrastructure as Code
* Terraform
* Kubernetes
* Cloud infrastructure
* Monitoring and observability
* Security and DevSecOps practices
* Troubleshooting and automation

## Portfolio Projects

Related daily challenges will be combined into larger projects instead of treating every small task as an isolated project.

Planned project areas include:

```text
Linux System Administration
        ↓
Containerisation
        ↓
CI/CD Automation
        ↓
Infrastructure as Code
        ↓
Kubernetes
        ↓
Cloud Infrastructure
        ↓
Observability
        ↓
DevSecOps
```

These projects will evolve throughout the challenge as new concepts are introduced.

## Security

No passwords, API keys, access tokens, private infrastructure credentials, or other sensitive information will be committed to this repository.

Any credentials used inside temporary training environments are excluded from the documentation.

## Repository Structure

```text
100-days-of-devops/
│
├── README.md
│
├── daily-notes/
│   └── day-001-linux-non-interactive-user.md
│
├── linux-administration/
├── docker/
├── cicd/
├── terraform/
├── kubernetes/
└── monitoring/
```

Project directories will be added only when enough related work exists to justify a standalone project.

---

## Current Focus

### Day 1 — Linux User Setup with Non-Interactive Shell

Created a dedicated Linux service account for a backup-agent use case and configured it with `/sbin/nologin` to prevent interactive shell access.

The configuration was independently verified through `/etc/passwd`, `getent`, and an attempted interactive login.

➡️ [View Day 1 notes](daily-notes/day-001-linux-non-interactive-user.md)

---

**Challenge:** KodeKloud 100 Days of DevOps
**Focus:** Practical DevOps → DevSecOps Engineering
