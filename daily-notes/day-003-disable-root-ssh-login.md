# Day 3 – Disable Direct Root SSH Login

## Challenge

Following a security audit, direct SSH login for the `root` account needed to be disabled across all application servers.

The task was to configure all three application servers so that users could no longer connect directly over SSH as `root`.

## What I Learned

- How SSH server configuration is managed through `sshd_config`
- The purpose of the `PermitRootLogin` directive
- Why direct root SSH login should be restricted
- The difference between `PermitRootLogin yes`, `prohibit-password`, and `no`
- How to validate SSH configuration before restarting the service
- How to restart and verify the SSH daemon
- How to inspect the effective SSH configuration with `sshd -T`
- Why configuration verification is important when making security changes

## Target Servers

The configuration was applied to all three application servers:

```text
Application Server 1
Application Server 2
Application Server 3
```

Each server was accessed through the jump host using its designated administration account.

## Initial Configuration

Before making any changes, I inspected the existing SSH configuration:

```bash
sudo grep -nEi '^[[:space:]]*#?[[:space:]]*PermitRootLogin' /etc/ssh/sshd_config
```

The servers contained settings similar to:

```text
#PermitRootLogin prohibit-password
PermitRootLogin yes
```

The commented `prohibit-password` setting was not active.

The active:

```text
PermitRootLogin yes
```

allowed direct root SSH login and therefore needed to be changed.

## Implementation

I edited the SSH daemon configuration:

```bash
sudo vi /etc/ssh/sshd_config
```

The active configuration was changed to:

```text
PermitRootLogin no
```

This disables direct SSH authentication as the `root` user.

## Configuration Validation

Before restarting SSH, I validated the configuration syntax:

```bash
sudo sshd -t
```

A successful validation returns no output.

This step is important because restarting SSH with an invalid configuration could cause the SSH service to fail and potentially lock administrators out of a remote server.

## Restart SSH

After confirming that the configuration syntax was valid:

```bash
sudo systemctl restart sshd
```

## Verification

I checked the effective SSH daemon configuration using:

```bash
sudo sshd -T | grep permitrootlogin
```

Each application server returned:

```text
permitrootlogin no
```

This confirmed that direct root SSH login was disabled successfully across all three application servers.

## Why `sshd -T` Is Useful

Checking `/etc/ssh/sshd_config` alone shows what is written in the configuration file.

However:

```bash
sudo sshd -T
```

shows the effective configuration that the SSH daemon will actually use.

This is useful because SSH configuration may contain defaults, included files, duplicate directives, or other settings that influence the final result.

## Real-World Relevance

The `root` account has unrestricted privileges on a Linux system.

Allowing direct remote authentication as `root` creates several security concerns:

- Attackers know that the username `root` exists.
- Successful compromise immediately gives full administrative privileges.
- Shared root access reduces accountability.
- Administrative activity is harder to attribute to individual users.
- Credential attacks can target a highly privileged account directly.

A safer administration model is:

```text
Administrator
      ↓
SSH using personal account
      ↓
sudo
      ↓
Elevated command only when required
```

Instead of:

```text
Internet
   ↓
SSH
   ↓
root
   ↓
Full server access
```

This improves accountability and supports the principle of least privilege.

## Security Context

The first three days now demonstrate different identity and access controls:

```text
Day 1
Service account
    ↓
/sbin/nologin
    ↓
No interactive access


Day 2
Temporary developer account
    ↓
Account expiration
    ↓
Time-limited access


Day 3
Administrative SSH access
    ↓
Disable direct root login
    ↓
Use named users + sudo
```

Together, these controls form the beginning of a Linux server access-hardening strategy.

## Key Takeaways

1. Direct root SSH access should generally be disabled.
2. `PermitRootLogin no` prevents direct root SSH authentication.
3. Configuration syntax should be tested with `sshd -t` before restarting SSH.
4. `sshd -T` is useful for checking the effective SSH configuration.
5. Named administrative accounts combined with `sudo` provide better accountability than shared root access.
6. Security configuration should be applied consistently across every server in an environment.
7. Always verify security controls after making configuration changes.

## Commands Used

```bash
# Connect to an application server
ssh <admin-user>@<server>

# Inspect root SSH configuration
sudo grep -nEi '^[[:space:]]*#?[[:space:]]*PermitRootLogin' /etc/ssh/sshd_config

# Edit SSH configuration
sudo vi /etc/ssh/sshd_config

# Validate SSH configuration
sudo sshd -t

# Restart SSH
sudo systemctl restart sshd

# Verify effective configuration
sudo sshd -T | grep permitrootlogin
```

Expected result:

```text
permitrootlogin no
```

---

**Status:** ✅ Completed
