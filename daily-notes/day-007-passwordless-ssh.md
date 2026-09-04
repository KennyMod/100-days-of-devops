# Day 7 – Passwordless SSH Authentication

## Challenge

The system administration team needed scripts running from the jump host to perform operations across all application servers.

To support this automation, the `thor` user on the jump host needed passwordless SSH access to all application servers through their respective sudo-capable users.

## Target Architecture

```text
thor@jump-host
     │
     ├──► tony@stapp01
     ├──► steve@stapp02
     └──► banner@stapp03
```

## What I Learned

- How SSH public-key authentication works
- The difference between private and public SSH keys
- How to generate SSH keys with `ssh-keygen`
- How to distribute public keys with `ssh-copy-id`
- How `authorized_keys` is used for authentication
- How to configure passwordless remote access
- How to verify SSH connectivity in non-interactive mode
- Why SSH key authentication is important for infrastructure automation

## SSH Key Authentication

Traditional SSH authentication can use:

```text
Username
   +
Password
```

For automation, requiring a password every time is impractical.

SSH keys instead use a public/private key pair:

```text
Jump Host

Private Key
    │
    │ proves identity
    ▼
Remote Server
    │
Public Key stored in
~/.ssh/authorized_keys
```

The private key remains on the originating machine.

Only the public key is copied to remote systems.

## Key Generation

The SSH key was generated for the `thor` account using:

```bash
ssh-keygen -t ed25519
```

This created:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Where:

```text
id_ed25519
└── Private key

id_ed25519.pub
└── Public key
```

The private key must never be shared or committed to source control.

## Distribute the Public Key

The public key was copied to Application Server 1:

```bash
ssh-copy-id tony@stapp01
```

Application Server 2:

```bash
ssh-copy-id steve@stapp02
```

Application Server 3:

```bash
ssh-copy-id banner@stapp03
```

`ssh-copy-id` installs the public key in the remote user's:

```text
~/.ssh/authorized_keys
```

After the key was installed, SSH authentication no longer required the user's password.

## Verification

I first verified normal SSH access:

```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

Each connection succeeded without requiring a password.

## Non-Interactive Verification

Because the purpose of this configuration was automation, I also tested SSH using:

```bash
ssh -o BatchMode=yes tony@stapp01 hostname
```

Result:

```text
stapp01
```

For Application Server 2:

```bash
ssh -o BatchMode=yes steve@stapp02 hostname
```

Result:

```text
stapp02
```

For Application Server 3:

```bash
ssh -o BatchMode=yes banner@stapp03 hostname
```

Result:

```text
stapp03
```

`BatchMode=yes` prevents SSH from prompting for passwords or other interactive authentication.

This confirmed that scripts could connect to all three servers without human interaction.

## Why This Matters for DevOps

Passwordless SSH is commonly used as a foundation for remote automation.

Examples include:

- Configuration management
- Deployment scripts
- Server maintenance
- Remote command execution
- Ansible
- CI/CD pipelines
- Infrastructure orchestration
- Automated backups

The workflow becomes:

```text
Automation script
       │
       ▼
Jump Host
       │
SSH key authentication
       │
       ├──► Server 1
       ├──► Server 2
       └──► Server 3
```

## Security Considerations

SSH keys provide strong authentication, but they must be handled carefully.

Important practices include:

- Never commit private SSH keys to Git
- Restrict private-key permissions
- Use dedicated keys for automation where appropriate
- Rotate compromised or obsolete keys
- Remove unused public keys from `authorized_keys`
- Use least-privilege accounts rather than remote root access
- Protect jump hosts because they may provide access to multiple servers

## Connection to Previous Challenges

The first seven challenges are now forming a broader Linux administration and security workflow:

```text
Day 1
Service account restrictions
        ↓
Day 2
Temporary account expiration
        ↓
Day 3
Disable direct root SSH
        ↓
Day 4
Secure file permissions
        ↓
Day 5
SELinux configuration
        ↓
Day 6
Scheduled automation with cron
        ↓
Day 7
SSH key-based remote automation
```

Day 7 is particularly important because it creates a foundation for future configuration-management and deployment automation.

## Key Takeaways

1. SSH key authentication enables secure non-interactive remote access.
2. The private key must remain secret.
3. The public key can be installed on authorized remote systems.
4. `ssh-copy-id` simplifies public-key deployment.
5. `authorized_keys` controls which SSH keys can authenticate.
6. `BatchMode=yes` is useful for verifying automation-ready SSH connectivity.
7. Passwordless SSH is a common building block for DevOps automation.
8. Remote automation should use named least-privilege accounts rather than direct root access.

## Commands Used

```bash
ssh-keygen -t ed25519

ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03

ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03

ssh -o BatchMode=yes tony@stapp01 hostname
ssh -o BatchMode=yes steve@stapp02 hostname
ssh -o BatchMode=yes banner@stapp03 hostname
```

---

**Status:** ✅ Completed
