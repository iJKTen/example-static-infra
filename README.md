# Static Site Infrastructure

CloudFormation template that deploys a plain HTML/CSS/JS static site to AWS with S3 hosting, CloudFront CDN, CI/CD pipeline, Route 53 DNS, and security headers.

## Prerequisites

- **Connection stack** deployed exporting `Example-GitHubConnectionArn` (series post 2)
- **Certificate stack** deployed in `us-east-1` exporting `Example-CertificateArn` (series post 3)
- **Git Sync role** from series post 4 with the permission additions from posts 7 through 9

## Parameters

| Parameter | Description |
|---|---|
| `DomainName` | Fully qualified domain name (e.g. `example.net`) |
| `GitHubRepository` | GitHub repository in `owner/repo` format |
| `HostedZoneId` | Route 53 hosted zone ID for the domain |

## What Gets Created

- S3 bucket for site files (private, versioned, 7-day noncurrent cleanup)
- CloudFront distribution with OAC, HTTP/2+3, TLS 1.2, and Brotli compression
- S3 bucket policy scoped to the specific CloudFront distribution
- Route 53 A and AAAA alias records pointing the domain to CloudFront
- S3 artifact bucket for pipeline intermediates
- CodePipeline with Source, Build, and Deploy stages
- CodeBuild build project (Node.js 20, `npm run build`)
- CodeBuild deploy project with S3 sync and CloudFront invalidation
- IAM roles for CodePipeline, CodeBuild, and the deploy project
- CloudFront response headers policy (HSTS, CSP, X-Frame-Options, X-Content-Type-Options)

## Deploy

The repo includes two files. `infra.yaml` is the CloudFormation template. `deployment-file.yaml` is the deployment file that Sync from Git reads to find the template and resolve parameter values.

1. Update the placeholder values in `deployment-file.yaml` with your domain, repository, and hosted zone ID.
2. Push the repo to GitHub.
3. Follow the Sync from Git process in series post 16 to create the stack.

## Architecture

Uses two-tier caching. Non-HTML files get `max-age=31536000,public,immutable` and HTML files get `max-age=0,must-revalidate` so browsers always revalidate pages. CloudFront invalidation after each deploy forces edge caches to fetch the latest versions of all files.
