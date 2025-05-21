# hashicorp-excersise

## Initial Excersise: Setup a basic AWS secret Engine using the documentation (https://www.vaultproject.io/docs/secrets/aws)

### Steps: 
1. Install Vault **Official Docs**: [Install Vault](https://developer.hashicorp.com/vault/docs/install)
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl gnupg software-properties-common

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install vault -y
```

2. Start Vault in dev mode and export the Vault address and Vault token [Dev Mode](https://developer.hashicorp.com/vault/docs/concepts/dev-server)
```bash
vault --version
vault server -dev -dev-listen-address="0.0.0.0:8200"
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=<vault-root-token>
```

3. Enable and Configure the AWS Secrets Engine
```bash
vault secrets enable aws
vault write aws/config/root \
    access_key=<AWS_ACCESS_KEY_ID> \
    secret_key=<AWS_SECRET_ACCESS_KEY> \
    region=us-east-1
```
4. Transferring command to API [AWS Secret Engine API](https://developer.hashicorp.com/vault/api-docs/secret/aws)
```bash
ubuntu@ip-172-31-93-62:~$ curl --header "X-Vault-Token: abc123abc123" \
     --request POST \
     --data '{
       "credential_type": "iam_user",
       "policy_document": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":\"ec2:*\",\"Resource\":\"*\"}]}"
     }' \
     http://127.0.0.1:8200/v1/aws/roles/my-role
```
5.  Creating the HCL file
```bash

$ cat aws-role-writer.hcl
path "aws/roles/*" {
  capabilities = ["create", "update"]
}

path "aws/config/root" {
  capabilities = ["read"]
}

path "sys/mounts/aws" {
  capabilities = ["create", "update"]
}
ubuntu@ip-172-31-93-62:~$ vault policy write aws-role-writer aws-role-writer.hcl
Success! Uploaded policy: aws-role-writer
```
6.  Run the command again
```bash
ubuntu@ip-172-31-93-62:~$ vault write aws/roles/my-role \
credential_type=iam_user \
policy_document=-<<EOF
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "ec2:*",
"Resource": "*"
}
]
}
EOF
Success! Data written to: aws/roles/my-role
```

## Option 1 - Dev server & TLS
### Customer response:
Hi <Customer name>
Thanks for reaching out and for sharing the details of the issue you're encountering during your initial Vault setup.
Regarding the error message you are getting
```bash
http: server gave HTTP response to HTTPS client
```
This indicates that Vault is running over HTTP (as it does by default in -dev mode), but your client (in this case, the vault CLI) is attempting to connect using HTTPS.

To resolve this please set the VAULT_ADDR environment variable to HTTP:// instead of HTTPS:// as follow:
```bash
export VAULT_ADDR=http://127.0.0.1:8200
```
Once you’ve done that, try running vault status again.

Let me know if that resolves your issue or if you need help configuring Vault for production use with HTTPS and TLS enabled.

Thanks,
Mohamed

### Troubleshooting
I got this error when installing Vault and did research and found this [page](https://stackoverflow.com/questions/63878533/vault-error-server-gave-http-response-to-https-client) 


## Option 2 - Generate-Root API
### Customer response:
Hi [Customer Name],

Thank you for your question and for referencing the Vault documentation regarding root token generation via the API. You're absolutely right—the documentation outlines how to generate and submit unseal keys to complete the root generation process, but it does not explicitly cover how to decode the encoded root token (using the OTP) via the API.

With that said I am going to open a feature request to have this functionality ready and for now as a workaround you can try using the CLI manually for decoding
. Use the CLI for the decode
```bash
vault operator decode-root -otp=<otp> -encoded-token=<encoded_token>
```

Let me know if this solution is what you are looking for.

Thanks,
Mohamed

### Troubleshooting
The customer referenced two sources and neither guide includes an API equivalent to the vault operator decode-root command. I assumed that it could be a product limitation and there is a room for a feature request. 
I checked internally with the team to see other opinions and ask the development team and they agreed that it can be raised as a feature request to be included in future releases.
And as a workaround for the customer I provided a temp solution to use the CLI
