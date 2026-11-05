# **CloudFormation Basics**

[← Back to AWS Fundamentals](readme.md)

> Infrastructure as Code (IaC) on AWS: describe your infrastructure in a template, and CloudFormation creates, updates and deletes it for you.

### Contents

- [What it is](#what-it-is)
- [Template anatomy](#template-anatomy)
- [Example: an EC2 instance](#example-an-ec2-instance)
- [Stacks, logical and physical resources](#stacks-logical-and-physical-resources)
- [Django/FastAPI map](#djangofastapi-map)
- [Exam notes](#exam-notes)

---

### What it is

CloudFormation lets you **create, update and delete AWS infrastructure from templates** instead of clicking through the console by hand. Templates are written in **YAML or JSON**.

It's **declarative**: you describe *what* you want to exist, and CloudFormation works out *how* to get there.

- **Add** a resource to the template → CloudFormation **creates** it.
- **Change** a resource in the template → CloudFormation **updates** it.
- **Remove** a resource from the template → CloudFormation **deletes** it.

---

### Template anatomy

| Section | Required? | What it does |
|---|---|---|
| **Resources** | **Yes, the only mandatory section** | The list of things to create (EC2 instances, buckets, VPCs, ...). |
| **AWSTemplateFormatVersion** | No | The template format version, which lets AWS extend the format over time. The only valid value is `"2010-09-09"`. If omitted, it's assumed. |
| **Description** | No | A free-text description of the template. **If both are present, `Description` must directly follow `AWSTemplateFormatVersion`.** |
| **Metadata** | No | Controls how the template is shown in the console UI, e.g. grouping and ordering parameters (`AWS::CloudFormation::Interface`). |
| **Parameters** | No | Inputs the user is **prompted for** when creating or updating the stack (e.g. instance type, environment name), with optional defaults and allowed values. |
| **Mappings** | No | **Lookup tables** of keys and values, e.g. which AMI to use in which region. Read with `!FindInMap`. |
| **Conditions** | No | **Decision making**: a resource is only created when its condition is true, e.g. only create a large instance if `EnvType` is `prod`. |
| **Outputs** | No | Values **shown once the stack finishes**, e.g. an instance ID or a website URL. They can also be exported for other stacks to use. |

---

### Example: an EC2 instance

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A single EC2 instance          # must directly follow the version

Parameters:
  InstanceType:                             # user is prompted for this
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small]
  LatestAmiId:                              # latest Amazon Linux AMI for this region
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

Resources:
  Instance:                                 # logical resource name
    Type: AWS::EC2::Instance                # what to create
    Properties:                             # how to configure it
      InstanceType: !Ref InstanceType
      ImageId: !Ref LatestAmiId

Outputs:
  InstanceId:
    Description: ID of the new instance
    Value: !Ref Instance
```

- `Instance` is a **logical resource**: a name used inside the template.
- Every logical resource has a **Type** (e.g. `AWS::EC2::Instance`), which tells CloudFormation what to create.
- Most logical resources have **Properties**, which CloudFormation uses to configure the resource.
- `!Ref` references a parameter or another resource in the same template.

---

### Stacks, logical and physical resources

```
Template ──create──> Stack ──────────────> Your AWS account
(YAML/JSON)          (logical resources)    (physical resources)
                      Instance       ───>    i-0abc123... (a real EC2 instance)
```

- CloudFormation takes a template and creates a **stack**. The stack contains every **logical resource** the template describes.
- For each logical resource, CloudFormation creates a matching **physical resource** in your AWS account, and keeps the two in sync.
- **Update the template** and apply it to the stack → the physical resources change to match.
- **Delete the stack** → its physical resources are deleted too.
- The same template can create many stacks, e.g. one for `dev` and one for `prod`, or one per region.

---

### Django/FastAPI map

- **Template**: like Django migrations, but for infrastructure. You change a file in Git, apply it, and the real environment changes to match, instead of hand-editing things in production.
- **Stack**: one deployed environment of your app (VPC, EC2, RDS, S3 bucket), so you can spin up an identical `staging` with one command and delete it just as easily.
- **Parameters / Outputs**: like environment variables in and settings out: pass `InstanceType` or `EnvType` in, get the database endpoint or bucket name out for your `settings.py` or `.env`.
- Similar tools you may have seen: **Terraform**, **AWS CDK** (writes CloudFormation for you, in Python).

---

### Exam notes

- **Resources** is the **only mandatory** section.
- If `Description` and `AWSTemplateFormatVersion` are both used, **`Description` must directly follow `AWSTemplateFormatVersion`**.
- **Parameters** = user input; **Mappings** = lookup tables; **Conditions** = create only if true; **Outputs** = values shown after completion.
- **Logical resource** (in the template/stack) → **physical resource** (real thing in the account).
- Deleting a stack deletes its physical resources.
- "Automate, repeatable, consistent infrastructure deployments across accounts/regions" usually points to **CloudFormation**.
