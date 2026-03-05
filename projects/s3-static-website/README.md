PROJECT 1 - Amazon S3 Static Website: Security, Versioning & Disaster Recovery
## Overview
Hosted a static website for a fictional café business using Amazon S3, implementing architectural best practices around data protection, cost optimization, and disaster recovery.

This project was built as part of my AWS Solutions Architect Associate (SAA) studies.

---

## Architecture

```
Source Bucket (us-east-1)
├── Static website hosting enabled
├── Public access via bucket policy
├── Versioning enabled
└── Lifecycle rules configured
        │
        │ Cross-Region Replication
        ▼
Destination Bucket (us-west-2)
└── Versioning enabled
```

---

## What I Built

### 1. Static Website Hosting
Created an S3 bucket in `us-east-1` and configured it to host a static website. Uploaded the site's `index.html`, CSS, and image assets.

### 2. Public Access via Bucket Policy
Configured a bucket policy to grant read-only access to anonymous public users. This required disabling Block Public Access settings and enabling ACLs.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

### 3. Versioning
Enabled S3 versioning to protect against accidental overwrites and deletions. Uploaded a modified version of `index.html` and confirmed that both versions were preserved in the bucket.

### 4. Lifecycle Rules
Configured two separate lifecycle rules to manage costs as the bucket grows:
- **Rule 1:** Move noncurrent object versions to S3 Standard-IA after 30 days
- **Rule 2:** Permanently delete noncurrent object versions after 365 days

### 5. Cross-Region Replication (CRR)
Created a destination bucket in `us-west-2` and configured CRR on the source bucket using the `CafeRole` IAM role. Verified that new objects uploaded to the source bucket were automatically replicated to the destination bucket.

---

## Challenges & Lessons Learned

- **403 Forbidden error** — The website returned 403 initially because Block Public Access was still enabled. Had to disable it before the bucket policy could be applied. A common gotcha when setting up public S3 websites.
- **Versioning** — Once enabled, versioning cannot be disabled, only suspended. Important to understand before enabling it in production.
- **Lifecycle rules** — The lab required two separate rules rather than two transitions in one rule. Keeping rules separate makes them easier to manage and audit.
- **Cross-Region Replication** — CRR only replicates new objects after the rule is created. Existing objects are not replicated unless you use S3 Batch Replication separately.

---

## AWS Services Used
- Amazon S3

## SAA Domains Covered
- Design Resilient Architectures (CRR, versioning)
- Design Cost-Optimized Architectures (lifecycle policies)
- Design Secure Architectures (bucket policies, IAM roles)

---

## Screenshots
See the `/screenshots` folder for evidence of each configuration step.

---

## Resources
- [AWS S3 Static Website Hosting Docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
- [S3 Replication Docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 Lifecycle Docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
