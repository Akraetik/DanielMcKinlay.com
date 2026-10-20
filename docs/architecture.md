# Architecture

```mermaid
flowchart LR
  dev[Local edits] -->|git push| gh[GitHub repo]
  gh -->|push to main| ga[GitHub Actions]
  ga -->|OIDC token| iam[IAM role]
  iam -->|temporary credentials| ga
  ga -->|aws s3 sync| s3[(S3 bucket, private)]
  ga -->|invalidate| cf[CloudFront]
  cf -->|Origin Access Control| s3
  user[Visitor] -->|HTTPS| r53[Route 53 DNS]
  r53 --> cf
  acm[ACM certificate] -.-> cf
  nc[Namecheap registrar] -.->|nameservers| r53
```

## Design choices

- **Private bucket + OAC.** The bucket is never public. Only CloudFront can read it.
- **OIDC, not access keys.** GitHub Actions gets short-lived credentials per run. Nothing long-lived is stored in GitHub.
- **Least-privilege role.** The deploy role can write to one bucket and invalidate one distribution. Nothing else.
- **Cache split.** HTML caches for 5 minutes so updates show fast. CSS and images cache for a day.
- **Git is the source of truth.** Files in S3 are never edited by hand. Every change goes through a commit.
