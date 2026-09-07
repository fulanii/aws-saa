# AWS Solutions Architect Associate — Study Notes & Labs

My working notebook for the **AWS Certified Solutions Architect – Associate (SAA-C03)** exam, written from the perspective of a backend developer who ships Python **Django / FastAPI** applications.

This is not a copy of the exam guide. Every service here is written in my own words, with the parts that actually matter when you have to deploy a real Python API: what it is, when to reach for it, when it's the wrong tool, and what it costs you operationally.

I'm following [Adrian Cantrill AWS Certified Solutions Architect - Associate (SAA-C03)](https://learn.cantrill.io/p/aws-certified-solutions-architect-associate-saa-c03)


## What's in here

- **Service notes** — one file per major AWS service (EC2, S3, RDS, VPC, IAM, Lambda, ECS, CloudFront, SQS/SNS, Route 53, …).
- **Architecture diagrams** — small ASCII/Mermaid diagrams showing how the pieces fit together.
- **Deployment examples** — real, runnable examples of deploying a Django or FastAPI app onto the service in question.
- **Django/FastAPI mapping** — how each service maps to something I already do in a Python backend (e.g. S3 → `django-storages`, RDS → `DATABASES`, SQS → Celery broker).
- **Exam notes** — gotchas, limits, and the "AWS wants this answer" patterns that show up on SAA-C03.

## Note format

Each service note follows the same structure, so it stays useful as a reference long after the exam:

```
1. What it is          — plain-English definition
2. When to use it      — the problems it actually solves
3. When NOT to use it  — cheaper/simpler alternatives, common misuse
4. Django/FastAPI map  — how it shows up in a Python backend
5. Architecture diagram
6. Deployment example  — console steps, CLI, or IaC
7. Exam notes          — limits, pricing traps, distractor answers
```

## Who this is for

- **Recruiters / hiring managers** — evidence of hands-on AWS work, not just a certificate PDF. The deployment examples are things I actually built and ran.
- **Anyone studying for SAA-C03** — especially if you come from a Python/Django background and want AWS explained in terms you already know.
- **Me, later** — the reference I'll open when I'm designing something at work.

## Table of contents

1. [**AWS Accounts**](aws-accounts/readme.md)
2. [**Cloud, Networking and Technical Fundamentals**](cloud-networking-and-technical-fundamentals/readme.md)
3. [**AWS Fundamentals**](aws-fundamentals/readme.md)
4. [**IAM, Accounts and AWS Organisations**](iam-accounts-and-aws-organisations/readme.md)