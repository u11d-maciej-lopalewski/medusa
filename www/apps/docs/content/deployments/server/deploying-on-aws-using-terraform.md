---
description: 'Step-by-step deployment instruction'
addHowToData: true
---

# Deploy the Medusa Backend using the Terraform Module on AWS

This guide explains how to use the [**Terraform module for Medusa**](https://registry.terraform.io/modules/u11d-com/medusajs/aws) to deploy a production-ready e-commerce backend on AWS. Designed for development teams building Medusa platforms, this module streamlines infrastructure setup while incorporating security best practices. It provides a robust and scalable foundation, enabling teams to quickly deploy a functional backend and then focus on advanced customizations and feature development accelerating development by eliminating the complexities of manual infrastructure provisioning. As a result, teams can rapidly deploy and iterate, significantly minimizing time spent on infrastructure management. This module establishes a reliable groundwork for even the most sophisticated and customized Medusa deployments.


## Introduction

### What is Terraform?
Terraform is an infrastructure-as-code tool that allows you to define and provision cloud resources using code. It simplifies the process of managing cloud infrastructure and ensures consistency across environments. Learn more about Terraform [here](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code).

### Why Use Terraform Module for Medusa?
[Terraform module for Medusa](https://registry.terraform.io/modules/u11d-com/medusajs/aws) simplifies the deployment of Medusa on AWS by automating the setup of essential infrastructure components. It aligns with Medusa’s [architectural modules](#mapping-medusa-architectural-modules-to-aws-infrastructure) for data caching, event distribution, asset and file access, and workflow management. By providing optional infrastructure services, it empowers developers to make informed decisions and optimize the system for production environments. This module is designed for development teams who want to focus on building their e-commerce stores without the overhead of managing cloud infrastructure.

---

## Prerequisites

Before using Terraform module for Medusa, ensure you have the following tools and accounts set up:

### Tools Required
- **Terraform CLI**: Install Terraform by following the [official installation guide](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli).
- **AWS CLI**: Install the AWS CLI by following the [official installation guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
- **Node.js**: Install Node.js from the [official website](https://nodejs.org/).

### AWS Account Setup
- **Create an AWS Account**: If you don’t already have an AWS account, follow the [AWS Getting Started Guide](https://docs.aws.amazon.com/accounts/latest/reference/getting-started.html) to create one.
- **Configure AWS Credentials**: Set up your AWS credentials for Terraform by following the [AWS CLI Quickstart guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html).

---

## Architectural Decisions Behind the Terraform Module
When designing the Terraform module for Medusa, we prioritized scalability, security, and performance while minimizing the operational overhead for development teams. To achieve this, we selected fully managed AWS services, allowing teams to focus on building e-commerce solutions without the complexity of maintaining cloud infrastructure.

Key architectural decisions include:
- Use of Managed Services: All core components — such as Amazon RDS for PostgreSQL, Amazon ElastiCache for Redis, and the Application Load Balancer (ALB) — are fully managed services. This eliminates the need for teams to handle infrastructure maintenance tasks like patching, backups, and scaling operations.
- Containerization with AWS Fargate: We run Medusa services as ECS tasks on AWS Fargate, removing the need for Kubernetes cluster management. Teams can focus on defining scaling policies without worrying about the underlying infrastructure.
- Optimized Docker Images: The Terraform module is complemented by Docker image definitions (provided as Dockerfiles in a separate [repository](https://github.com/u11d-com/medusa-starter/tree/v1)). These images follow best practices for security, performance, and minimal size, ensuring efficient resource utilization in production environments.

By leveraging managed services, this architecture reduces the time, effort, and expertise required to maintain infrastructure. It provides the flexibility to control scalability while ensuring high availability, robust security, and optimal performance — allowing development teams to concentrate on delivering business value through their e-commerce platforms.

## Overview of the Terraform Module

The [Terraform module for Medusa](https://registry.terraform.io/modules/u11d-com/medusajs/aws) sets up the following AWS resources to support Medusa:

### Core Infrastructure
- RDS for PostgreSQL: A managed relational database for storing Medusa data.
- ElastiCache (Redis): A caching layer to improve performance.
- AWS Fargate for ECS Backend Tasks: Runs the Medusa backend as containerized tasks.
- Amazon S3: Cloud object storage that ensures scalability, data availability, security, and performance.
- Application Load Balancer (ALB): Distributes incoming traffic to backend tasks.

### Optional Infrastructure
- Elastic Container Registry (ECR): Stores Docker images for the backend and frontend (if enabled).
- Frontend ECS Tasks: Runs the frontend application (if enabled).
- CloudFront: A CDN that serves the frontend with low latency (if enabled).

### Mapping Medusa Architectural Modules to AWS Infrastructure

| Medusa Architectural Module | AWS Provisioned Infrastructure | Notes |
|----------------------------|-------------------------------|--------|
| Internal Database | Amazon RDS for PostgreSQL | |
| [Cache Modules](https://docs.medusajs.com/v1/development/cache/modules/redis) | Amazon ElastiCache | |
| [Event Modules](https://docs.medusajs.com/v1/development/events/modules/redis) | Amazon ElastiCache | |
| [File Module Providers](https://docs.medusajs.com/v1/plugins/file-service/s3) | Amazon S3 | |
| Workflow Engine Modules | Amazon ElastiCache | |
| Notification Module Providers | Amazon SES | *Planned for future implementation* |

---

## Step-by-Step Guide to Using the Terraform Module

### Step 1: Download or Clone the Module
Clone the Terraform module repository to your local machine:
```bash
git clone https://github.com/u11d-com/terraform-aws-medusajs
cd terraform-u11d-medusajs/examples/minimal
```

### Step 2: Provide Variables for the Module
See module inputs documentation for available configuration options.
[Example configurations](https://registry.terraform.io/modules/u11d-com/medusajs/aws) as a reference.

Example variables:
```hcl
backend_container_image = "your-medusa-backend-image:v1.20.11"
project     = "medusa"
environment = "prod"
owner       = "my-team"
```

### Step 3: Initialize Terraform
Run the following command to initialize Terraform and download the required providers:
```bash
terraform init
```

### Step 4: Plan the Infrastructure
Preview the resources that Terraform will create:
```bash
terraform plan
```

### Step 5: Apply the Infrastructure
Deploy the infrastructure by running:
```bash
terraform apply
```
Confirm the deployment by typing `yes` when prompted.

### Step 6: Verify the Setup
1. Check the Terraform outputs for the Medusa backend URL.
2. Use the URL to verify that the backend is running.


## Connecting Medusa to the Deployed Infrastructure

Once the infrastructure is deployed, the Medusa backend URL will be available in the Terraform outputs. Use this URL to connect your Medusa application to the deployed backend.


### Configuring Medusa with Provisioned AWS Services
In addition to the backend, the Terraform module provisions key AWS resources, including an Amazon S3 bucket for object storage and an Amazon ElastiCache (Redis) instance. To integrate these services with your Medusa application:

1. Configure S3 and Redis Plugins:
- Set up the appropriate Medusa plugins for S3 and Redis to enable object storage and caching functionalities.
  - [Redis Cache Module](https://docs.medusajs.com/v1/development/cache/modules/redis)
  - [File Service - S3 plugin](https://docs.medusajs.com/v1/plugins/file-service/s3)
- The same Redis instance can be utilized for multiple Medusa modules, including cache, event handling, file storage, and the workflow engine.

2. Use Predefined Environment Variables:
- The Terraform module automatically generates all necessary environment variables, following Medusa’s naming conventions.
- These variables simplify the configuration process, ensuring seamless compatibility with Medusa.

All internal connections between services are securely configured with proper access controls, reducing the need for additional security configurations.
By following these steps, you’ll ensure your Medusa application is fully integrated with the cloud infrastructure, optimized for performance, scalability, and security.

## Optional Features

### Enable ECR for Container Images
To create an ECR repository for storing container images, provide the following variable:
```hcl
ecr_backend_create = true
ecr_storefront_create = true
```

See [ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html) to learn how to log in and push images.

### Use an External Image Repository
If you’re using an external image repository, provide the repository address and image tag as variables, optionally you might need to provide credentials for repository:
```hcl
backend_container_image = "your-repo/medusa-backend:v1.20.11"
backend_container_registry_credentials = {
    username = "your-registry-username"
    password = "your-registry-password"
  }
storefront_container_image = "your-repo/medusa-backend:v1.20.11"
storefront_container_registry_credentials = {
    username = "your-registry-username"
    password = "your-registry-password"
  }
```

## Common Issues and Troubleshooting

### Common Errors
- AWS Permission Errors: Ensure your AWS credentials are correctly configured.
- Terraform State Issues: Avoid manually modifying resources created by Terraform.

### Clean Up Resources
To delete all resources created by Terraform, run:
```bash
terraform destroy
```

---

## Next Steps
- Learn how to set up Medusa locally using the [medusa-starter repository](https://github.com/u11d-com/medusa-starter/tree/v1).
- Explore the Medusa starter templates:
  - [Backend Starter Template](https://github.com/medusajs/medusa-starter-default).
  - [Frontend Starter Template](https://github.com/medusajs/nextjs-starter-medusa).


### Deploy the Frontend
Medusa starter kits typically use the Next.js framework to build the storefront web application. Next.js requires that the Medusa backend is available and responsive during the build process to properly fetch data and create a fully functional storefront. This is important because storefront will be built using the API provided by the backend application.

To deploy the frontend, provide the following variables:
```hcl
storefront_create = true
storefront_container_image  = "your-repo/frontend:v1.20.11"
```

For a detailed step-by-step guide on building, pushing, and configuring the storefront container image, refer to the [Storefront Deployment Guide](../storefront/deploying-next-on-aws-using-terraform.md). Following this guide ensures that your storefront is correctly built, stored in AWS ECR, and integrated with the Terraform module.

---

## FAQs

### Can I use this for local development?
No, you can use the [medusa-starter repository](https://github.com/u11d-com/medusa-starter/tree/v1) to set up Medusa locally.

### How do I update the infrastructure?
Modify the Terraform configuration files and run `terraform apply` to apply the changes.

---
:heart: _Technology made with passion by [u11d](https://u11d.com)_
