# Day 10 – Bash Website Backup and Remote Archive Automation

## Challenge

The production support team needed a Bash script to automate backups of a static website hosted on Application Server 3.

The requirements were to:

- Create `/scripts/beta_archive.sh`
- Archive `/var/www/html/beta`
- Create `/archives/xfusioncorp_beta.zip`
- Keep a local copy on Application Server 3
- Copy the archive to `/archives/` on the Nautilus Storage Server
- Perform the remote copy without prompting for a password
- Ensure the normal application-server user can execute the script
- Avoid using `sudo` inside the script
- Ensure the `zip` package is installed before running the script

## What I Learned

- How to write a practical Bash automation script
- How to create ZIP archives from the command line
- How Bash variables make scripts easier to maintain
- How to transfer files using `scp`
- How SSH keys enable unattended remote file transfers
- How to verify both local and remote backup copies
- Why privileged commands should not be embedded unnecessarily in automation
- How several Linux tools can be combined into one operational workflow

## Architecture

The backup workflow was:

```text
Application Server 3

/var/www/html/beta
        │
        │ zip
        ▼
/archives/xfusioncorp_beta.zip
        │
        │ scp over SSH key authentication
        ▼
Storage Server

/archives/xfusioncorp_beta.zip
```

## Environment Inspection

Before writing the script, I verified that the source website existed:

```bash
ls -ld /var/www/html/beta
```

I also inspected some of the files:

```bash
find /var/www/html/beta -maxdepth 2 -type f | head
```

The directory contained website content including:

```text
/var/www/html/beta/index.html
```

I checked the required directories:

```bash
ls -ld /scripts /archives
```

and verified that the `zip` package was installed:

```bash
rpm -q zip
```

## Passwordless Storage Server Access

Because the script needed to copy the archive automatically, it could not stop and request a password.

I generated an SSH key for the application-server user:

```bash
ssh-keygen -t ed25519
```

The key pair consisted of:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

The private key remains on the source server and must never be exposed.

I copied the public key to the storage-server account:

```bash
ssh-copy-id natasha@ststor01
```

## Verify Non-Interactive SSH

I tested the connection using:

```bash
ssh -o BatchMode=yes natasha@ststor01 hostname
```

The command returned:

```text
ststor01
```

without requesting a password.

This confirmed that automation could connect to the storage server non-interactively.

## Bash Script

I created:

```text
/scripts/beta_archive.sh
```

with the following script:

```bash
#!/bin/bash

SOURCE="/var/www/html/beta"
ARCHIVE="/archives/xfusioncorp_beta.zip"
REMOTE_USER="natasha"
REMOTE_HOST="ststor01"
REMOTE_DIR="/archives"

# Create/update the website archive
zip -r "$ARCHIVE" "$SOURCE"

# Copy the archive to the storage server
scp "$ARCHIVE" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
```

## Why Variables Were Used

Instead of repeating paths and connection information throughout the script, variables were defined at the beginning:

```bash
SOURCE="/var/www/html/beta"
ARCHIVE="/archives/xfusioncorp_beta.zip"
REMOTE_USER="natasha"
REMOTE_HOST="ststor01"
REMOTE_DIR="/archives"
```

This makes the script easier to:

- Read
- Modify
- Troubleshoot
- Reuse

## Make the Script Executable

I granted execute permission:

```bash
chmod +x /scripts/beta_archive.sh
```

I then verified the permissions:

```bash
ls -l /scripts/beta_archive.sh
```

The script was executable by the application-server user.

## Execute the Backup

I ran:

```bash
/scripts/beta_archive.sh
```

The script:

1. Created the ZIP archive.
2. Stored it in `/archives/`.
3. Transferred the archive to the storage server.
4. Completed the remote copy without requesting a password.

## Local Verification

I verified the local archive:

```bash
ls -lh /archives/xfusioncorp_beta.zip
```

I then inspected its contents:

```bash
unzip -l /archives/xfusioncorp_beta.zip
```

The archive contained the expected website files, including:

```text
var/www/html/beta/
var/www/html/beta/index.html
```

This proved that the archive itself was valid rather than merely confirming that a file with a `.zip` extension existed.

## Remote Verification

I verified the copy on the storage server without opening an interactive SSH session:

```bash
ssh -o BatchMode=yes natasha@ststor01 \
'ls -lh /archives/xfusioncorp_beta.zip'
```

The remote archive was returned successfully.

This confirmed the full backup workflow had completed.

## Verify No `sudo` Was Used in the Script

The requirement specifically prohibited using `sudo` inside the automation script.

I checked the script using:

```bash
grep -n sudo /scripts/beta_archive.sh
```

The command returned no output.

This confirmed that no `sudo` commands were embedded in the script.

## End-to-End Verification

The completed workflow was verified at several layers:

```text
Source website exists
        ↓
Script executable
        ↓
ZIP archive created
        ↓
Archive contains website files
        ↓
Passwordless SSH operational
        ↓
SCP succeeds
        ↓
Remote archive exists
```

## Connection to Previous Challenges

Several earlier tasks contributed directly to this automation:

```text
Day 4
Linux file permissions
        ↓
Executable scripts

Day 7
SSH key authentication
        ↓
Passwordless remote access

Day 10
Bash + ZIP + SCP
        ↓
Automated remote backup
```

This shows how individual Linux concepts can be combined into a practical infrastructure workflow.

## Real-World Relevance

Similar scripts can be used to automate:

- Website backups
- Configuration backups
- Database dumps
- Application artifacts
- Log archives
- Off-host backup replication
- Disaster-recovery copies

A real backup process should not only create an archive.

It should also verify that:

```text
Backup created
      +
Backup contains expected data
      +
Remote copy succeeded
```

## Security Considerations

Several security principles are relevant to this workflow.

### Protect SSH Private Keys

The private key:

```text
~/.ssh/id_ed25519
```

must never be committed to Git or shared publicly.

### Avoid Embedded Passwords

The script does not contain passwords.

SSH public-key authentication is used instead.

### Avoid Unnecessary Privilege Escalation

No `sudo` commands are included inside the script.

The account running the automation should receive only the permissions required to perform the backup.

### Directory Permissions

The training environment provided writable `/scripts` and `/archives` directories.

In a production environment, these directories should normally have more restrictive ownership and permissions rather than being writable by every system user.

## Production Improvements

For a production-grade version of this backup script, I would extend it with:

- `set -euo pipefail`
- Error handling
- Logging
- Timestamped archive names
- Backup retention
- Checksums
- Remote integrity verification
- Disk-space checks
- Failure notifications
- Restricted SSH keys
- Automated scheduling

These improvements will be incorporated later as this portfolio grows from individual exercises into larger infrastructure projects.

## Key Takeaways

1. Bash can combine multiple Linux tools into repeatable operational workflows.
2. `zip` can package application data for backup.
3. `scp` provides secure remote file transfer over SSH.
4. SSH keys allow unattended automation without storing plaintext passwords.
5. Automation should be verified at both the source and destination.
6. A backup should be inspected to confirm that it actually contains the expected files.
7. Scripts should avoid unnecessary elevated privileges.
8. Private SSH keys must never be committed to source control.
9. Individual Linux skills become more valuable when combined into end-to-end workflows.

## Commands Used

```bash
ssh banner@stapp03

ls -ld /var/www/html/beta

find /var/www/html/beta -maxdepth 2 -type f | head

ls -ld /scripts /archives

rpm -q zip

ssh-keygen -t ed25519

ssh-copy-id natasha@ststor01

ssh -o BatchMode=yes natasha@ststor01 hostname

chmod +x /scripts/beta_archive.sh

/scripts/beta_archive.sh

ls -lh /archives/xfusioncorp_beta.zip

unzip -l /archives/xfusioncorp_beta.zip

ssh -o BatchMode=yes natasha@ststor01 \
'ls -lh /archives/xfusioncorp_beta.zip'

grep -n sudo /scripts/beta_archive.sh
```

---

**Status:** ✅ Completed
