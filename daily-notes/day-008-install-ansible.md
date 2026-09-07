# Day 8 – Install Ansible on the Control Node

## Challenge

The DevOps team selected Ansible as its initial configuration-management and automation tool.

The jump host would act as the Ansible control node and needed:

- Ansible version `4.8.0`
- Installation using `pip3`
- A globally available Ansible executable
- Access for all users on the system to run Ansible commands

## What I Learned

- What an Ansible control node is
- How Ansible fits into configuration management
- How to install Python packages using `pip3`
- The difference between the Ansible community package and `ansible-core`
- Why installation location matters for system-wide tools
- How to verify executable paths using `command -v`
- How Ansible builds on SSH-based remote administration

## Architecture

The jump host now acts as the central automation node:

```text
                   Jump Host
                     thor
                      │
                   Ansible
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       stapp01      stapp02      stapp03
        tony         steve        banner
```

Day 7 provided passwordless SSH connectivity.

Day 8 installs the automation engine that can use those connections.

## Installation

The required version was installed using `pip3`:

```bash
sudo pip3 install ansible==4.8.0
```

### Command Breakdown

```text
sudo
└── Install the package system-wide

pip3
└── Python 3 package manager

install
└── Install a Python package

ansible==4.8.0
└── Install exactly Ansible version 4.8.0
```

The exact version was specified to ensure a reproducible installation.

## Verify the Package Version

I verified the installed Ansible package with:

```bash
pip3 show ansible
```

The result included:

```text
Name: ansible
Version: 4.8.0
```

This confirmed that the required package version was installed successfully.

## Verify the Ansible CLI

I then checked:

```bash
ansible --version
```

The output reported:

```text
ansible [core 2.11.12]
```

This is expected.

Ansible `4.8.0` is the community package distribution, while the command-line tool reports the version of the underlying `ansible-core` package.

Conceptually:

```text
Ansible Community Package
        4.8.0
          │
          └── ansible-core 2.11.12
```

Therefore both outputs describe different components of the same installation.

## Verify Global Availability

I checked where the executable was installed:

```bash
command -v ansible
```

The result was:

```text
/usr/local/bin/ansible
```

This confirms that Ansible was installed in a globally accessible executable path rather than only inside the `thor` user's home directory.

I also inspected the executable:

```bash
ls -l "$(command -v ansible)"
```

This confirmed that the Ansible command was available system-wide.

## Why Ansible?

Ansible is a configuration-management and automation platform that allows administrators to describe desired system configuration in code.

Instead of manually logging into multiple servers and running the same commands:

```text
Admin
 ├── SSH → Server 1 → configure
 ├── SSH → Server 2 → configure
 └── SSH → Server 3 → configure
```

Ansible allows the process to become:

```text
Ansible Playbook
       │
       ├──► Server 1
       ├──► Server 2
       └──► Server 3
```

This improves:

- Consistency
- Repeatability
- Scalability
- Auditability
- Automation

## Connection to Day 7

Day 7 established:

```text
thor@jump-host
     │
     ├── passwordless SSH → stapp01
     ├── passwordless SSH → stapp02
     └── passwordless SSH → stapp03
```

Day 8 adds:

```text
thor@jump-host
     │
   Ansible
     │
     ├──► stapp01
     ├──► stapp02
     └──► stapp03
```

The previous SSH configuration therefore becomes the transport layer for future Ansible automation.

## Real-World Relevance

Configuration-management tools such as Ansible are commonly used for:

- Installing software across fleets of servers
- Managing users and groups
- Configuring SSH
- Deploying applications
- Managing services
- Applying security hardening
- Creating configuration files
- Performing repeatable server setup
- Enforcing configuration consistency

This reduces configuration drift caused by manually administering servers one at a time.

## Security Considerations

The installation produced a warning about running `pip` as root.

In normal production environments, Python environments and operating-system package management should be considered carefully to avoid package conflicts.

For this exercise, however, the requirement specifically called for a globally available Ansible installation using `pip3`, so the system-wide installation fulfilled the scenario requirements.

Other important considerations include:

- Protect SSH private keys used by Ansible
- Avoid storing plaintext credentials in playbooks
- Use Ansible Vault or external secret-management systems for secrets
- Restrict access to the control node
- Use least-privilege remote accounts

## Progress So Far

```text
Day 1 – Service-account restrictions
Day 2 – Temporary-account expiration
Day 3 – SSH hardening
Day 4 – File permissions
Day 5 – SELinux configuration
Day 6 – Cron automation
Day 7 – Passwordless SSH
Day 8 – Ansible control node
```

The progression is now moving from manual administration toward centralized automation.

```text
Manual Linux Administration
           ↓
SSH Automation
           ↓
Ansible
           ↓
Configuration as Code
```

## Key Takeaways

1. Ansible can act as a central configuration-management platform.
2. `pip3` can install exact package versions using `package==version`.
3. Ansible `4.8.0` and `ansible-core 2.11.12` refer to different layers of the installation.
4. `command -v` verifies where an executable is available.
5. Installing tools globally allows multiple users to access them.
6. Passwordless SSH provides the connectivity Ansible can use for remote automation.
7. Configuration management is more scalable and repeatable than administering servers manually.

## Commands Used

```bash
python3 --version

pip3 --version

sudo pip3 install ansible==4.8.0

pip3 show ansible

ansible --version

command -v ansible

ls -l "$(command -v ansible)"
```

---

**Status:** ✅ Completed
