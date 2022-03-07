# Vault
## Samples of Handling a KEY VALUE secrets
[https://learn.hashicorp.com/tutorials/vault/versioned-kv]
```
vault kv put secret/test key="some value" # This path only receives secrets because it is already configured
vault kv get secret/test
vault kv metadata get secret/test
vault kv put secret/test key="something different"
vault kv get -version=1 secret/test
vault kv get -version=2 secret/test
vault kv get -format=json -version=2 secret/test |jq -r .data.data.key
vault kv delete secret/test
vault kv get secret/test
vault kv undelete -versions=2 secret/test 
vault kv put secret/company/acme name="Acme Inc." contact="andre@acme.com"
vault kv patch secret/company/acme contact="admin@acme.com"
vault kv metadata put -custom-metadata=Membership="Platinum" secret/customer/acme
vault kv metadata put -custom-metadata=Membership="Platinum" -custom-metadata=Region="US West" secret/customer/acme
vault write secret/test max_versions=4
vault kv destroy -versions=4 secret/customer/acme # permanently
vault read secret/config
vault kv metadata put -delete-version-after=40s secret/test
vault kv put secret/test message="data1"
vault kv get secret/test 
vault secrets enable -path=app kv
```
## Integrate with AWS
```
vault secrets enable -path=aws aws
vault write aws/config/root \
    access_key=$AWS_ACCESS_KEY_ID \
    secret_key=$AWS_SECRET_ACCESS_KEY \
    region=us-east-1
vault write aws/roles/my-role \
        credential_type=iam_user \
        policy_document=iam-role.json
vault read aws/creds/my-role
```

## Getting help with Paths
```
#vault path-help <path_name>
vault path-help aws
```

