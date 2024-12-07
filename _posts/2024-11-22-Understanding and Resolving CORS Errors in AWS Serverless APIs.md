---
title: Understanding and Resolving CORS Errors in AWS Serverless APIs
date: 2024-11-22 01:00:00 -0400
categories: "web_dev"
header:
  teaser: "/assets/images/user-auth.svg"
tags:
  - AWS
  - API
toc: true
toc_sticky: true
---

# Introduction
Have you ever tested an API using `curl` or `postman` and thought the API is good to be integrated with your web interface, only to find it breaks in the browser? It certainly happened to me when I was working on a Lambda API. And once again, I found myself encounter the CORS error when using the API from a browser. I previously delved into CORS errors arising from S3 and Cloudfront configuration in one of the articles, however, I'd like to discuss how to resolve such errors in the context of AWS serverless API development. I'll also share my terraform code for managing API infrastructure.

# Technology Stack
The technologies included in this article is as follows:

#### AWS
- Lambda
- API Gateway
#### IaC
- Terraform
#### Python Libraries
- FastAPI
- Magnum


# CORS
![cors-error](/assets/images/cors-error.png)

To put it simply, CORS errors essentially occur due to the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) that restricts cross-origin HTTP requests initiated from scripts. What is a cross-origin request? An example would be your frontend running on `http://localhost:3000` and using `fetch()` to access your backend API hosted at `https://api.example.com`.
Browsers make a `preflight` requests to the server with the cross-origin resource, to check that the server will respond to the actual request. Unless there are appropriate CORS headers in the response from the other origin, the browser will block such requests.

{: .notice--info}
An origin is the combination of protocol (http, https), domain (myapp.com, localhost, localhost.tiangolo.com), and port (80, 443, 8080).


# The Solution: Adding CORS Middleware In Lambda
The fix was fairly simple: I needed to add CORS headers to API responses. Since I was using FastAPI for my Lambda function, I leveraged its built-in `CORSMiddleware`. Here's how to set it up:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], # Allows all origins
    allow_credentials=True,
    allow_methods=["*"], # Allows all HTTP methods
    allow_headers=["*"], # Allows all headers
)
```

This middleware ensures that every response includes the appropriate CORS headers, allowing your browser to securely communicate with the API.


{: .notice--info}
While allow_origins=["*"] is useful during development, consider restricting origins to trusted domains in production for security reasons. For example: allow_origins=["https://myapp.com"].


# Implementing Infrastructure with Terraform
While the article mainly focuses on resolving CORS errors, understanding how to set up the Lambda function and API Gateway is crucial for a serverless API. Below is a detailed Terraform configuration for deploying a Lambda-based API using API Gateway.

### Terraform Configuration

```terraform
provider "aws" {
  region = "us-east-1"
}

# IAM Role for Lambda Execution
resource "aws_iam_role" "lambda_execution_role" {
  name = "lambda-execution-role"
  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      }
    }
  ]
}
EOF
}

# Attach Basic Execution Role for CloudWatch Logging
resource "aws_iam_role_policy_attachment" "basic_execution_role_policy" {
  role       = aws_iam_role.lambda_execution_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# Example: Add Custom IAM Policies as Needed
# Ensure you create policies to allow Lambda access to services like S3, DynamoDB, or others.
# For example:
resource "aws_iam_policy" "custom_policy" {
  name = "CustomPolicy"

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = [
          "s3:ListBucket",
          "s3:GetObject",
          "dynamodb:Query"
        ],
        Effect   = "Allow",
        Resource = "*"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "custom_policy_attachment" {
  role       = aws_iam_role.lambda_execution_role.name
  policy_arn = aws_iam_policy.custom_policy.arn
}

# Security Group for Lambda
resource "aws_security_group" "lambda_security_group" {
  name   = "lambda-security-group"
  vpc_id = var.vpc_id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Lambda Function
resource "aws_lambda_function" "api_lambda" {
  function_name = "my-api-lambda"
  role          = aws_iam_role.lambda_execution_role.arn
  package_type  = "Image"
  image_uri     = "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-lambda:latest"
  timeout       = 120
  memory_size   = 2048

  vpc_config {
    security_group_ids = [aws_security_group.lambda_security_group.id]
    subnet_ids         = [var.private_subnet_id]
  }
}

# API Gateway Setup
resource "aws_api_gateway_rest_api" "api_gateway" {
  name        = "my-api"
  description = "API Gateway for Lambda function"
}

resource "aws_api_gateway_resource" "proxy" {
  rest_api_id = aws_api_gateway_rest_api.api_gateway.id
  parent_id   = aws_api_gateway_rest_api.api_gateway.root_resource_id
  path_part   = "{proxy+}"
}

resource "aws_api_gateway_method" "proxy_method" {
  rest_api_id   = aws_api_gateway_rest_api.api_gateway.id
  resource_id   = aws_api_gateway_resource.proxy.id
  http_method   = "ANY"
  authorization = "NONE"
}

resource "aws_api_gateway_integration" "proxy_integration" {
  rest_api_id             = aws_api_gateway_rest_api.api_gateway.id
  resource_id             = aws_api_gateway_resource.proxy.id
  http_method             = aws_api_gateway_method.proxy_method.http_method
  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = aws_lambda_function.api_lambda.invoke_arn
}

resource "aws_api_gateway_deployment" "api_gateway_deployment" {
  rest_api_id = aws_api_gateway_rest_api.api_gateway.id

  depends_on = [
    aws_api_gateway_integration.proxy_integration
  ]
}

# Lambda Permission for API Gateway
resource "aws_lambda_permission" "lambda_permission" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.api_lambda.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_api_gateway_rest_api.api_gateway.execution_arn}/*/*"
}

output "api_endpoint" {
  value = aws_api_gateway_rest_api.api_gateway.execution_arn
}
```
The Lambda function's permissions should be updated according to your API requirements (e.g., access to DynamoDB, S3, or other services). This is demonstrated with a custom policy in the example.

##### reference:
- <https://fastapi.tiangolo.com/tutorial/cors/#use-corsmiddleware>