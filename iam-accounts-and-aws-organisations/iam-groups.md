# **IAM Groups**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> Containers for IAM users that make it easier to manage permissions for many users at once. A group is not an identity: you can't log in as one.

### Contents

- [What it is](#what-it-is)
- [How permissions combine](#how-permissions-combine)
- [What groups are not](#what-groups-are-not)
- [Limits](#limits)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

An **IAM group** is a **container for IAM users**. Groups exist only to make **organising and managing large sets of IAM users** easier.

- You attach policies to the group (**inline** or **managed**), and every user in the group gets those permissions.
- An IAM user can be a member of **multiple groups**.
- Groups make admin-style management effective: one group per **team**, **role** or **project**, e.g. `Developers`, `QA`, `Admins`.
- A new user joining the team just gets added to the right group. No policies to copy.

```
 Group: Developers  ── managed policy: DevAccess
   ├── alice
   └── bob
 Group: QA          ── managed policy: QAAccess
   ├── bob          ← bob is in both groups
   └── carol
```

---

### How permissions combine

A user's effective permissions are **everything that applies to them**:

- Policies attached **directly to the user** (inline or managed)
- **Plus** the policies of **every group** the user belongs to

AWS evaluates all of them together, with the usual rules: **explicit deny > explicit allow > implicit deny**. So in the example above, `bob` gets `DevAccess` **and** `QAAccess`. If any of those policies has a `Deny` that matches a request, the deny wins, wherever it came from.

---

### What groups are not

- **Not a login**: you **can't log in to a group**. It has **no credentials of its own** (no password, no access keys). A common exam trick.
- **Not a true identity**: a group **can't be referenced as a principal** in a policy. For example, an S3 **bucket policy** can grant access to a user or a role, but not to a group.
- **No built-in "all users" group**: if you want a group with every user in it, you have to create it and add the users yourself.
- **No nesting**: groups can't contain other groups. Only users go in groups.

---

### Limits

| Limit | Value |
|---|---|
| Groups per account | **300** (can be increased with a support ticket) |
| Groups per user | **10** |

---

### Django/FastAPI map

- **IAM groups are almost exactly Django's `Group` model**: you give the group permissions, add users to it, and each user gets the group's permissions plus their own.
- The difference: a Django `Group` can't log in either, but in IAM this is a **hard rule** the exam checks. Only users (and roles) authenticate.
- Typical setup for a small team: an `Admins` group with `AdministratorAccess`, a `Developers` group with a customer managed policy for the services your API uses, and every human IAM user added to one of them. No policies attached to individual users.

---

### Exam notes

- A group is a **container for users**, used **only** to manage permissions for many users.
- **You can't log in to a group**; it has **no credentials**.
- A group is **not a true identity** and **can't be a principal** in a resource policy (e.g. an S3 bucket policy).
- **No nesting** of groups, and **no built-in "all users"** group.
- A user can be in **multiple groups** (max **10**). Their permissions = user policies **+** all group policies, with explicit deny still winning.
- **300 groups per account** by default, increasable via a support ticket.
- "Many users need the same permissions" → put them in a **group** with a **managed policy**.
