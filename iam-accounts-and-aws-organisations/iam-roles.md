# **IAM Roles**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> An identity that represents a level of access, not a person. Principals **assume** a role for a while and get temporary credentials, instead of having long term credentials of their own.

### Contents

- [What it is](#what-it-is)
- [User or role?](#user-or-role)
- [The two policies of a role](#the-two-policies-of-a-role)
- [How assuming a role works](#how-assuming-a-role-works)
- [Example policies](#example-policies)
- [Common uses](#common-uses)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

An **IAM role** is one of the two types of **identity** in an AWS account. The other is the **IAM user**. (Groups are not identities, see [IAM Groups](iam-groups.md).)

- A role doesn't represent **you**. It represents **a level of access** inside an AWS account.
- It's meant to be **used temporarily**: a principal **assumes** the role, uses its access for a while, then stops.
- It's best for an **unknown number of principals**, or **many** of them: AWS users, humans, applications, AWS services, inside **or outside** the account.
- A role has **no long term credentials**: no password and no access keys of its own.

---

### User or role?

| | IAM user | IAM role |
|---|---|---|
| **Represents** | One specific person or app | A level of access |
| **Who uses it** | One named principal | Any number of principals, often unknown |
| **Credentials** | Long term (password, access keys) | **Temporary**, issued when the role is assumed |
| **How you use it** | Log in as it | **Assume** it |

Rule of thumb: if you **can't say exactly who or how many** principals will use the identity (or there are more than **5,000**, the IAM user limit), it's a candidate for a **role**.

---

### The two policies of a role

| Policy | Answers | Type |
|---|---|---|
| **Trust policy** | **Who** is allowed to assume the role? | A resource policy attached to the role. It names **principals**: identities in the same account, other AWS accounts, AWS services (e.g. EC2, Lambda), or external identity providers (e.g. Google, a company SSO) |
| **Permissions policy** | **What** can they do once they've assumed it? | A normal identity policy (inline or managed), same JSON as for users |

- If a principal isn't allowed by the trust policy, it can't assume the role at all.
- Once assumed, the permissions policy decides what the temporary credentials can access, with the usual **explicit deny > explicit allow > implicit deny** rules.

---

### How assuming a role works

```
 Principal                          STS                                AWS resource
 (user, app, AWS service)           ───                                ────────────
     │
     │ 1. sts:AssumeRole on the role
     ├──────────────────────────> 2. Check the role's TRUST policy:
     │                               is this principal allowed?
     │                                 │ yes
     │ 3. Temporary credentials <──────┘
     │    (access key ID + secret + session token, expire after a set time)
     │
     │ 4. Request using the temporary credentials
     ├───────────────────────────────────────────────────────────────>  5. Check the role's
     │                                                                     PERMISSIONS policy
     │                                                                     → allow or deny
     │
     │ 6. Credentials expire → assume the role again to get new ones
```

- **`sts:AssumeRole`** is the call that generates the temporary credentials. **STS** (Security Token Service) is the AWS service that issues them.
- Temporary credentials are like access keys, but they **expire**: they include a **session token**, and once they expire they stop working. To keep going, the principal assumes the role again and gets a new set.
- Because nothing long term is handed out, there are no keys to leak, rotate or forget about.

---

### Example policies

**Trust policy**: let EC2 instances assume the role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Trust policy**: let another AWS account assume the role (cross-account access).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::111122223333:root" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

> `:root` here means **the whole account 111122223333**, not its root user. That account still has to give its own users permission to call `sts:AssumeRole` on this role.

**Permissions policy**: what the role can do once assumed (same format as any [identity policy](iam-identity-policies.md)).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadWriteMediaBucket",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::my-media-bucket/*"
    }
  ]
}
```

---

### Common uses

Each of these is covered in detail, with the reasons behind it, in [When to Use IAM Roles](when-to-use-iam-roles.md). In short:

- **AWS services acting for you**: an EC2 instance or Lambda function assumes a role to call other AWS services, with no access keys stored on the server.
- **Cross-account access**: users in one AWS account assume a role in another.
- **Identity federation**: people who log in with Google, a company SSO or Cognito assume a role, which avoids the 5,000 IAM user limit.
- **Break-glass access**: a role with extra permissions that someone assumes only in an emergency.

---

### Django/FastAPI map

- **Instead of access keys in `.env`**: attach a role to the EC2 instance, ECS task or Lambda running your app. `boto3` finds the temporary credentials **automatically**, so `boto3.client("s3")` just works with no `AWS_ACCESS_KEY_ID` anywhere.
- **Expiry is handled for you**: on AWS compute, the SDK refreshes the temporary credentials before they expire. Your code never sees it.
- **A role is like a temporary "sudo"**: you take on a set of permissions for a while, then drop them.
- **Least privilege**: the permissions policy for a `django-storages` app only needs `s3:GetObject` and `s3:PutObject` on your media bucket, as in the example above.

---

### Exam notes

- IAM identities are **users** and **roles**. Groups are not identities.
- Roles are for an **unknown or large number of principals**, inside or outside the account (more than **5,000** points to roles).
- A role has **two policies**: the **trust policy** (**who** can assume it) and the **permissions policy** (**what** it can do).
- Assuming a role calls **`sts:AssumeRole`**. **STS** issues **temporary credentials** that expire, so there are no long term keys.
- A role has **no long term credentials** of its own.
- Unlike a group, a role **can be a principal** in a resource policy (e.g. an S3 bucket policy).
- "Application on EC2 needs to access S3 securely" → **an IAM role attached to the instance**, never access keys stored on the server.
