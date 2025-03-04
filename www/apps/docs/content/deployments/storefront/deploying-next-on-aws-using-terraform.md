---
description: 'Step-by-step deployment instruction'
addHowToData: true
---

# Build and Deploy the Medusa Next.js Storefront on AWS

This guide explains how to build the Medusa Storefront using the provided starter template and deploy the resulting Docker container image to an AWS Elastic Container Registry (ECR). This process is essential for deploying the frontend as part of the Medusa e-commerce solution.

## Prerequisites

Before proceeding, ensure you have the following:
- **Docker**: Installed and running on your machine. Download Docker [here](https://docs.docker.com/get-started/get-docker/).
- **AWS CLI**: Installed and configured with your AWS credentials. Follow the [AWS CLI Quickstart guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) if you haven’t set it up yet.
- **Git**: Installed on your machine. Download Git [here](https://git-scm.com/).
- **Terraform**: (Optional but recommended) Installed for deploying infrastructure. Download Terraform [here](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli).

---

## Setting Up the Storefront Repository

1. Clone the [`medusa-starter`](https://github.com/u11d-com/medusa-starter/tree/v1) repository to your local machine:
   ```bash
   git clone https://github.com/u11d-com/medusa-starter.git
   cd medusa-starter
   git checkout -b v1 origin/v1
   ```

2. Navigate to the `storefront` directory and initialize the repository, this sets up the storefront based on the [Medusa Next.js starter template](https://github.com/medusajs/nextjs-starter-medusa) for version 1:
2. Navigate to the `storefront` directory and initialize the repository, this sets up the storefront based on the [Medusa Next.js starter template](https://github.com/medusajs/nextjs-starter-medusa) for version 1:
   ```bash
   cd storefront
   git init
   git remote add origin https://github.com/medusajs/nextjs-starter-medusa.git
   git fetch --depth 1 origin 0f5452dfe44838f890b798789d75db1a81303b7a
   git checkout 0f5452dfe44838f890b798789d75db1a81303b7a
   ```


---

## Building the Storefront Docker Container Image

1. Build the Docker container image for the storefront:
   ```bash
   docker build --build-arg MEDUSA_BACKEND_URL="https://my.medusa.backend" -t medusa_storefront:1.0.0 .
   ```
   - Replace `medusa_storefront:1.0.0` with your preferred image tag in the format `<image_tag>:<version>`, e.g., `my-storefront:1.0.0`.
   - Replace `https://my.medusa.backend` with your existing Medusa backend URL address.

---

## Pushing the Container Image to AWS Elastic Container Registry (ECR)

To deploy the storefront, you’ll need to push the Docker container image to an AWS ECR repository. Follow these steps:

### Step 1: Authenticate Docker to Your ECR Repository
Run the following command to authenticate Docker to your AWS ECR registry:
```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```
Replace:
- `<region>` with your desired AWS region code (for example, `eu-west-1`). You can find the complete list of region codes in the [AWS Regions documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-regions). Make sure to use the same region where your Amazon Elastic Container Registry (ECR) is deployed.
- `<aws_account_id>` with your [AWS account ID](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-identifiers.html#FindAccountId).
  You can check it by running:
  ```bash
  aws sts get-caller-identity \
      --query Account \
      --output text
  ```
If you created your ECR repository using the [Terraform Module for Medusa on AWS](https://registry.terraform.io/modules/u11d-com/medusajs/aws), you can find the ECR repository URL in the module's output values after deployment.


### Step 2: Tag the Docker Container Image
Tag the Docker container image with your ECR repository URI:
```bash
docker tag medusa_storefront:1.0.0 <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<repository>:<tag>
```
Replace:
- `medusa_storefront:1.0.0` with the image tag you created earlier.
- `<aws_account_id>`, `<region>`, `<repository>`, and `<tag>` with your AWS account ID, region, ECR repository name, and desired image tag, respectively.

### Step 3: Push the Docker Image
Push the Docker container image to your ECR repository:
```bash
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<repository>:<tag>
```

Example:
```bash
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-storefront:1.0.0
```

For more details, refer to the [AWS ECR documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html).

---

## Connecting the Storefront to Your Terraform Infrastructure

After pushing the Docker image, connect it to your infrastructure deployed via Terraform:
1. Reference the ECR Image in Terraform:
Add the ECR repository URI and tag to your Terraform configuration:
```hcl
storefront_container_image = "<aws_account_id>.dkr.ecr.<region>.amazonaws.com/<repository>:<tag>"
```

2. Enable storefront deployment:
Ensure the storefront is enabled in your Terraform module configuration:
```hcl
storefront_create = true
```

3. Plan and apply Terraform changes:
Deploy the updated configuration:
```bash
terraform init
terraform plan
terraform apply
```
Confirm the changes when prompted.

---

## Troubleshooting

### Common Issues
- **Docker Build Fails**:
  - Ensure all dependencies are installed.
  - Verify the `Dockerfile` is correctly configured.
  - Check for syntax errors or missing files.
- **ECR Authentication Fails**:
  - Confirm that your AWS credentials are correctly configured with sufficient permissions.
  - Ensure your AWS CLI session is active (re-authenticate if necessary).
- **Image Push Fails**:
  - Ensure the ECR repository exists.
  - Ensure the repository URI and image tag are correctly specified.
  - Check for network connectivity issues.
- **Terraform Apply Fails**:
  - Confirm that the referenced Docker image exists in the ECR repository.
  - Check Terraform logs for specific error messages.

## Additional Resources
- [Medusa Documentation](https://docs.medusajs.com/v1/)
- [Next.js Documentation](https://nextjs.org/)
- [Terraform Module for Medusa on AWS Provider Documentation](https://registry.terraform.io/modules/u11d-com/medusajs/aws)

---
:heart: _Technology made with passion by [u11d](https://u11d.com)_