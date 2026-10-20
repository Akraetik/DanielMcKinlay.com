# danielmckinlay.com

[![Deploy site](https://github.com/akraetik/danielmckinlay.com/actions/workflows/deploy.yml/badge.svg)](https://github.com/akraetik/danielmckinlay.com/actions/workflows/deploy.yml)

Source code for my personal website, [danielmckinlay.com](https://danielmckinlay.com).

## Status

🚧 Under construction. A placeholder page is live while the full site is designed.

## Stack

- **Site:** static HTML and CSS
- **Version control:** Git and GitHub
- **CI/CD:** GitHub Actions
- **Hosting:** Amazon S3 (private) behind Amazon CloudFront
- **DNS:** Amazon Route 53, domain registered with Namecheap
- **TLS:** AWS Certificate Manager
- **Auth for deploys:** GitHub OIDC → IAM role (no stored AWS keys)

## How it deploys

A push to `main` that changes `site/` triggers the [deploy workflow](.github/workflows/deploy.yml):

1. Validate that required files exist.
2. Assume a least-privilege IAM role through OIDC.
3. Sync `site/` to S3.
4. Invalidate the CloudFront cache.

See [docs/architecture.md](docs/architecture.md) for the diagram and design choices, and [docs/aws-setup.md](docs/aws-setup.md) for the full build steps.

## Project structure

```
.
├── .github/workflows/
│   └── deploy.yml   # CI/CD pipeline
├── docs/
│   ├── architecture.md
│   └── aws-setup.md
├── site/            # Everything deployed to the web
│   ├── index.html
│   ├── 404.html
│   ├── favicon.svg
│   ├── robots.txt
│   ├── sitemap.xml
│   └── css/
│       └── style.css
├── .editorconfig
├── .gitignore
├── LICENSE
└── README.md
```

## Run locally

No build step. Serve the folder:

```bash
cd site
python3 -m http.server 8000
```

Then visit http://localhost:8000.

## Roadmap

- [x] Repo and placeholder page
- [x] CI/CD workflow with OIDC
- [ ] AWS hosting live on danielmckinlay.com
- [ ] Full site design (Claude Design → Figma → code)
- [ ] Infrastructure as code (Terraform or AWS CDK)

## License

Code is MIT licensed. See [LICENSE](LICENSE). Written content and images are © Daniel McKinlay.
