# **Service-linked Roles & PassRole**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> Service-linked roles are roles that belong to an AWS service and are defined by it. `iam:PassRole` is the permission that lets you hand an existing role to a service.

### Contents

- [Service-linked roles](#service-linked-roles)
- [Service-linked role vs normal service role](#service-linked-role-vs-normal-service-role)
- [Permissions to create a service-linked role](#permissions-to-create-a-service-linked-role)
- [PassRole](#passrole)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### Service-linked roles

A **service-linked role** is an IAM role **linked to one specific AWS service**.

- **Predefined by the service**: the service decides its trust policy and permissions. Only that service can assume it.
- It gives the service the permissions it needs to **interact with other AWS services on your behalf**. For example, Elastic Load Balancing needs to manage network interfaces in your VPC.
- The service may **create and delete the role for you**, or let you create it **during setup** or **in IAM**.
- You **can't delete the role while it's still needed**: first you have to remove the resources in that service that depend on it.
- Easy to recognize: names start with **`AWSServiceRoleFor`** (e.g. `AWSServiceRoleForElasticLoadBalancing`, `AWSServiceRoleForAutoScaling`) and the path is **`/aws-service-role/`**.

---

### Service-linked role vs normal service role

| | Normal service role | Service-linked role |
|---|---|---|
| **Defined by** | You | The AWS service |
| **Trust and permissions policies** | You write and edit them | Predefined. You **can't edit** them |
| **Created by** | You | Usually the service, or you during setup or in IAM |
| **Deleted** | Whenever you like | Only once the service **no longer needs it** |
| **Example** | A Lambda execution role you write | `AWSServiceRoleForAutoScaling` |

---

### Permissions to create a service-linked role

To let an identity create a service-linked role, it needs `iam:CreateServiceLinkedRole`, plus permission to attach or put policies on that role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:CreateServiceLinkedRole",
      "Resource": "arn:aws:iam::*:role/aws-service-role/SERVICE-NAME.amazonaws.com/SERVICE-LINKED-ROLE-NAME-PREFIX*",
      "Condition": {"StringLike": {"iam:AWSServiceName": "SERVICE-NAME.amazonaws.com"}}
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:AttachRolePolicy",
        "iam:PutRolePolicy"
      ],
      "Resource": "arn:aws:iam::*:role/aws-service-role/SERVICE-NAME.amazonaws.com/SERVICE-LINKED-ROLE-NAME-PREFIX*"
    }
  ]
}
```

> **Don't try to guess the `SERVICE-NAME` and role name prefix.** The format differs between services and it's **case sensitive**. Look it up in each service's documentation ("AWS services that work with IAM" lists them).

---

### PassRole

**The problem**: you want an EC2 instance (or Lambda function, or CloudFormation stack) to run with a role. Someone has to **attach** that role when creating the resource. What stops anyone from attaching an `AdministratorAccess` role to an instance they launch, and getting admin through the back door?

**The answer**: `iam:PassRole`. To give a role to a service, you need **permission to pass that specific role**.

- **Passing** a role means configuring a service to use it, e.g. choosing an instance role when launching EC2.
- You need `iam:PassRole` **on that role**, plus permission for the action itself (e.g. `ec2:RunInstances`).
- You **don't need the role's permissions yourself**. A user with no S3 access can launch an instance with a role that has S3 access, as long as an admin allowed them to pass that role.
- It's **passing, not assuming**: the user never gets the role's credentials. The **service** assumes the role.
- `iam:PassRole` is a **permission only**, not an API call you make. AWS checks it when you create or update the resource.

```
 Bob: ec2:RunInstances + iam:PassRole on "app-ec2-role" (no S3 access himself)
   │
   │ launches an instance with app-ec2-role
   v
 EC2 instance ──assumes──> app-ec2-role ──> reads/writes S3
```

Example policy for Bob: he can pass only `app-ec2-role`, and only to EC2.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "LaunchInstances",
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "*"
    },
    {
      "Sid": "PassOnlyTheAppRoleToEc2",
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::123456789012:role/app-ec2-role",
      "Condition": {"StringEquals": {"iam:PassedToService": "ec2.amazonaws.com"}}
    }
  ]
}
```

This gives **separation of duties**: admins create roles with the right permissions ahead of time, and developers can use them for their deployments without being able to grant themselves more access.

---

### Django/FastAPI map

- **Service-linked roles appear on their own**: the first time you create a load balancer or Auto Scaling group for your API, AWS creates `AWSServiceRoleForElasticLoadBalancing` or `AWSServiceRoleForAutoScaling` in your account. Don't delete them.
- **PassRole in your deploy pipeline**: if GitHub Actions deploys your app with CloudFormation, or launches EC2/ECS with a role, the deploy role needs `iam:PassRole` on your app's role. A missing PassRole permission is a common cause of `AccessDenied` errors during deployment.
- **Scope it tightly**: allow passing only your app's role (as in the example), never `"Resource": "*"`. Otherwise the pipeline could pass an admin role to anything.

---

### Exam notes

- **Service-linked role** = linked to **one AWS service**, **predefined** by it, used to act on your behalf. You **can't edit** its policies, and you **can't delete** it while the service still needs it.
- Names start with **`AWSServiceRoleFor...`**, path **`/aws-service-role/`**.
- Creating one requires **`iam:CreateServiceLinkedRole`**.
- **`iam:PassRole`** = permission to **give an existing role to a service** (EC2, Lambda, CloudFormation, ...).
- With PassRole you **don't need the role's permissions** yourself, and you **never assume it**: the service does.
- "Let developers launch resources with pre-approved roles without being able to escalate their own privileges" → **`iam:PassRole`** scoped to specific roles.
