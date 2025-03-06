# Bastion Host Challenge Overview

This challenge focuses on setting up and utilizing a **Bastion Host** for secure access to other resources in the **"Assumed Breach: Application Compromise/Network Access"** category.

## Objectives:

1. **Enable Bastion Host**:  
   Modify the `terraform.tfvars` file to enable the Bastion Host and apply the configuration using Terraform.

2. **Access EC2 via SSM**:  
   Use **AWS Systems Manager (SSM)** for secure access to the deployed EC2 instance.

3. **Discover IAM Permissions**:  
   Utilize **Cloudfox** to identify the IAM permissions associated with the instance.

4. **Find the Flag**:  
   Access an S3 bucket, locate the `flag.txt` file, and retrieve the flag.

5. **Cleanup**:  
   After completing the challenge, disable the Bastion Host by editing the `terraform.tfvars` file and running `terraform apply`.

# Solution

### Enable Bastion Host
- Edit the `cloudfoxable/aws/terraform.tfvars` file and set:
  ```hcl
  bastion_enabled = true

## Apply Terraform Configuration

- Run the following command to apply the configuration:

  ```bash
  terraform apply

### Resolve IP Fetching Issue

- If an error occurs due to an invalid JSON format, it may be caused by the USER_IP variable not fetching the correct IP. Retrieve your public IP manually using:

  ```bash
  curl ifconfig.me

### Hardcode the IP and Rerun Terraform Apply

- Hardcode the retrieved IP into the `terraform.tfvars` file and rerun the following command:

  ```bash
  terraform apply

### Install Cloudfox on Ubuntu

- You can install Cloudfox on Ubuntu by following the instructions in the [Cloudfox Installation Guide](https://cloudfox.dev/docs/install).

### Find Instance ID

- List the EC2 instances to find the instance ID of your bastion host:

  ```bash
  cloudfox aws -p cloudfoxable instances -v2
### Connect to the Instance Using SSM

- Install the Session Manager Plugin (if not already installed) and use the command from the loot file to connect via AWS SSM:

  ```bash
  aws ssm start-session --target <instance-id>
### Retrieve the Flag

- List the contents of the S3 bucket and retrieve the flag:

  ```bash
  aws s3 ls s3://<bucket-name>
  aws s3 cp s3://<bucket-name>/flag.txt -

# Reflection Questions and Answers

### 1. **What did you learn about securing access to EC2 instances using a Bastion Host?**
The Bastion Host serves as a secure gateway to EC2 instances in private subnets, reducing the need for direct SSH access. By using AWS Systems Manager (SSM), I was able to securely connect to instances without needing open SSH ports, enhancing security and control.

---

### 2. **How does AWS Systems Manager (SSM) improve security compared to traditional SSH access?**
SSM eliminates the need for SSH access, reducing the risk of external attacks. It uses IAM roles for authentication and authorization, ensuring only authorized users can access instances. SSM also provides secure, encrypted communication without exposing open ports.

---

### 3. **How does hardcoding the IP address resolve issues with dynamic IP fetching?**
The dynamic IP fetching using `USER_IP` caused errors by returning HTML instead of an IP. By manually retrieving and hardcoding the correct IP into the `terraform.tfvars` file, I bypassed this issue and allowed Terraform to apply the configuration successfully.

---

### 4. **How do you determine which IAM permissions are associated with an instance?**
I used Cloudfox to inspect the instance’s IAM role and determine its permissions, such as access to S3. This is critical for ensuring the instance has the correct permissions to interact with other AWS resources, like retrieving the flag from S3.

---

### 5. **How can this knowledge be applied in real-world cloud infrastructure management?**
This challenge reinforced the importance of securing access to cloud resources and managing permissions effectively. Using Bastion Hosts and SSM for secure access and understanding IAM roles is key to maintaining a secure, scalable cloud infrastructure.

# Conclusion

This challenge provided hands-on experience with securing access to EC2 instances using a Bastion Host and AWS Systems Manager (SSM). By configuring Terraform and resolving IP fetching issues, I gained insights into managing secure access and permissions in AWS. The process highlighted the importance of leveraging IAM roles and avoiding open SSH access, enhancing overall security. The skills learned are essential for managing cloud infrastructure securely in real-world environments.

