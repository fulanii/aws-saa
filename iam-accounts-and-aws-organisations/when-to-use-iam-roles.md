# **When to Use IAM Roles**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> The real situations where an IAM role is the right answer instead of an IAM user, and why. For how roles work (trust policy, permissions policy, STS), see [IAM Roles](iam-roles.md).

### Contents

- [Quick decision table](#quick-decision-table)
- [1. AWS services acting for you](#1-aws-services-acting-for-you)
- [2. Emergency (break-glass) access](#2-emergency-break-glass-access)
- [3. Corporate identities and SSO](#3-corporate-identities-and-sso)
- [4. Mobile and web apps with many users](#4-mobile-and-web-apps-with-many-users)
- [5. Cross-account access](#5-cross-account-access)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### Quick decision table

| Situation | Why not an IAM user? | Use |
|---|---|---|
| Lambda, EC2 or ECS needs to call AWS | Access keys stored in code or on servers can leak and must be rotated | **Service role** (e.g. Lambda execution role) |
| Staff need extra permissions only in emergencies | Giving everyone admin all the time breaks least privilege | **Break-glass role** they assume when needed |
| Company already has Active Directory or SSO | Duplicating every employee as an IAM user, and the 5,000 limit | **Identity federation** → roles |
| Mobile or web app with thousands to millions of users | 5,000 user limit, and you can't put AWS keys in an app | **Web identity federation** (e.g. Cognito) → roles |
| Another AWS account needs access to your resources | Creating users for outsiders means managing their credentials | **Cross-account role** |

---

### 1. AWS services acting for you

**The problem**: your Lambda function needs to read from S3 and write to DynamoDB. It needs AWS permissions from somewhere.

- **Bad option**: create an IAM user, then hardcode its access keys in the function's code or environment variables. The keys are long term, can leak, and have to be rotated by hand.
- **Better option**: give the function a **Lambda execution role**. Its trust policy lets the **Lambda service** assume it.

```
 Lambda invoked ──> Lambda service assumes the execution role (sts:AssumeRole)
                        │
                        v
            Temporary credentials injected into the runtime environment
                        │
                        v
            Your code calls S3 / DynamoDB, allowed by the role's permissions policy
```

- Lambda can run **one copy or hundreds at once**: an unknown number of principals, the classic signal for a role.
- **No credentials in the code**. The SDK picks up the temporary ones automatically.
- The same idea applies to **EC2** (an instance role, attached through an *instance profile*) and **ECS tasks** (a task role). These are covered in more depth later in the course.

---

### 2. Emergency (break-glass) access

**The problem**: helpdesk staff normally need **read-only** access. Occasionally, during an incident at 3am, someone needs to fix things.

- Giving everyone admin all the time breaks **least privilege**.
- Instead, create an **emergency role** with the extra permissions, and allow the helpdesk's identities to assume it.
- Like **breaking the glass** on a fire alarm: anyone *can* do it, but it's deliberate, temporary, and **logged** (CloudTrail records who assumed the role and when).

---

### 3. Corporate identities and SSO

**The problem**: a company already has an identity system (e.g. **Microsoft Active Directory**) with thousands of staff, and wants them to use AWS.

- External identities like AD accounts **can't be used directly** in AWS: IAM only allows its own identities.
- Recreating every employee as an IAM user means double management, and hits the **5,000 user limit**.
- Instead, use **identity federation / SSO**: staff log in with their normal company account, then **assume a role** in AWS. One set of credentials for everything, and no IAM user per person.

---

### 4. Mobile and web apps with many users

**The problem**: a mobile app with **millions of users** needs to store data in DynamoDB or upload photos to S3.

- You can't create millions of IAM users (5,000 limit).
- You must **never embed AWS access keys in an app**: anyone can extract them.
- Instead, use **web identity federation**: users log in with an existing identity (**Google, Facebook, Apple, Amazon**, usually through **Amazon Cognito**), exchange that login for **temporary credentials**, and assume a role that only allows what the app needs.
- **Scales to millions of users**, no AWS credentials stored in the app, and users reuse logins they already have.

---

### 5. Cross-account access

**The problem**: a partner company (a different AWS account) needs to upload files to a bucket in your account. Or your company has separate `dev` and `prod` accounts.

- Creating IAM users in your account for their staff means managing credentials for people you don't control.
- Instead, create a role in **your** account whose **trust policy** allows the **other account**. Their identities assume the role and get temporary access to only what its permissions policy allows.
- This is how access between accounts works in **AWS Organizations** too (coming later in this section).

---

### Django/FastAPI map

- **Your app on AWS**: run Django/FastAPI on EC2, ECS or Lambda with a role attached. `boto3` gets the credentials itself, so there's no `AWS_ACCESS_KEY_ID` in `.env` or `settings.py`.
- **Your app's users are not IAM users**. They log in through your own auth. If the frontend or mobile app needs to talk to AWS directly (e.g. upload straight to S3), use **Cognito** with web identity federation, or have your API hand out **presigned URLs**.
- **CI/CD**: GitHub Actions can assume a role through **OIDC federation** instead of storing an IAM user's keys as repo secrets. Same idea as the web identity case.
- **Prod access for the team**: developers have read-only access day to day, and assume a break-glass role to run a migration or fix a production incident.

---

### Exam notes

- **AWS service needs permissions** (Lambda, EC2, ECS) → **role**, never access keys stored in code or on the server.
- **Lambda execution role** = the role Lambda assumes to run your function.
- **Break-glass / emergency access** → a role that's assumed only when needed, and logged.
- **Existing corporate identities (Active Directory), SSO, more than 5,000 staff** → **identity federation + roles**.
- **Mobile or web app with many users, Google/Facebook login** → **web identity federation** (Cognito) + roles. **Never embed AWS keys in an app.**
- **Another AWS account needs access** → **cross-account role** whose trust policy allows that account.
- General signal: **unknown or very large number of principals**, **temporary access**, or **identities from outside IAM** → **role**.
