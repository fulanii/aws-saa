# **AWS Organizations**

[← Back to IAM, Accounts and AWS Organisations](readme.md)

> A free service that lets a business manage many AWS accounts together, in a cost-effective way with little to no overhead: one bill, one structure, and one place to control them all.

### Contents

- [What it is](#what-it-is)
- [Management vs member accounts](#management-vs-member-accounts)
- [Adding accounts](#adding-accounts)
- [Structure: root, OUs and accounts](#structure-root-ous-and-accounts)
- [Consolidated billing](#consolidated-billing)
- [Accessing member accounts](#accessing-member-accounts)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

Large businesses rarely run everything in a single AWS account. They have separate accounts for **dev**, **staging** and **prod**, for different teams, or after an acquisition. Managing each one alone (separate bills, separate logins, separate rules) doesn't scale.

**AWS Organizations** groups those accounts into one **organization**:

- **One bill** for every account (consolidated billing).
- **A hierarchy** of accounts grouped into organizational units (OUs).
- **Central control** over what accounts can do, with Service Control Policies (SCPs, covered in the next lesson).
- **No extra cost**: Organizations itself is free.

---

### Management vs member accounts

| | Management account | Member account |
|---|---|---|
| **Also called** | Master account, **payer** account | |
| **How many** | **Exactly one** per organization | As many as you need |
| **What it is** | The account that **created** the organization | Any other account that joined or was created in it |
| **Pays** | **The bill for the whole organization** | Nothing directly. Its costs go to the management account |

- An account can only belong to **one organization at a time**.
- The management account can't be changed to a different account. To change it you'd have to create a new organization.
- **Best practice**: keep **no workloads or resources** in the management account. Use it only for billing and organization management, because it has full control over everything else.

---

### Adding accounts

There are two ways an account becomes a member:

1. **Invite an existing account**: the management account sends an invitation, and the existing account has to **accept** it. After accepting, it becomes a **member account**.
2. **Create a new account directly in the organization**: no invitation needed. You only provide a **unique email address** and a name. The account is a member from the start.

---

### Structure: root, OUs and accounts

```
 Organization root  (top container, NOT the root user)
 ├── Management account
 ├── OU: Development
 │    ├── dev-account
 │    └── staging-account
 └── OU: Production
      ├── prod-account
      └── OU: Payments      ← OUs can be nested
           └── payments-prod-account
```

- **Organization root**: the container at the top of the hierarchy. It holds everything. Don't confuse it with the **account root user**.
- **Organizational Units (OUs)**: folders that group accounts (by environment, team, business unit, ...). OUs can contain accounts **and other OUs**.
- Policies like SCPs attached to an OU apply to **every account below it**.

---

### Consolidated billing

- Every member account's **payment method is removed**. All charges roll up to the **management (payer) account**, which receives **one monthly bill**.
- The bill still shows the breakdown **per account**.
- **Volume pricing is shared**: services with tiered pricing (e.g. S3 cost per GB getting cheaper as usage grows) add up usage across **all** accounts, so the organization reaches cheaper tiers sooner.
- **Reservations and Savings Plans are shared** across accounts by default, so unused capacity bought by one account can discount another.

---

### Accessing member accounts

Organizations doesn't give you one login for everything. You **use IAM roles** to move between accounts:

- Accounts **created** inside the organization automatically get a role called **`OrganizationAccountAccessRole`**, which the management account can assume for full admin access.
- Accounts that **joined by invitation** don't have it. You create it yourself in that account (same name, trusting the management account).
- A common pattern: users log in to **one** account (often a dedicated login or identity account), then **assume roles** into the other accounts. The console's **Switch Role** feature does exactly this. See [IAM Roles](iam-roles.md).

---

### Django/FastAPI map

- **Separate accounts per environment**: put your Django/FastAPI `dev`, `staging` and `prod` in different accounts under one organization. A bad command in `dev` can't touch production data, and each environment's costs show up separately on the one bill.
- **Like separate projects with shared billing**: similar to having separate Heroku apps or GCP projects for each environment, all billed to one company card.
- **Deploying across accounts**: your CI/CD pipeline assumes a role in the target account (e.g. `deploy-role` in `prod`), instead of storing keys for each account.

---

### Exam notes

- **Management account** = **master** = **payer** account. **Only one** per organization, and it's the account that created it.
- Every other account is a **member account**. An account can be in **only one** organization.
- Existing accounts must **accept an invitation**. Accounts can also be **created directly** in the organization.
- **Consolidated billing**: **one bill**, member payment methods removed, **volume discounts and reservations shared** across accounts.
- Structure: **organization root** → **OUs** (can be nested) → **accounts**. The organization root is **not** the root user.
- Accounts created in the org get **`OrganizationAccountAccessRole`**. Invited accounts need it **created manually**.
- Best practice: **no resources in the management account**. Users log in once and **assume roles** into other accounts.
- "Many accounts, one bill, centrally managed, minimal overhead" → **AWS Organizations**.
