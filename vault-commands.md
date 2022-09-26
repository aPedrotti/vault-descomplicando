# Vault
```
agent      auth       delete     lease      login      namespace  path-help  policy     read       server     status     unwrap     
audit      debug      kv         list       monitor    operator   plugin     print      secrets    ssh        token      write
```
## AUTH

```bash
vault auth enable userpass

vault write auth/userpass/users/<name> password=<pwd>

vault login -method=userpass username=<name> password=<pwd>

vault token lookup
``` 

## SECRETS - https://www.vaultproject.io/docs/secrets
```
disable  enable   list     move     tune

```
### LIST
defaults
```
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_3d85f070    per-token private secret storage
identity/     identity     identity_92c2b33e     identity store - only seen as 'root'
secret/       kv           kv_4829f5eb           key/value secret storage
sys/          system       system_46221ea4       system endpoints used for control, policy and debugging
```
### Getting help with Paths
```
#vault path-help <path_name>
vault path-help aws
```

### ENABLE
```
# defaults
aws consul database generic pki plugin rabbitmq ssh totp transit

# XARGS
-address                       -ca-cert                       -description                   -local                         -output-curl-string            -seal-wrap                     -wrap-ttl
-agent-address                 -ca-path                       -external-entropy-access       -max-lease-ttl                 -passthrough-request-headers   -tls-server-name               
-allowed-response-headers      -client-cert                   -force-no-cache                -mfa                           -path                          -tls-skip-verify               
-audit-non-hmac-request-keys   -client-key                    -header                        -namespace                     -plugin-name                   -unlock-key                    
-audit-non-hmac-response-keys  -default-lease-ttl             -listing-visibility            -options                       -policy-override               -version
```

```
# types 
#custom path
vault secrets enable -path=app kv #

## Integrate with AWS

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

## KV
#### delete    destroy     enable-versioning   get     list     metadata   patch   put     rollback    undelete           
#### Samples of Handling a KEY VALUE secrets
[https://learn.hashicorp.com/tutorials/vault/versioned-kv]
```
vault secret enable 
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
vault kv destroy -versions=4 secret/customer/acme # permanently destroy values - preserves metadata

vault read secret/config
vault kv metadata put -delete-version-after=10s secret/test
vault kv put secret/test message="data1"
vault kv get secret/test sleep 11 && vault kv get secret/test 

```
