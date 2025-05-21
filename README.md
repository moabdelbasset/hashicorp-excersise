# hashicorp-excersise

## Initial Excersise: Setup a basic AWS secret Engine using the documentation (https://www.vaultproject.io/docs/secrets/aws)

## Installing Vault on Ubuntu
**Official Docs**: [Install Vault](https://developer.hashicorp.com/vault/docs/install)

## Steps:
1. Install Vault
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl gnupg software-properties-common

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install vault -y
```

2. Start Vault in dev mode and export the Vault address and Vault token
vault --version
vault server -dev -dev-listen-address="0.0.0.0:8200"
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=<vault-root-token>


