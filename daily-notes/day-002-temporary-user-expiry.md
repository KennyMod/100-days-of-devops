# Day 2 – Temporary Linux User with an Expiry Date

## Challenge

A developer required temporary access to an application server for a limited period.

The task was to create a Linux user named `mark` on Application Server 3 and set the account to expire on `2027-03-28`.

## What I Learned

- Creating Linux users with `useradd`
- Setting account expiration dates
- The difference between account expiration and password expiration
- Inspecting account aging information with `chage`
- Verifying Linux user creation with `getent` and `id`
- Why temporary access should expire automatically
- How account lifecycle management contributes to system security

## Implementation

I first connected to the required application server:

```bash
ssh banner@stapp03
```

I then created the temporary user and assigned the required expiration date:

```bash
sudo useradd -e 2027-03-28 mark
```

### Command Breakdown

```text
sudo
└── Run the command with elevated privileges

useradd
└── Create a new Linux user

-e
└── Set the account expiration date

2027-03-28
└── Expiration date in YYYY-MM-DD format

mark
└── Username being created
```

## Verification

After creating the account, I verified that the configuration was correct rather than assuming the command had succeeded.

### Verify the user exists

```bash
getent passwd mark
```

The account was returned successfully with a normal login shell:

```text
mark:x:1001:1001::/home/mark:/bin/bash
```

### Verify UID and group membership

```bash
id mark
```

This confirmed that Linux had created a UID and primary group for the new user.

### Verify the account expiration date

```bash
sudo chage -l mark
```

The important result was:

```text
Account expires : Mar 28, 2027
```

This confirmed that the account would automatically expire on the required date.

## Account Expiration vs Password Expiration

One important distinction from this exercise is that account expiration and password expiration are not the same thing.

### Account expiration

The entire account becomes unusable after a specified date.

For this task:

```text
Account expires: March 28, 2027
```

### Password expiration

The account can remain active, but the user may be required to change their password after a defined period.

For temporary employees, contractors, consultants, or project users, account expiration is useful because access automatically ends when it is no longer required.

## Real-World Relevance

Temporary accounts are commonly created for:

- Contractors
- Consultants
- Temporary developers
- Vendors
- Short-term project staff
- Emergency or time-limited access

Without an expiration policy, these accounts may remain active long after the person no longer requires access.

This can increase the attack surface of a server and create unnecessary security risk.

By setting an expiry date when the account is created, access lifecycle management is enforced automatically rather than relying on someone to remember to manually disable the account later.

## Day 1 vs Day 2

Day 1 focused on restricting the capabilities of a service account:

```text
Service account
      ↓
Non-interactive shell
      ↓
/sbin/nologin
      ↓
No interactive login
```

Day 2 focused on restricting the lifetime of a human account:

```text
Temporary developer
      ↓
Normal interactive shell
      ↓
Expiration date
      ↓
Access automatically ends
```

Both approaches apply the principle of least privilege.

The account receives only the access and lifetime required for its intended purpose.

## Security Considerations

A few security practices demonstrated by this task are:

- Temporary access should have a defined end date.
- Unnecessary accounts should not remain active indefinitely.
- Account configuration should always be verified after creation.
- Sensitive files such as `/etc/shadow` should not be exposed in public documentation.
- Access controls should match the purpose of the identity being created.

## Key Takeaways

1. `useradd -e` can be used to assign an expiration date when creating a Linux user.
2. Linux account expiration and password expiration are separate controls.
3. `chage -l` provides useful information about account aging and expiration.
4. `getent passwd` is useful for confirming Linux account information.
5. `id` confirms UID, GID, and group membership.
6. Temporary access should expire automatically whenever possible.
7. Configuration changes should always be verified.

## Commands Used

```bash
ssh banner@stapp03

sudo useradd -e 2027-03-28 mark

getent passwd mark

id mark

sudo chage -l mark
```

---

**Status:** ✅ Completed
