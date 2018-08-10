### Create Cloudfront Keypair

Create AWS account and go to the my security credentials menu, if popup, click "Continue to Security Credentials":  
  
![image](./images/security-credentials-button.png?raw=true)
  
Then expand the "CloudFront key pairs" section and click on "Create New Key Pair":  
  
![image](./images/cloudfront-keypair-expanded.png?raw=true)
  
Download the private key:  
  
![image](./images/download-keypair.png?raw=true)
  
Make a note of the Access Key Id (also contained in the file name of the private key file):  
  
![image](./images/keypair-id.png?raw=true)

### Create a cross account role

While still in the IAM console, click on Roles in the side menu and then Create Roles button.    

![image](./images/create-role.png?raw=true)
  
Click on "Another AWS account" button:

![image](./images/another-account.png?raw=true)  
  
Enter the account id supplied and optionally enter an external id or enable MFA. *(Users could enable either of these two  
options and they would have to give the externalid and mfa code to the application when AssumeRole is called, need to handle it  
appropriately or tell them not to click either option)*  
  
Don't add any permissions yet, click next and give the role a name. *(This name can be restricted and this will help with greater  
security, the application can reject any role that is not called e.g. "ExampleRoleName")*
  
Add permissions inline, copy and paste the supplied iam policy document:  

![image](./images/role-add-inline.png?raw=true)
![image](./images/paste-policy-json.png?raw=true)

#### Optional steps
  
Modify the trust relationship of the role from allowing all users from other account access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "test-id"
        }
      }
    }
  ]
}
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AccountId>:user/<SpecificUsername>"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "test-id"
        }
      }
    }
  ]
}
```

That's the setup for the user complete, the rest is handled by the application.  


### To test access to the other account from the AWS-CLI

Create a user in the IAM console and add this policy inline (replace "ExampleRoleName" with the name of the role created above):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::*:role/ExampleRoleName"
        }
    ]
}
```

Then create a new profile and add the credentials to the aws-cli e.g.

```json
aws configure --profile crossaccount
## set environment in usual way (powershell)
$Env:AWS_PROFILE = "crossaccount"

aws sts assume-role --role-arn "arn:aws:iam::<AccountId>:role/ExampleRoleName" --role-session-
name "cross-account-session" --external-id "test-id-5678"
```

```json
{
    "AssumedRoleUser": {
        "AssumedRoleId": "",
        "Arn": ""
    },
    "Credentials": {
        "SecretAccessKey": "",
        "SessionToken": "",
        "Expiration": "",
        "AccessKeyId": ""
    }
}
```

```json
Set the following environment variables from above:
$Env:AWS_SECRET_ACCESS_KEY="SecretAccessKey"
$Env:AWS_ACCESS_KEY_ID="AccessKeyId"
$Env:AWS_SESSION_TOKEN="SessionToken"
```
  
Can now send commands in other account, similar process is used with any of the aws-sdk such as:  

```json
aws cloudformation deploy --template-file template.cform --stack-name TestStackName
```
  


