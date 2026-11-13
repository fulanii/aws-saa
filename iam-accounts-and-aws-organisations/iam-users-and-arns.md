# **IAM Users and ARNs**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> IAM users are long term identities for humans, apps and service accounts. ARNs are the unique names AWS uses to point at any resource.

### Contents

- [What an IAM user is](#what-an-iam-user-is)
- [Principals](#principals)
- [Authentication and authorization](#authentication-and-authorization)
- [ARNs (Amazon Resource Names)](#arns-amazon-resource-names)
- [Limits and why they matter](#limits-and-why-they-matter)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What an IAM user is

An **IAM user** is an identity for anything that needs **long term access** to an AWS account:

- **Humans**: e.g. you, a teammate, a contractor.
- **Applications**: e.g. a backup script running on your own server.
- **Service accounts**: e.g. a CI/CD tool like GitHub Actions deploying to AWS.

Simple rule: if you can **name one specific person or app** that needs access, an IAM user is usually the right choice. (Later sections cover IAM roles, which are better when access is short term or shared by many unknown identities.)

- IAM is a **global** service. Users are not tied to a region.
- A new IAM user has **no permissions** until you attach a policy (the implicit deny from [IAM Identity Policies](iam-identity-policies.md)).

---

### Principals

Everything in IAM starts with a **principal**.

A principal is **an entity trying to access an AWS account**. At the start, AWS doesn't know who it is: it is **unidentified**.

A principal can be:

- An individual person
- A computer or server
- An application or service
- A group of any of those

For a principal to do anything, it must be **authenticated** (prove who it is) and then **authorized** (be allowed to do the action).

---

### Authentication and authorization

```
 Principal                  IAM                                   AWS resource
 (unidentified)             ───                                   ────────────
     │
     │ 1. Authenticate: username + password, or access keys
     ├──────────────────> "Are you really iam-user-alice?"
     │                         │ yes
     │                         v
     │                  Authenticated identity
     │                         │
     │ 2. Request: s3:GetObject on catgifs/cat.gif
     ├──────────────────> 3. Authorize: check all policy
     │                       statements for this identity
     │                         │ allowed
     │                         v
     └───────────────────────────────────────────────────>  S3 object returned
```

**Authentication**: the principal **proves to IAM** that it is the identity it claims to be. Two ways for an IAM user:

| Method | Used for | What it is |
|---|---|---|
| **Username and password** | Signing in to the **AWS console** (humans) | Normal login. Can (and should) add MFA. |
| **Access keys** | **CLI, SDKs and APIs** (scripts, apps, humans on the command line) | An access key ID + a secret access key. An IAM user can have **max 2** access keys, so you can rotate them. |

Both are **long term credentials**: they don't expire on their own. You have to change or rotate them yourself.

Once authentication succeeds, the principal becomes an **authenticated identity**. AWS now knows exactly which identity is making the requests.

**Authorization**: for every request, IAM **checks all the policy statements that apply to that identity** and decides to allow or deny it, using the rules Explicit Deny > Explicit Allow > Default Deny.

> Authentication = **who are you?** Authorization = **what are you allowed to do?**

---

### ARNs (Amazon Resource Names)

> An ARN **uniquely identifies a resource** within any AWS account.

When you work with resources from the **CLI or APIs**, you need a way to point at them **without any confusion** (e.g. there can be many buckets or instances, in many accounts and regions). ARNs solve this. They are also what you put in the `Resource` field of a policy.

An ARN can point at **one resource**, or at **a group of resources** using a wildcard `*`.

**Formats** (which one depends on the service):

```
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

| Part | Meaning | Example |
|---|---|---|
| `partition` | Which group of AWS regions. Almost always `aws`. Others: `aws-cn` (China), `aws-us-gov` (GovCloud). | `aws` |
| `service` | The AWS service | `s3`, `ec2`, `iam` |
| `region` | The region the resource is in. **Empty** for global services. | `us-east-1` |
| `account-id` | The 12 digit AWS account ID that owns it. **Empty** when not needed. | `123456789012` |
| `resource` | The resource itself, sometimes with a type in front | `instance/i-0abc123`, `user/alice` |

**Examples:**

| ARN | Points at |
|---|---|
| `arn:aws:s3:::catgifs` | The **bucket** called `catgifs` |
| `arn:aws:s3:::catgifs/*` | **All objects inside** the `catgifs` bucket (not the bucket itself) |
| `arn:aws:ec2:us-east-1:123456789012:instance/i-0abc123` | One EC2 instance in `us-east-1` |
| `arn:aws:iam::123456789012:user/alice` | The IAM user `alice` (no region, because IAM is global) |

- **`::` means a part was left empty, not a wildcard.** S3 has no region or account in its ARN because bucket names are already globally unique. IAM has no region because it is global.
- `catgifs` and `catgifs/*` **do not overlap**. One is the bucket, the other is the objects in it. If a policy needs both bucket actions and object actions, it needs both ARNs.

---

### Limits and why they matter

- Max **5,000 IAM users per account**.
- An IAM user can be a member of max **10 groups**.

These limits have **system design impacts**:

| Situation | Problem | Solution |
|---|---|---|
| **Internet-scale apps** (e.g. a mobile app with millions of users) | You can't create an IAM user for every customer | **IAM roles** with **web identity federation** (e.g. Amazon Cognito, sign in with Google) |
| **Large organisations** (more than 5,000 staff) | Not enough IAM users for every employee | **Identity federation / SSO**: staff log in with the company's existing identity system (e.g. Active Directory) and use **IAM roles** |
| **Organisation merges** | Two companies' users need access, and numbers grow fast | Same: **IAM roles and identity federation** instead of one IAM user each |

---

### Django/FastAPI map

- **Your Django app's users are not IAM users.** Customers sign up in your app's own auth system (`django.contrib.auth`, JWTs in FastAPI). IAM users are for people and tools that access **AWS itself**.
- **Access keys are like an API key plus secret** for AWS. `boto3` reads them from `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` or `~/.aws/credentials`. Never commit them to git or put them in `settings.py`.
- **Authentication vs authorization** is the same idea as in Django: logging in (`authenticate()`) vs checking permissions (`has_perm()`).
- Running on EC2, ECS or Lambda? Don't give the app an IAM user's access keys. Use an **IAM role**, which gives temporary credentials automatically (covered later).

---

### Exam notes

- IAM users = **long term access** for humans, apps and service accounts you can name.
- **Principal** = an unidentified entity trying to access AWS. After authentication it becomes an **authenticated identity**.
- **Authentication** = proving who you are (password for the console, access keys for the CLI/API). **Authorization** = IAM checking policies to allow or deny.
- Max **2 access keys** per IAM user.
- ARN format: `arn:partition:service:region:account-id:resource`. Empty parts (`:::`) are not wildcards.
- `arn:aws:s3:::bucket` is the bucket, `arn:aws:s3:::bucket/*` is the objects. They don't overlap.
- Limits: **5,000 IAM users per account**, **10 groups per user**.
- A question about **millions of users** or **more than 5,000 staff** needing AWS access is pointing at **IAM roles and identity federation**, not IAM users.
