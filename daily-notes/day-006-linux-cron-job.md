# Day 6 – Linux Cron Job Automation

## Challenge

The system administration team needed to test scheduled automation across all application servers.

The requirements were to:

- Install the `cronie` package on all application servers
- Start the `crond` service
- Configure a cron job for the `root` user
- Run the job every 5 minutes
- Write the text `hello` to `/tmp/cron_text`

## What I Learned

- How Linux cron scheduling works
- How to install and manage the `crond` service
- How to configure root's crontab
- How cron expressions are structured
- How to verify a scheduled task actually executed
- Why service verification matters in automation
- How scheduled jobs are used for recurring infrastructure tasks

## Target Servers

The configuration was applied to all three application servers:

```text
Application Server 1
Application Server 2
Application Server 3
```

## Install Cronie

I installed the cron package:

```bash
sudo dnf install -y cronie
```

I verified the package was installed using:

```bash
rpm -q cronie
```

## Start and Enable the Cron Service

I started the service and enabled it to start automatically after reboot:

```bash
sudo systemctl enable --now crond
```

I then verified that the service was running:

```bash
sudo systemctl is-active crond
```

Expected result:

```text
active
```

## Cron Schedule

The required cron job was:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

A cron schedule contains five time fields:

```text
┌──────── minute
│ ┌────── hour
│ │ ┌──── day of month
│ │ │ ┌── month
│ │ │ │ ┌ day of week
│ │ │ │ │
* * * * *
```

For this task:

```text
*/5 * * * *
```

means:

```text
Every 5 minutes
Every hour
Every day
Every month
Every day of the week
```

## Configure Root's Crontab

Because the task required the job to run as `root`, I edited root's crontab:

```bash
sudo crontab -e
```

I added:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

## Verification

I verified the root crontab:

```bash
sudo crontab -l
```

The expected entry was:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

I also confirmed that `crond` was active:

```bash
sudo systemctl is-active crond
```

Finally, after the scheduled job had time to execute, I checked:

```bash
cat /tmp/cron_text
```

The result was:

```text
hello
```

This confirmed that the cron scheduler successfully executed the task.

## Real-World Relevance

Cron jobs are commonly used for recurring operational tasks such as:

- Database backups
- Log rotation
- Temporary-file cleanup
- Monitoring checks
- Report generation
- File synchronisation
- Scheduled maintenance scripts
- Security scans
- Deployment housekeeping

A scheduled task is only useful if both the schedule and the service responsible for running it are functioning correctly.

## Operational Lesson

There are several layers to verify when configuring automation:

```text
Package installed
      ↓
Service running
      ↓
Schedule configured
      ↓
Correct user context
      ↓
Job executes
      ↓
Expected output produced
```

Simply seeing a cron entry is not enough to prove that automation works.

## Security Considerations

Cron jobs running as `root` have full system privileges.

For production environments:

- Root cron jobs should only be used when elevated privileges are necessary.
- Scripts executed by root should not be writable by untrusted users.
- Absolute paths should be preferred in important automation scripts.
- Job output and failures should be monitored.
- Credentials should never be hardcoded directly into cron entries.

## Progress So Far

```text
Day 1 – Service accounts
Day 2 – Temporary account expiry
Day 3 – SSH hardening
Day 4 – File permissions
Day 5 – SELinux
Day 6 – Scheduled automation with cron
```

Day 6 adds automation to the Linux administration foundation built during the previous challenges.

## Key Takeaways

1. `cronie` provides cron functionality on CentOS/RHEL systems.
2. `crond` must be running for scheduled cron jobs to execute.
3. `sudo crontab -e` edits the root user's crontab.
4. `*/5 * * * *` means every five minutes.
5. A scheduled task should be verified by checking its actual output.
6. Root-level automation should be treated carefully because it runs with full privileges.
7. Infrastructure automation should be both configured and observable.

## Commands Used

```bash
sudo dnf install -y cronie

sudo systemctl enable --now crond

rpm -q cronie

sudo systemctl is-active crond

sudo crontab -e

sudo crontab -l

cat /tmp/cron_text
```

Cron entry:

```cron
*/5 * * * * echo hello > /tmp/cron_text
```

---

**Status:** ✅ Completed
