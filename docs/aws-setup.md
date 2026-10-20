# AWS setup

How the hosting for danielmckinlay.com is built. Follow in order.

## 0. Account safety

1. Enable MFA on the root user. Stop using root after this.
2. Create an admin user in IAM Identity Center. Sign in as that user.
3. Billing > Budgets: create a $5 monthly budget with an email alert.

## 1. S3 bucket

- Name: `danielmckinlay.com-site` (any unique name works)
- Region: your choice, e.g. `us-west-2`
- **Block all public access: ON.** CloudFront reads the bucket. The public never does.
- Optional: enable versioning, with a lifecycle rule that expires noncurrent versions after 30 days.

## 2. TLS certificate (ACM)

- Region: **us-east-1** (required for CloudFront)
- Domain names: `danielmckinlay.com` and `www.danielmckinlay.com`
- Validation: DNS. Create the records in Route 53 (step 4) when prompted.

## 3. CloudFront distribution

- Origin: the S3 bucket (REST endpoint, not the website endpoint)
- Origin access: **Origin Access Control (OAC)**. Apply the bucket policy CloudFront generates.
- Viewer protocol policy: Redirect HTTP to HTTPS
- Alternate domain names: `danielmckinlay.com`, `www.danielmckinlay.com`
- Certificate: the ACM cert from step 2
- Default root object: `index.html`
- Custom error response: 403 and 404 → `/404.html`, response code 404
- Pricing: the Free flat-rate plan

## 4. DNS (Route 53 + Namecheap)

1. Route 53: create a public hosted zone for `danielmckinlay.com`.
2. Copy its four NS records.
3. Namecheap: Domain List > Manage > Nameservers > **Custom DNS**. Paste the four nameservers.
4. Route 53: create `A` and `AAAA` alias records for the apex and `www`, targeting the CloudFront distribution.

Nameserver changes can take a few hours to spread.

## 5. GitHub OIDC deploy role

1. IAM > Identity providers > Add provider
   - Type: OpenID Connect
   - URL: `https://token.actions.githubusercontent.com`
   - Audience: `sts.amazonaws.com`
2. IAM > Roles > Create role > Web identity, using that provider.
3. Trust policy (lock it to this repo's `production` environment):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:akraetik/danielmckinlay.com:environment:production"
        }
      }
    }
  ]
}
```

4. Permissions policy (least privilege):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::<BUCKET_NAME>"
    },
    {
      "Sid": "WriteObjects",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::<BUCKET_NAME>/*"
    },
    {
      "Sid": "InvalidateCache",
      "Effect": "Allow",
      "Action": "cloudfront:CreateInvalidation",
      "Resource": "arn:aws:cloudfront::<ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
    }
  ]
}
```

## 6. Connect GitHub

1. Repo > Settings > Environments > New environment: `production`.
2. Repo > Settings > Secrets and variables > Actions > **Variables** tab. Add:

| Name | Value |
|---|---|
| `AWS_ROLE_ARN` | ARN of the role from step 5 |
| `AWS_REGION` | Bucket region, e.g. `us-west-2` |
| `S3_BUCKET` | Bucket name |
| `CLOUDFRONT_DISTRIBUTION_ID` | Distribution ID |

3. Actions tab > Deploy site > Run workflow.

## Verify

- `https://danielmckinlay.com` loads with a valid certificate.
- `http://` redirects to `https://`.
- `https://danielmckinlay.com/does-not-exist` shows the 404 page.
- The bucket URL itself returns Access Denied.
