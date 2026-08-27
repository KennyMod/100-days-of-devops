# Day 1 – Linux User Setup with Non-Interactive Shell

## Challenge

A backup agent required a dedicated Linux user account on an application server, but the account should not allow interactive shell access.

The task was to create a user named `ravi` on **Application Server 1** and configure it with a non-interactive shell.

## What I Learned

* Creating Linux users with `useradd`
* Understanding login shells
* The purpose of `/sbin/nologin`
* How Linux stores user information in `/etc/passwd`
* Using `getent` to query user account information
* Using `sudo` for privileged system administration
* Verifying configuration instead of assuming a command succeeded
* Why service accounts should follow the principle of least privilege

## Implementation

I first connected from the jump host to the required application server using SSH:

```bash
ssh tony@stapp01
```

I then created the `ravi` account and assigned `/sbin/nologin` as its shell:

```bash
sudo useradd -s /sbin/nologin ravi
```

### Command Breakdown

```text
sudo
└── Run the command with elevated privileges

useradd
└── Create a new Linux user

-s
└── Specify the user's login shell

/sbin/nologin
└── Prevent interactive shell access

ravi
└── Name of the user being created
```

## Verification

Creating the account was only the first step. I verified that the configuration was actually correct.

### Check the user account

```bash
getent passwd ravi
```

The result showed:

```text
ravi:x:1001:1001::/home/ravi:/sbin/nologin
```

The final field confirms that the account uses:

```text
/sbin/nologin
```

### Verify through `/etc/passwd`

```bash
grep '^ravi:' /etc/passwd
```

This returned the same account configuration.

### Test Interactive Login

Finally, I attempted to start an interactive session as the user:

```bash
sudo -iu ravi
```

The system responded:

```text
This account is currently not available.
```

This confirmed that interactive shell access was successfully disabled.

## Real-World Relevance

Linux service accounts are commonly created for:

* Backup agents
* Monitoring services
* Web servers
* Database services
* CI/CD runners
* Automation tools
* Background applications

These accounts often require an operating-system identity to run processes but do not require a human to log in interactively.

Allowing unnecessary interactive access increases the attack surface of a server.

Assigning a shell such as `/sbin/nologin` helps enforce the **principle of least privilege** by giving the account only the access it requires.

## Key Takeaways

1. A successful command does not necessarily mean the configuration is correct — always verify.
2. `/etc/passwd` contains important information about Linux user accounts, including their configured shell.
3. Service accounts should generally not have interactive login access unless there is a genuine operational requirement.
4. Security can begin with simple system-administration decisions such as restricting unnecessary account capabilities.

## Commands Used

```bash
ssh tony@stapp01

sudo useradd -s /sbin/nologin ravi

getent passwd ravi

grep '^ravi:' /etc/passwd

sudo -iu ravi
```

---

**Status:** ✅ Completed
