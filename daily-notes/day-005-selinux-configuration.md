# Day 5 – SELinux Installation and Persistent Configuration

## Challenge

Following a security audit, SELinux needed to be installed and configured on Application Server 2.

The requirements were to:

- Install the required SELinux packages
- Configure SELinux to be disabled persistently
- Avoid rebooting the server immediately
- Ensure the system would remain disabled after the next scheduled reboot

## What I Learned

- What SELinux is and why it exists
- The difference between Linux file permissions and Mandatory Access Control
- How SELinux packages are installed on CentOS/RHEL systems
- The difference between runtime SELinux state and persistent configuration
- How `/etc/selinux/config` controls SELinux behaviour after reboot
- How to verify installed packages using `rpm`
- Why configuration verification matters before considering a task complete

## Environment

The task was completed on:

```text
Application Server 2
Operating System: CentOS Stream 9
```

## Initial Inspection

I first checked the operating system:

```bash
cat /etc/os-release
```

The server was running:

```text
CentOS Stream 9
```

I then checked the current SELinux runtime state:

```bash
getenforce
```

The result was:

```text
Disabled
```

However, the challenge specifically required the persistent configuration to be correct after the next reboot.

The runtime state therefore was not enough to prove that the task was complete.

## Check Existing SELinux Packages

I checked whether the required SELinux components were already installed:

```bash
rpm -qa | grep -E '^selinux-policy|^policycoreutils'
```

Initially, only some supporting components were available.

The SELinux policy packages needed to be installed.

## Install SELinux Packages

I installed the required policy packages:

```bash
sudo dnf install -y selinux-policy selinux-policy-targeted
```

After installation, I verified the packages:

```bash
rpm -q selinux-policy selinux-policy-targeted policycoreutils
```

The system returned installed versions for:

```text
selinux-policy
selinux-policy-targeted
policycoreutils
```

This confirmed that the required SELinux components were installed.

## Persistent SELinux Configuration

After the SELinux packages were installed, the persistent configuration file became available:

```text
/etc/selinux/config
```

I inspected the configuration:

```bash
cat /etc/selinux/config
```

The SELinux mode was configured as:

```text
SELINUX=enforcing
```

The requirement was for SELinux to remain disabled after the next scheduled reboot.

I edited the file:

```bash
sudo vi /etc/selinux/config
```

and changed:

```text
SELINUX=enforcing
```

to:

```text
SELINUX=disabled
```

The final relevant configuration was:

```text
SELINUX=disabled
SELINUXTYPE=targeted
```

## Verification

I verified the persistent setting with:

```bash
grep '^SELINUX=' /etc/selinux/config
```

The result was:

```text
SELINUX=disabled
```

I also confirmed that the required packages were installed:

```bash
rpm -q selinux-policy selinux-policy-targeted policycoreutils
```

This ensured that both the package installation and persistent configuration requirements had been satisfied.

## Runtime State vs Persistent Configuration

One important lesson from this task was the difference between:

```text
Runtime state
    ↓
getenforce
```

and:

```text
Persistent boot configuration
    ↓
/etc/selinux/config
```

A command such as:

```bash
getenforce
```

shows the current state of SELinux.

However:

```text
/etc/selinux/config
```

determines how SELinux should behave after the system boots.

For this task, the persistent configuration was the important requirement because the server was scheduled to reboot later.

## What Is SELinux?

SELinux stands for:

```text
Security-Enhanced Linux
```

Traditional Linux permissions control access using:

```text
Owner
Group
Others
```

with:

```text
r = read
w = write
x = execute
```

SELinux provides an additional security layer using Mandatory Access Control.

Conceptually:

```text
Application
    ↓
Normal Linux permissions
    ↓
SELinux policy
    ↓
Resource access allowed or denied
```

This means that even when traditional Linux permissions allow an action, SELinux policy can still prevent it.

## Real-World Relevance

SELinux is commonly used to restrict what applications and services can access on Linux systems.

It can reduce the impact of a compromised service.

For example, even if a web server process becomes compromised, SELinux policies may prevent that process from accessing unrelated sensitive system files.

Although this exercise required temporarily disabling SELinux, production environments generally benefit from properly configuring and enforcing SELinux rather than permanently disabling it.

The exercise helped demonstrate how SELinux installation, state, and persistent configuration are managed.

## Security Perspective

The previous challenges focused mostly on identity and traditional Linux permissions:

```text
Day 1
Service-account restrictions

Day 2
Temporary-account expiration

Day 3
SSH root-login hardening

Day 4
File permission management
```

Day 5 introduced another security layer:

```text
Linux permissions
        ↓
Mandatory Access Control
        ↓
SELinux
```

This expands the Linux security model beyond basic users, groups, and file permissions.

## Key Takeaways

1. SELinux provides Mandatory Access Control on Linux systems.
2. Runtime SELinux state and persistent SELinux configuration are different.
3. `getenforce` displays the current runtime state.
4. `/etc/selinux/config` controls persistent SELinux behaviour.
5. `rpm -q` can verify whether required packages are installed.
6. Configuration changes should always be independently verified.
7. Reboots should not be performed unnecessarily when maintenance is already scheduled.
8. SELinux is an important Linux hardening technology even when a temporary task requires disabling it.

## Commands Used

```bash
ssh steve@stapp02

cat /etc/os-release

getenforce

rpm -qa | grep -E '^selinux-policy|^policycoreutils'

sudo dnf install -y selinux-policy selinux-policy-targeted

rpm -q selinux-policy selinux-policy-targeted policycoreutils

cat /etc/selinux/config

sudo vi /etc/selinux/config

grep '^SELINUX=' /etc/selinux/config
```

---

**Status:** ✅ Completed
