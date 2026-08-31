# Day 4 – Linux File Permissions for an Executable Script

## Challenge

A backup automation script had been distributed to the application servers, but the script on Application Server 1 did not have the required permissions.

The task was to make:

```text
/tmp/xfusioncorp.sh
```

executable by all users.

## What I Learned

- How Linux file permissions work
- The meaning of `r`, `w`, and `x`
- The difference between owner, group, and others
- How to inspect permissions using `ls -l`
- How to modify permissions with `chmod`
- How symbolic and numeric permissions relate
- Why granting only necessary permissions is safer than using `777`
- How to verify permissions from the perspective of an unprivileged user

## Initial State

I first inspected the script:

```bash
ls -l /tmp/xfusioncorp.sh
```

The file initially had:

```text
---------- 1 root root ... /tmp/xfusioncorp.sh
```

This meant that the file had no read, write, or execute permissions for:

```text
Owner
Group
Others
```

## Linux Permission Model

Linux permissions are grouped into three categories:

```text
Owner    Group    Others
rwx      rwx      rwx
```

Where:

```text
r = read
w = write
x = execute
```

For this task, every user needed to be able to execute the Bash script.

Since shell scripts also need to be readable by the interpreter, I granted both read and execute permissions.

## Implementation

I added read and execute permissions for all users:

```bash
sudo chmod a+rx /tmp/xfusioncorp.sh
```

### Command Breakdown

```text
chmod
└── Change file permissions

a
└── All users: owner, group, and others

+
└── Add permissions

r
└── Read permission

x
└── Execute permission
```

This granted the permissions required to use the script without granting unnecessary write access.

## Verification

I checked the new permissions:

```bash
ls -l /tmp/xfusioncorp.sh
```

The result was:

```text
-r-xr-xr-x 1 root root ... /tmp/xfusioncorp.sh
```

This shows:

```text
Owner   Group   Others
r-x     r-x     r-x
```

I also checked the numeric permission mode:

```bash
stat -c '%A %a %n' /tmp/xfusioncorp.sh
```

The result was:

```text
-r-xr-xr-x 555 /tmp/xfusioncorp.sh
```

## Verification as an Unprivileged User

Instead of simply assuming that other users could access the script, I verified the permissions using an unprivileged account.

Read test:

```bash
sudo -u nobody test -r /tmp/xfusioncorp.sh && echo "Readable"
```

Result:

```text
Readable
```

Execute test:

```bash
sudo -u nobody test -x /tmp/xfusioncorp.sh && echo "Executable"
```

Result:

```text
Executable
```

This confirmed that users outside the file owner and group could both read and execute the script.

## Why I Did Not Use `chmod 777`

A common shortcut would be:

```bash
chmod 777 file
```

However, `777` means:

```text
Owner    rwx
Group    rwx
Others   rwx
```

That would allow every user on the system to modify the backup script.

For an automation or administrative script, this could create a serious security risk because another user could change the script and potentially cause malicious commands to execute.

Instead, the final permissions were:

```text
555
```

which means:

```text
Owner    r-x
Group    r-x
Others   r-x
```

Users can read and execute the script, but they cannot modify it through the normal permission bits.

## Real-World Relevance

File permissions are important in production environments because scripts frequently perform privileged or automated operations such as:

- Backups
- Deployments
- Monitoring
- Log rotation
- Maintenance
- Scheduled jobs
- CI/CD tasks

Giving excessive permissions to automation scripts can introduce security vulnerabilities.

The safer approach is to grant only the permissions required for the script to perform its intended function.

This follows the principle of least privilege.

## Progress So Far

The first four challenges are now building a Linux security and administration foundation:

```text
Day 1
Service account
    ↓
Non-interactive shell

Day 2
Temporary account
    ↓
Automatic expiration

Day 3
SSH hardening
    ↓
Disable direct root login

Day 4
File permissions
    ↓
Controlled script execution
```

## Key Takeaways

1. Linux permissions are divided between owner, group, and others.
2. `chmod` can modify permissions using symbolic or numeric notation.
3. `a+rx` grants read and execute access to all users.
4. `555` represents `r-xr-xr-x`.
5. Avoid using `777` unless there is a very specific reason.
6. Security is often about granting only the permissions required.
7. Permissions should be verified after making changes.
8. Testing from the perspective of an unprivileged user provides stronger verification.

## Commands Used

```bash
ssh tony@stapp01

ls -l /tmp/xfusioncorp.sh

sudo chmod a+rx /tmp/xfusioncorp.sh

ls -l /tmp/xfusioncorp.sh

stat -c '%A %a %n' /tmp/xfusioncorp.sh

sudo -u nobody test -r /tmp/xfusioncorp.sh && echo "Readable"

sudo -u nobody test -x /tmp/xfusioncorp.sh && echo "Executable"
```

---

**Status:** ✅ Completed
