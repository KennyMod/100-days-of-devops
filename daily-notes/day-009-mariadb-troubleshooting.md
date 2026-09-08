# Day 9 – Troubleshoot and Restore MariaDB Service

## Challenge

The Nautilus application was unable to connect to its database.

The production support team identified that the MariaDB service on the database server was down.

The task was to investigate the issue, identify the root cause, restore the database service, and verify that MariaDB was operational again.

## What I Learned

- How to troubleshoot a failed `systemd` service
- How to inspect service logs using `journalctl`
- How to distinguish between a stopped service and a failed service
- How file ownership can prevent an application from starting
- Why MariaDB requires correct ownership of its data directory
- How to verify database availability at multiple layers
- Why troubleshooting should follow evidence rather than assumptions

## Initial Investigation

I first connected to the database server:

```bash
ssh peter@stdb01
```

I checked the MariaDB service:

```bash
sudo systemctl status mariadb --no-pager -l
```

The service was:

```text
inactive (dead)
```

I also confirmed that MariaDB was installed:

```bash
rpm -q mariadb-server
```

and checked whether anything was listening on the default database port:

```bash
sudo ss -lntp | grep 3306
```

Nothing was listening on port `3306`.

## First Start Attempt

I attempted to start and enable MariaDB:

```bash
sudo systemctl enable --now mariadb
```

The service failed to start.

This confirmed that the incident was not simply caused by MariaDB being stopped.

## Investigate the Failure

I inspected the service logs:

```bash
sudo systemctl status mariadb --no-pager -l
```

and:

```bash
sudo journalctl -xeu mariadb.service --no-pager
```

The important errors were related to the MariaDB data directory:

```text
chown: changing ownership of '/var/lib/mysql': Operation not permitted

Cannot change ownership of the database directories to the 'mysql' user.

Initialization of MariaDB database failed.
```

This pointed to a permissions or ownership problem with:

```text
/var/lib/mysql
```

## Inspect the Data Directory

I checked the directory:

```bash
sudo ls -ld /var/lib/mysql
```

The ownership was:

```text
root:mysql
```

MariaDB needs its data directory to be owned by the `mysql` service account.

The expected ownership was:

```text
mysql:mysql
```

## Root Cause

The MariaDB service was unable to initialise its database because the data directory had incorrect ownership.

The root cause was:

```text
/var/lib/mysql

Owner: root
Group: mysql
```

instead of:

```text
Owner: mysql
Group: mysql
```

## Fix

I corrected the ownership recursively:

```bash
sudo chown -R mysql:mysql /var/lib/mysql
```

I then verified the directory:

```bash
sudo ls -ld /var/lib/mysql
```

The ownership now showed:

```text
mysql mysql
```

## Restore the Service

I started MariaDB again:

```bash
sudo systemctl start mariadb
```

I also ensured that the service was enabled for future reboots:

```bash
sudo systemctl enable mariadb
```

## Verification

I verified the service state:

```bash
sudo systemctl is-active mariadb
```

Result:

```text
active
```

I inspected the full service status:

```bash
sudo systemctl status mariadb --no-pager -l
```

The service reported:

```text
Active: active (running)
```

## Network Verification

I verified that MariaDB was listening on its default TCP port:

```bash
sudo ss -lntp | grep 3306
```

The result showed MariaDB listening on:

```text
3306
```

## Database-Level Verification

Finally, I verified that the database engine could execute a query:

```bash
sudo mariadb -e "SELECT 1;"
```

The query returned successfully:

```text
1
1
```

This confirmed that MariaDB was not only running but was actually able to process SQL requests.

## Troubleshooting Workflow

The incident followed this troubleshooting path:

```text
Application cannot reach database
            ↓
Check MariaDB service
            ↓
Service inactive
            ↓
Attempt to start service
            ↓
Service startup fails
            ↓
Inspect systemd logs
            ↓
Database directory ownership error
            ↓
Inspect /var/lib/mysql
            ↓
Incorrect owner: root:mysql
            ↓
Correct ownership to mysql:mysql
            ↓
Restart MariaDB
            ↓
Verify service
            ↓
Verify TCP port 3306
            ↓
Run SQL query
            ↓
Database restored
```

## Why This Matters in DevOps

Production incidents often begin with only a symptom:

```text
"The application cannot connect to the database."
```

The actual root cause may exist several layers below the application.

A structured troubleshooting process helps avoid unnecessary changes.

Instead of immediately modifying configuration files or reinstalling the database, I worked through the system one layer at a time:

```text
Application
    ↓
Service
    ↓
Logs
    ↓
Filesystem
    ↓
Permissions / ownership
    ↓
Network
    ↓
Database
```

## Real-World Relevance

This type of incident can happen because of:

- Incorrect ownership changes
- Restore operations
- Deployment scripts
- Manual administrator mistakes
- Migration procedures
- Files copied under the wrong account
- Misconfigured automation

Database data directories require strict ownership and permission controls.

If the database service cannot access its own files, it may be unable to initialise or start.

## Security Considerations

I avoided unsafe fixes such as:

```bash
chmod 777 /var/lib/mysql
```

because granting unrestricted permissions would introduce unnecessary security risk.

Instead, I corrected the actual ownership problem:

```bash
sudo chown -R mysql:mysql /var/lib/mysql
```

This restored the required access while maintaining proper privilege boundaries.

## Key Takeaways

1. Never assume a stopped service only needs restarting.
2. `systemctl status` provides the first layer of service diagnostics.
3. `journalctl` is critical for identifying the actual cause of a service failure.
4. File ownership can prevent services from starting even when packages and configuration are correct.
5. MariaDB requires its data directory to be accessible by the `mysql` service account.
6. Fix the root cause instead of weakening permissions.
7. Verification should happen at multiple layers.
8. A database service is not fully verified until it can actually execute a query.
9. Troubleshooting methodology is an important DevOps skill.

## Commands Used

```bash
ssh peter@stdb01

sudo systemctl status mariadb --no-pager -l

sudo journalctl -xeu mariadb.service --no-pager

rpm -q mariadb-server

sudo ss -lntp | grep 3306

sudo ls -ld /var/lib/mysql

sudo chown -R mysql:mysql /var/lib/mysql

sudo systemctl start mariadb

sudo systemctl enable mariadb

sudo systemctl is-active mariadb

sudo systemctl status mariadb --no-pager -l

sudo ss -lntp | grep 3306

sudo mariadb -e "SELECT 1;"
```

---

**Status:** ✅ Completed
