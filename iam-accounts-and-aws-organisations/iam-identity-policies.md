# **IAM Identity Policies**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> JSON documents attached to an identity (user, group or role) that say which AWS actions it is allowed or denied on which resources.

### Contents

- [What it is](#what-it-is)
- [Policy document structure](#policy-document-structure)
- [Parts of a statement](#parts-of-a-statement)
- [How AWS decides: allow vs deny](#how-aws-decides-allow-vs-deny)
- [Inline vs managed policies](#inline-vs-managed-policies)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

An **IAM identity policy** is a set of security statements that you **attach to an identity** in AWS. An identity is an **IAM user**, **IAM group** or **IAM role**.

- The policy **allows or denies** access to AWS products and features.
- Any identity that has the policy attached gets those permissions.
- Policies are written in **JSON**. They are also called **policy documents**.
- One identity can have **many policies** attached, and each policy can have **many statements**. AWS combines all of them when it decides what the identity can do.

---

### Policy document structure

At a high level, a policy document is just **one or more statements** inside a `Statement` list.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListSpecificBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::my-example-bucket"
    },
    {
      "Sid": "AllowReadObjectsInBucket",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-example-bucket/*"
    }
  ]
}
```

- **`Version`**: the version of the policy language, not a date you pick. Always use `"2012-10-17"`, the latest one. Some newer features don't work with the older `2008-10-17` version.
- **`Statement`**: the list of statements. Statements are what actually **grant or deny** access.

A statement only applies when a request to AWS **matches both its `Action` and its `Resource`**. If either doesn't match, the statement is ignored for that request.

> Notice the two different resources above. `ListBucket` acts on the **bucket**, so it uses the bucket ARN. `GetObject` acts on **objects inside** the bucket, so it needs `/*` at the end. Mixing these up is a very common mistake.

---

### Parts of a statement

| Field | Required? | What it does | Example |
|---|---|---|---|
| **`Sid`** | Optional | Statement ID. A name that says what the statement is for. Best practice is to always add one, whatever the size of the policy. | `"AllowReadObjectsInBucket"` |
| **`Effect`** | Yes | Either `Allow` or `Deny`. What happens when the statement matches. | `"Allow"` |
| **`Action`** | Yes | Which operations the statement covers. Format is `service:operation`. | `"s3:GetObject"` |
| **`Resource`** | Yes | Which AWS resources the statement covers. Written in **ARN** format. | `"arn:aws:s3:::my-example-bucket/*"` |

`Action` can be written 3 ways:

1. **One action**: `"s3:GetObject"`
2. **A wildcard**: `"s3:*"` matches every S3 action. `"s3:Get*"` matches every S3 action starting with `Get`.
3. **A list of actions**: `["s3:GetObject", "s3:PutObject"]`

`Resource` works the same way: one ARN, a wildcard (`"*"` for everything, or `"arn:aws:s3:::my-example-bucket/*"`), or a list of ARNs.

> An **ARN** (Amazon Resource Name) is the unique ID of a resource in AWS: `arn:partition:service:region:account-id:resource`. S3 bucket ARNs leave region and account empty (`arn:aws:s3:::bucket-name`) because bucket names are globally unique.

---

### How AWS decides: allow vs deny

When an identity makes a request, AWS looks at **all** the policies that apply to it and follows three rules, in this order:

| Order | Rule | Meaning |
|---|---|---|
| 1 | **Explicit deny** | If any matching statement says `Deny`, the request is **denied**. Nothing can override it. |
| 2 | **Explicit allow** | If there is no deny and a matching statement says `Allow`, the request is **allowed**. |
| 3 | **Default (implicit) deny** | If nothing matches at all, the request is **denied**. Everything starts out denied. |

**Deny, Allow, Deny**: explicit deny wins, then explicit allow, and anything not mentioned is denied by default.

- A new IAM user with no policies **can do nothing**. That's the implicit deny.
- The only exception is the **account root user**: it has full access by default and identity policies can't restrict it.

Example: allow everything in S3, but block one sensitive bucket. The explicit deny wins for that bucket, even though `s3:*` allows it.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAllS3",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    },
    {
      "Sid": "DenySecretBucket",
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": ["arn:aws:s3:::secret-bucket", "arn:aws:s3:::secret-bucket/*"]
    }
  ]
}
```

---

### Inline vs managed policies

There are 2 types of identity policy. The JSON inside is **exactly the same**. The difference is **how they are managed**.

**Inline policy**: written **directly into one identity**.

- It belongs only to that identity. If 3 users need the same access, you create 3 separate copies.
- Changing access means editing every copy one by one.
- Deleting the identity deletes its inline policies too.
- Use it for **special cases or exceptions**, e.g. one user who needs a single extra permission or a deny.

**Managed policy**: a **standalone object** in IAM that you attach to as many identities as you want.

- Create it once, attach it to many users, groups or roles.
- Update it once and every identity using it gets the change.
- Two kinds:
  - **AWS managed**: created and maintained by AWS, e.g. `AdministratorAccess`, `ReadOnlyAccess`, `AmazonS3ReadOnlyAccess`. You can't edit them.
  - **Customer managed**: created and maintained by you, so you can shape them to your exact needs.
- Use it for **normal, reusable access rights**. This should be your default choice.

| | Inline | Managed |
|---|---|---|
| Lives | Inside one identity | As its own object in IAM |
| Reusable | No | Yes, attach to many identities |
| Updating | Edit every copy | Edit once, applies everywhere |
| Best for | Exceptions and one-off rights | Default, shared access rights |

---

### Django/FastAPI map

- **A policy is like permission rules in Django.** `Allow` statements are like giving a user a permission (`app.view_model`). The implicit deny is like a new Django user having no permissions until you add some.
- **Managed policies are like Django Groups**: define the permissions once, then attach them to many identities.
- **Least privilege for your app**: if your Django app uses `django-storages` to upload files to S3, give its IAM role only `s3:PutObject` and `s3:GetObject` on `arn:aws:s3:::your-media-bucket/*`, not `s3:*` on `*`.
- **Explicit deny is your guardrail**: e.g. deny `s3:DeleteObject` on the backups bucket so even a buggy Celery task can never delete backups.

---

### Exam notes

- Identity policies are **JSON**, attached to **users, groups or roles**. A statement needs **Effect, Action and Resource**. `Sid` is optional.
- Evaluation order: **Explicit Deny > Explicit Allow > Default (implicit) Deny**. An explicit deny **always wins**.
- **No policy = no access**. Everything is denied by default (except for the account root user).
- **Inline** = attached to one identity, for exceptions. **Managed** = reusable, the default choice. Know **AWS managed** vs **customer managed**.
- AWS managed policies **can't be edited**. To customise one, copy it into a customer managed policy.
- A question saying "many users need the same permissions and it should be easy to update" is pointing at a **managed policy** (usually attached to a group).
- S3 gotcha: bucket actions (`s3:ListBucket`) use `arn:aws:s3:::bucket`, object actions (`s3:GetObject`) use `arn:aws:s3:::bucket/*`.
