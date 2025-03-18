# CloudFoxable CTF Challenge: It's a Secret

## **Overview**
The **CloudFoxable CTF Challenge** is designed to test your ability to enumerate and extract sensitive information from AWS environments using **CloudFox** and AWS CLI. In this challenge, you will retrieve a hidden secret stored in AWS Systems Manager (SSM) Parameter Store.

## **Objectives**
By completing this challenge, you will:
- Understand AWS IAM policies and permissions.
- Use **CloudFox** to enumerate AWS secrets.
- Retrieve sensitive data using **AWS CLI**.
- Analyze IAM permissions to understand access rights.

---

## **Solution**

### **Step 1: Verify AWS Credentials**
Before proceeding, ensure that your AWS CLI profile is correctly configured:
```sh
aws configure --profile cloudfoxable
```
Verify your credentials:
```sh
aws --profile cloudfoxable sts get-caller-identity
```
Expected output:
```json
{
    "UserId": "AIDXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/ctf-starting-user"
}
```

---

### **Step 2: Enumerate Secrets Using CloudFox**
Run CloudFox to identify AWS secrets stored in Secrets Manager and SSM Parameter Store:
```sh
cloudfox aws -p cloudfoxable secrets -v2
```
CloudFox will generate a loot file. Locate and inspect the secrets list:
```sh
cat ~/.cloudfox/cloudfox-output/aws/cloudfoxable/table/secrets.txt
```
Identify `/cloudfoxable/flag/its-a-secret` in the output.

---

### **Step 3: Identify Attached IAM Policies**
Check the IAM policies assigned to the `ctf-starting-user`:
```sh
aws --profile ashish_security iam list-attached-user-policies --user-name ctf-starting-user
```
Expected output:
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "SecurityAudit",
            "PolicyArn": "arn:aws:iam::aws:policy/SecurityAudit"
        },
        {
            "PolicyName": "bastion-ssm",
            "PolicyArn": "arn:aws:iam::495804826271:policy/bastion-ssm"
        },
        {
            "PolicyName": "CloudFox-policy-perms",
            "PolicyArn": "arn:aws:iam::495804826271:policy/CloudFox-policy-perms"
        },
        {
            "PolicyName": "its-a-secret-policy",
            "PolicyArn": "arn:aws:iam::495804826271:policy/its-a-secret-policy"
        }
    ]
}
```

---

### **Step 4: Inspect IAM Policy Permissions**
Retrieve the policy document for `its-a-secret-policy`:
```sh
aws --profile ashish_security iam get-policy-version --policy-arn arn:aws:iam::495804826271:policy/its-a-secret-policy --version-id v1
```
Look for the `ssm:GetParameter` permission:
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
                        "arn:aws:ssm:us-west-2:495804826271:parameter/cloudfoxable/flag/its-a-secret"
                    ]
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2025-03-05T22:48:50+00:00"
    }
}
```

---

### **Step 5: Retrieve the Secret**
Now that we have confirmed the correct permissions, retrieve the secret from AWS Systems Manager (SSM) Parameter Store:
```sh
aws --profile ashish_security ssm get-parameter --name "/cloudfoxable/flag/its-a-secret" --with-decryption --region us-west-2
```
Expected output:
```json
{
    "Parameter": {
        "Name": "/cloudfoxable/flag/its-a-secret",
        "Type": "SecureString",
        "Value": "FLAG{ItsASecret::IsASecretASecretIfTooManyPeopleHaveAccessToIt?}",
        "Version": 1,
        "LastModifiedDate": "2025-03-05T17:48:45.510000-05:00",
        "ARN": "arn:aws:ssm:us-west-2:495804826271:parameter/cloudfoxable/flag/its-a-secret",
        "DataType": "text"
    }
}
```
Copy the `FLAG{your-secret-flag-here}` value for submission.

---

## **Analysis Questions & Answers**

1. **What specific IAM policy granted access to the secret?**  
   The `its-a-secret-policy` policy granted access by including the `ssm:GetParameter` action for the specific SSM parameter resource.

2. **What AWS service stored the secret, and why is it used in real-world applications?**  
   The secret was stored in AWS Systems Manager (SSM) Parameter Store. This service is used for securely storing configuration data and secrets, such as API keys, database credentials, and encryption keys.

3. **How could AWS administrators prevent unauthorized users from accessing sensitive secrets?**  
   Administrators should follow the principle of least privilege, applying IAM policies with precise permissions. Secrets should be encrypted using AWS Key Management Service (KMS) and access logs should be monitored using AWS CloudTrail.

4. **If the secret were stored in AWS Secrets Manager instead of SSM, what would change in the retrieval process?**  
   Instead of using `ssm get-parameter`, we would use:
   ```sh
   aws secretsmanager get-secret-value --secret-id "/cloudfoxable/flag/its-a-secret" --profile ashish_security --region us-west-2
   ```
   AWS Secrets Manager also provides automatic secret rotation, which SSM Parameter Store does not.

5. **Why does CloudFox help in pentesting AWS environments, and how could attackers misuse this tool?**  
   CloudFox automates the enumeration of AWS resources, helping security professionals identify misconfigurations and security risks. However, if used by attackers, it could be leveraged to discover exposed credentials, excessive IAM permissions, and unprotected secrets.

---

## **Conclusion**
This challenge demonstrated the importance of **IAM policies**, **AWS security auditing**, and **CloudFox reconnaissance**. By leveraging **CloudFox** and **AWS CLI**, you successfully:
- Enumerated AWS secrets.
- Retrieved sensitive data from AWS SSM Parameter Store.
- Understood IAM policy configurations and their implications.

For real-world applications, enforcing strict **IAM policies**, encrypting secrets, and monitoring access logs are crucial to securing cloud environments.

---

## **References**
- [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
- [AWS CLI SSM Commands](https://docs.aws.amazon.com/cli/latest/reference/ssm/)
- [CloudFox GitHub Repository](https://github.com/BishopFox/cloudfox)

---

## **License**
This project is licensed under the [MIT License](LICENSE).

