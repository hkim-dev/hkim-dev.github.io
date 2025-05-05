---
title: Automating Static UI Deployment with AWS + CloudFormation + CircleCI
date: 2025-05-04 23:00:00 -0400
categories: "web_dev"
tags:
  - AWS
toc: true
toc_sticky: true
classes: wide
---

# Introduction
In this post, I'm sharing a template project for automating the build & deployment of single page applications (SPA). I often build and deploy SPAs at work, and after doing this a few times manually, I figured it was time to put together a clean & reusable setup. Feel free to check out the repo here for the full code:

👉 https://github.com/hkim-dev/infra-static-ui-deploy

So this small project aims to:

- define the AWS infrastructure as code using CloudFormation,
- automate deployment with CircleCI,
- and document the process so I (or anyone else) can reuse it later.


# 💻 Tech Stack

- React (Vite) for the sample static UI
- AWS S3 + CloudFront for hosting
- Route53 for DNS
- ACM for HTTPS
- CloudFormation to define infrastructure as code
- CircleCI for CI/CD automation

## IaC with CloudFormation

Instead of manually managing AWS resources in the console or using the AWS CLI, I used AWS CloudFormation to automate everything. It allows engineers to define AWS resources such as S3, CloudFront, and Route53 in the form of code. It might take more time up front, but, once the stack is defined as code, it is really easy to spin up the stack in different environments (`dev`, `prod`, etc.) by just tweaking parameters. 

The following is a snippet from the CloudFormation template that defines S3 and CloudFront:

```yaml
Resources:
  FrontendBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub ${ProjectPrefix}-s3-${Env}

  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
          - DomainName: !Join ['', [!Ref FrontendBucket, '.s3.amazonaws.com']]
            S3OriginConfig:
              OriginAccessIdentity: ...
```

Here's a quick look at the overall AWS setup:

![aws-setup](/assets/images/static-ui-aws-setup.png)

## Continuous Deployment with CircleCI
CircleCI handles the build and deployment pipeline from start to finish:
1. Build: It checks out the code, installs dependencies, and builds the static UI.
2. Infrastructure Provisioning: The AWS resources are created automatically with the CloudFormation template.
3. Deployment: The built static files (e.g., `/dist`) are synced to the S3 bucket, and CloudFront is used to serve the content globally.

You can manage environment-specific settings using [CircleCI contexts](https://circleci.com/docs/contexts/), and secure credentials via environment variables.

## Mock UI
I created a mock React app with Vite for demonstration purposes. You can swap it out with your own static frontend by replacing the /sample-ui directory.

# 📝 Wrapping Up
This project was something I actually built a while ago at work while deploying SPAs on AWS - revisiting it reminded me of how important it is to have a clean and consistent pipeline for UI deployments. I hope it can be a useful starting point for anyone looking to automate static UI hosting.