# CloudFoxable - Challenge 1: Do This First!

## Overview
CloudFoxable is a Terraform-based AWS security challenge. The first challenge, **"Do This First!"**, is designed to help participants validate their setup and retrieve their first flag.

## Setup Instructions
### Steps to Deploy CloudFoxable
1. Select or create an AWS account (avoid using production accounts).
2. Create a **non-root user** with administrative access.
3. Generate an **access key** for this user.
4. Install and configure **AWS CLI**:
   ```sh
   aws configure
   aws sts get-caller-identity
   ```
5. Install **Terraform**, clone the repository, and navigate to the directory:
   ```sh
   git clone https://github.com/BishopFox/cloudfoxable
   cd cloudfoxable/aws
   ```
6. Configure Terraform:
   ```sh
   cp terraform.tfvars.example terraform.tfvars
   ```
   Edit `terraform.tfvars` to set your AWS profile:
   ```hcl
   aws_local_profile = "YOUR_PROFILE"
   ```
7. Initialize and deploy CloudFoxable:
   ```sh
   terraform init
   terraform apply
   ```

## Finding the First Flag
1. Carefully read the **Terraform output** after deployment.
2. Locate the credentials for `ctf-starting-user`.
3. Configure AWS CLI for the challenge user:
   ```sh
   aws configure --profile ctf-starting-user
   ```
4. Verify access:
   ```sh
   aws sts get-caller-identity --profile ctf-starting-user
   ```
5. Find the flag in the Terraform output (`FLAG{}` format) and submit it.

**Output:**
```
FLAG{congrats_you_are_now_a_terraform_expert_happy_hunting}
```

## Analysis
- Ensures correct AWS environment setup.
- Introduces AWS Identity and Access Management (IAM).
- Highlights the importance of securely handling credentials.
- Demonstrates how Terraform can provision cloud resources.
- Provides a foundation for more advanced security challenges.

## Conclusion
The "Do This First!" challenge verifies that participants can successfully configure and interact with AWS resources. Completing this step ensures a smooth experience for future challenges. To remove all deployed resources, run:
```sh
terraform destroy
```

---

