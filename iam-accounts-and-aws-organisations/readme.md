# **IAM, Accounts and AWS Organisations**

[← Back to main readme](../readme.md)

> Who can do what in AWS: identities, the policies that give them permissions, and how to run many AWS accounts together. One file per topic.

### Contents

| # | Notes | What's inside |
|---|---|---|
| 1 | [**IAM Identity Policies**](iam-identity-policies.md) | Policy JSON structure, statement parts, explicit deny vs allow vs implicit deny, inline vs managed |
| 2 | [**IAM Users and ARNs**](iam-users-and-arns.md) | Principals, authentication vs authorization, access keys, ARN format, user and group limits |
| 3 | [**IAM Groups**](iam-groups.md) | Groups as containers for users, how permissions combine, what groups can't do, limits |
| 4 | [**IAM Roles**](iam-roles.md) | Users vs roles, trust vs permissions policies, sts:AssumeRole and temporary credentials |
| 5 | [**When to Use IAM Roles**](when-to-use-iam-roles.md) | Service roles, break-glass access, SSO/federation, web identity federation, cross-account access |
| 6 | [**Service-linked Roles & PassRole**](service-linked-roles-and-passrole.md) | Roles predefined by AWS services, creating them, passing roles to services with iam:PassRole |
| 7 | [**AWS Organizations**](aws-organizations.md) | Management vs member accounts, inviting and creating accounts, OUs, consolidated billing, role-based access |

Each file has its own contents list at the top, plus a **Django/FastAPI map** and **Exam notes** at the bottom.
