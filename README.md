# PGP Decryption for Transfer Family

## Project Architecture Diagram
![image](https://user-images.githubusercontent.com/59907142/183982218-031ca1a6-41cc-4dac-9a1c-c82c0faf49eb.png)

---


## Project Requirements
### Steps Completed by setupEnvironment.sh script
- Custom Lambda Layer including required binary/python package.

### Steps Completed by CloudFormation Stack
- REQUIRED - Setup Environment Stack: Creates necessary IAM Roles, Lambda Function, PGP_PrivateKey Secret, and S3 Bucket.

- OPTIONAL - Custom Transfer Family Identity Provider Stack: Creates a Transfer Family server with a custom Lambda identity provider.

### Steps Completed Manually via AWS Console
- Paste PGP Private Key into PGP_PrivateKey secret within Secrets Manager console.
- Attach Transfer Family Managed Workflow to Transfer Family server within Transfer Family console.

---

### Overview of Process
1. Open CloudShell and clone this GitHub repository.
2. Run setupEnvironment.sh bash script.
3. Deploy <a href = "https://github.com/bukuwarung/pgp-decryption-for-transfer-family/blob/main/template.yaml"> this CloudFormation stack </a> which creates necessary IAM Roles, Lambda Function, PGP_PrivateKey Secret, and S3 Bucket.
4. Create a Transfer Family Server / Transfer Family User. (Must pick one of the following options)
    - Option 1: Deploy a Transfer Family server with a custom Secrets Manager based identity provider via CloudFormation stack.
    - Option 2: Use an existing Transfer Family server that is already configured within your environment.
    - Option 3: Manually create your own Transfer Family server / Transfer Family managed user via the AWS Console.
5. Add your PGP private key in Secrets Manager.
6. Attach Transfer Family Managed Workflow to Transfer Family server.

---

## Step-by-Step Instructions

### CloudShell - Deploying setupEnvironment bash script and CloudFormation stacks.

- Open CloudShell within your AWS account.
- Run the following command to clone this Git repository to access the required files for this project:

  `git clone https://github.com/bukuwarung/pgp-decryption-for-transfer-family`

- Change into the new pgp-decryption-for-transfer-family directory:

  `cd pgp-decryption-for-transfer-family/`

- Run this command to give the setupEnvironment.sh script executable permissions:

  `chmod +x setupEnvironment.sh`

- Run this command to create the required IAM roles and Lambda layer:

  `./setupEnvironment.sh`

- Now, deploy the CloudFormation stack that will build IAM roles and Lambda function:

  `sam deploy --guided --capabilities CAPABILITY_NAMED_IAM`

    - Respond to the following prompts:
        - Enter in a stack name:
        - Select a region: **Press enter to leave as default**
        - Enter in a name for the S3 bucket:
        - Confirm changes before deploy[y/N]: **n**
        - Allow SAM CLI IAM role creation[Y/n]: **y**
        - Disable rollback [y/N]: **n**
        - Save arguments to configuration file [Y/n]: **y**
        - SAM configuration file [samconfig.toml]: **Press enter to leave as default**
        - SAM configuration environment [default]: **Press enter to leave as default**

 ![image](https://user-images.githubusercontent.com/59907142/182432098-b61d2272-4c41-4663-807d-11e2c54e427c.png)



---

### Creating Transfer Family Server
- NOTE: This project requires a Transfer Family server, you must choose one of these three options:
    - Option 1: Deploy a Transfer Family server with a custom Secrets Manager based identity provider via CloudFormation stack.
    - Option 2: Use an existing Transfer Family server that is already configured within your environment.
    - Option 3: Manually create your own Transfer Family server / Transfer Family managed user via the AWS Console.


#### Option 1: Deploying CloudFormation Stack to Create Transfer Family Server with Custom Identity Provider
- Refer to this link for detailed instructions on deploying a CloudFormation stack that will create a Transfer Family server, custom identity provider, and all the required IAM policies:  <a href = "https://aws.amazon.com/blogs/storage/enable-password-authentication-for-aws-transfer-family-using-aws-secrets-manager-updated/">Enable password authentication for AWS Transfer Family using AWS Secrets Manager</a>
- Short Summary of Steps Required:
  - In CloudShell, run the following:

    - Create a new directory for this CloudFormation stack and change into the new directory:

        `mkdir tmp`

        `cd tmp`

    - Download the CloudFormation stack using the link mentioned on the blog post linked above, at the time of creating this, the command is as follows:

        `wget https://s3.amazonaws.com/aws-transfer-resources/custom-idp-templates/aws-transfer-custom-idp-secrets-manager-sourceip-protocol-support-apig.zip`

    - After downloading the zip, unzip it:

        `unzip aws-transfer-custom-idp-secrets-manager-sourceip-protocol-support-apig.zip`

    - Run the following command:

        `sam deploy --guided`

    - Respond to the following prompts:
        - Enter in a stack name:
        - Select a region: **Press enter to leave as default**
        - Parameter CreateServer [true]: **Press enter to leave as default**
        - Parameter SecretsManagerRegion []: **Press enter to leave as default**
        - Parameter TransferEndpointType [PUBLIC]: **Press enter to leave as default**
        - Parameter TransferSubnetIDs []: **Press enter to leave as default**
        - Parameter TransferVPCID []: **Press enter to leave as default**
        - Confirm changes before deploy[y/N]: **n**
        - Allow SAM CLI IAM role creation[Y/n]: **y**
        - Disable rollback [y/N]: **n**
        - Save arguments to configuration file [Y/n]: **y**
        - SAM configuration file [samconfig.toml]: **Press enter to leave as default**
        - SAM configuration environment [default]: **Press enter to leave as default**


![image](https://user-images.githubusercontent.com/59907142/182436028-902168af-c2ce-4986-aa6a-827c9ce5c475.png)


#### Option 1 Continued: Create Custom Transfer Family Identity Provider User Account in Secrets Manager
- Naviage to the AWS Secrets Manager console (https://console.aws.amazon.com/secretsmanager)
    - Create a new secret by choosing Store a new secret.
        - Choose Other type of secret.
        - Create the following key-value pairs. The key names are case-sensitive.


<div align="center">

|         Secret Key                                                               |     Secret Value                                                                 |
|:--------------------------------------------------------------------------------:|:--------------------------------------------------------------------------------:|
|       Password                                                                   |        TestPassword1234!                                                         |
|       Role                                                                       |      arn:aws:iam::**INSERT-ACCOUNT-ID**:role/PGPDecryptionTransferFamilyUserRole |
|       HomeDirectoryDetails                                                       |      [{"Entry": "/", "Target": "/**INSERT-S3-BUCKET-NAME/INSERT-USER-NAME**"}]   |
|       HomeDirectoryType                                                          |        LOGICAL                                                                   |

</div>

#### Option 1 Continued: Getting Required Values from CloudFormation Console
- To get the specific role ARN and S3 bucket name, go to the CloudFormation console and select:

    -  Stacks -> Stack 1 Name (Ex. pgpdecryptionstack) -> Outputs


![image](https://user-images.githubusercontent.com/59907142/183429862-5a25cef5-c4d8-4927-85bc-c9367e2a1c01.png)

#### Option 1 Continued: Finish Creating the Secret
 - Click "Next"
 - Name the secret in the format: **serverID/username**
    - If you deployed Transfer Family CloudFormation stack:
        - Go to CloudFormation console and select: Stacks -> Stack 1 Name (Ex. transferFamilyServer) -> Outputs
            - Select "ServerId"
    - If you did not deploy Transfer Family CloudFormation stack:
        - Go to the Transfer Family console, select "Servers", and then select the appropriate serverId.

 - Select "Next" -> "Next" -> "Store"

 <div align="center"> <strong> Example Secret for Reference </strong>

![image](https://user-images.githubusercontent.com/59907142/182291079-499c7d6e-17e7-4757-a843-f7b94ecd6666.png)
![image](https://user-images.githubusercontent.com/59907142/183749422-e96d0fd5-7ee0-413b-a351-f2d1cb801c64.png)

  </div>

---

#### Option 2: Use an existing Transfer Family Server + Transfer Family User Account
- No configuration needed at this time, proceed to next step of pasting your PGP Private Key into the PGP_PrivateKey secret within Secrets Manager.

---

#### Option 3: Manually Create a Transfer Family Server + Transfer Family User Account via AWS Console
- If you don't want to deploy the custom Transfer Family identity provider via CloudFormation mentioned in option 1, and you don't have an active Transfer Family server, please refer to this link for instructions on how to create a new Transfer Family server + Transfer Family managed user: https://docs.aws.amazon.com/transfer/latest/userguide/getting-started.html

---

### Adding Private Key to Secrets Manager
- Navigate to the AWS Secrets Manager console: https://console.aws.amazon.com/secretsmanager
- Select "Secrets"
- Select the secret named: "PGP_PrivateKey"
- Select "Retrieve secret value"
- Select "Edit"
- Remove the text: "Within the Secrets Manager console, paste your PGP private key here"
- Paste in your PGP Private key
- Select "Save"

---

### Attach Managed Workflow to Transfer Family Server
- On the Transfer Family console, select "Servers"
- Select your desired Transfer Family server
- Under "Additional details", select "Edit"
- Select the Workflow with the description: "Transfer Family Workflow for PGP decryption process"
- Select the Managed workflow execution role with the name: "PGPDecryptionManagedWorkflowRole"
- Select "Save"

---

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

---

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

