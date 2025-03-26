# CloudFoxable CTF Challenge: It's Another Secret

## **Overview**
The **CloudFoxable CTF Challenge: It's Another Secret** tests your ability to enumerate and extract sensitive information from AWS environments using **CloudFox** and **AWS CLI**. In this challenge, you will retrieve a hidden secret stored in **AWS Systems Manager (SSM) Parameter Store**.

---

## **Objectives**
By completing this challenge, you will:
- Understand AWS IAM policies and permissions.
- Use **CloudFox** to enumerate AWS secrets.
- Retrieve sensitive data using **AWS CLI**.
- Analyze IAM permissions to understand access rights.

---

## **Solution**

### **Step 1: Configure AWS Profile and Verify Access**
Edit your AWS configuration file to add the **Ertz** profile:
```bash
nano ~/.aws/config
```
Add the following content:
```
[profile ertz]
region = us-west-2
role_arn = arn:aws:iam::Account_ID:role/Ertz
source_profile = ashish_security
```

Verify that you can assume the role:
```bash
aws --profile ertz sts get-caller-identity
```
Expected output:
```json
{
    "UserId": "AROAXG4CFIKP3Z2V7JDZF:botocore-session-1742960010",
    "Account": "Account_ID",
    "Arn": "arn:aws:sts::Account_ID:assumed-role/Ertz/botocore-session-1742960010"
}
```

---

### **Step 2: Identify Attached IAM Policies**
Check the attached policies for the **Ertz** role:
```bash
aws --profile ashish_security iam list-attached-role-policies --role-name Ertz
```
Expected output:
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "its-another-secret-policy",
            "PolicyArn": "arn:aws:iam::Account_ID:policy/its-another-secret-policy"
        }
    ]
}
```

---

### **Step 3: Inspect IAM Policy Details**
Retrieve the policy details:
```bash
aws --profile ashish_security iam get-policy --policy-arn arn:aws:iam::Account_ID:policy/its-another-secret-policy
```
Expected output:
```json
{
    "Policy": {
        "PolicyName": "its-another-secret-policy",
        "PolicyId": "ANPAXG4CFIKPS7S7KOQDB",
        "Arn": "arn:aws:iam::Account_ID:policy/its-another-secret-policy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "policy that only allows access to the its-another-secret flag",
        "CreateDate": "2025-03-05T22:48:55+00:00",
        "UpdateDate": "2025-03-05T22:48:55+00:00",
        "Tags": []
    }
}
```

---

### **Step 4: Inspect Policy Version and Permissions**
Retrieve the policy version document:
```bash
aws --profile ashish_security iam get-policy-version --policy-arn arn:aws:iam::Account_ID:policy/its-another-secret-policy --version-id v1
```
Expected output:
```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "ssm:GetParameter"
                    ],
                    "Effect": "Allow",
                    "Resource": [
                        "arn:aws:ssm:us-west-2:Account_ID:parameter/cloudfoxable/flag/its-another-secret"
                    ]
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2025-03-05T22:48:55+00:00"
    }
}
```

---

### **Step 5: Retrieve the Secret**
Execute the following command to fetch the secret from SSM Parameter Store:
```bash
aws --profile ertz ssm get-parameter --name /cloudfoxable/flag/its-another-secret --with-decryption --region us-west-2
```
Expected output:
```json
{
    "Parameter": {
        "Name": "/cloudfoxable/flag/its-another-secret",
        "Type": "SecureString",
        "Value": "FLAG{ItsAnotherSecret::ThereWillBeALotOfAssumingRolesInThisCTF}",
        "Version": 1,
        "LastModifiedDate": "2025-03-05T17:48:49.480000-05:00",
        "ARN": "arn:aws:ssm:us-west-2:Account_ID:parameter/cloudfoxable/flag/its-another-secret",
        "DataType": "text"
    }
}
```

---

## **Analysis Questions & Answers**

Here are some insightful questions that reflect the thought process and challenges faced during this CTF exercise:

1. **What approach did you use to verify the successful assumption of the Ertz role?**
   - I used the command `aws --profile ertz sts get-caller-identity` to confirm the role assumption. The output displayed the correct UserId and ARN, indicating that I had successfully assumed the role.

2. **Why did you use the `ashish_security` profile to inspect the Ertz role's policies?**
   - The Ertz role itself lacked permissions to view its attached policies, so I leveraged the `ashish_security` profile to list and inspect the policies, as it had the necessary IAM privileges to access policy details.

3. **What did you learn from inspecting the `its-another-secret-policy`?**
   - The policy granted the `ssm:GetParameter` permission specifically for the resource `arn:aws:ssm:us-west-2:AccountID:parameter/cloudfoxable/flag/its-another-secret`. This confirmed that the secret was stored in the **SSM Parameter Store** rather than **Secrets Manager**.

4. **Why did the initial attempts to retrieve the secret fail?**
   - The errors occurred because I initially assumed the secret was in **Secrets Manager** and used `secretsmanager:GetSecretValue`, which the role did not have permission for. Once I realized that the secret was stored in **SSM Parameter Store**, I adjusted the command accordingly.

5. **What key takeaway did you gain from this challenge?**
   - This challenge reinforced the importance of properly analyzing IAM policies and accurately identifying resource permissions. Misconfigured policies can lead to either excessive or insufficient permissions, making it crucial to review roles and policies regularly.

---

## **Conclusion**
By following these steps, you successfully retrieved the secret flag from AWS SSM Parameter Store. This challenge demonstrates the importance of correctly analyzing IAM policies and understanding AWS security configurations.

---

## **References**
- [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/)
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [CloudFox GitHub Repository](https://github.com/BishopFox/cloudfox)

