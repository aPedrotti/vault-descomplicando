# Descomplicando o Nomad


### Run a VM with nomad installed:
```
vagrant up

vagrant ssh
```

### Start your Nomad Server / Client
```
sudo nomad agent -dev -bind 0.0.0.0 -log-level INFO &
```

### Main Nomad commands
```
nomad node status
nomad server members

# To generate a sample of job / deploy - example.nomad
nomad job init 

nomad job run <file.nomad>
nomad job status
```

## Vault start
```
vault server -dev -dev-listen-address :8200 -dev-root-token-id naosei &
export VAULT_ADDR='http://10.0.2.15:8200'


```

## Integrate Vault with Nomad
[https://www.nomadproject.io/docs/configuration/vault]
```
curl https://nomadproject.io/data/vault/nomad-server-policy.hcl -O -s -L
vault policy write nomad-server nomad-server-policy.hcl
# A Vault token role must be created 
cat <<EOF > nomad-cluster-role.json
{
  "disallowed_policies": "nomad-server",
  "allowed_policies": "access-tables",
  "token_explicit_max_ttl": 0,
  "name": "nomad-cluster",
  "orphan": true,
  "token_period": 259200,
  "renewable": true
}
EOF

vault write /auth/token/roles/nomad-cluster @nomad-cluster-role.json

vault token create -policy nomad-server -period 72h -orphan 
# orphan means that it does not take into consideration parents periods policy
# take note token to add to nomad server - vault stanza
```

### Configuring Nomad Servers
#### https://www.nomadproject.io/docs/configuration/vault
```
cat <<EOF >> /etc/nomad.d/nomad.hcl
vault {
    enabled = true
    address = "http://10.0.2.15:8200"
    task_token_ttl = "1h"
    create_from_role = "nomad-cluster¨
    token = ""
}
EOF
systemctl restart nomad.service
```
### Configuring Nomad Clients
```
cat <<EOF >> /etc/nomad.d/nomad.hcl
vault {
    enabled = true
    address = "http://10.0.2.15:8200"
}
EOF

systemctl restart nomad.service
```

## Configuring Dynamic Database 
```
# Enables database in a path
vault secrets enable -path dbs database

# Deploy Mysql Application
nomad run vault-secrets-database/0-mysql.nomad

# Config Connection
vault write dbs/config/mysql @vault-secrets-database/1-connection-mysql.json

# Configure Vault Access Management and TTL
vault write dbs/roles/accessdb @vault-secrets-database/2-accessdb-role-mysql.json

# Configure policy to be able to read credentials
vault policy write mysql-read-policy vault-secrets-database/3-access-tables-policy-mysql.hcl

# Confirms credential reading
vault read dbs/creds/accessdb

# Deploy the app to comunicate:
nomad run vault-secrets-database/app.nomad #currently not working - "Vault not enabled and Vault policies requested" 

## For Postgres - not being able to read creds 
nomad run vault-secrets-database/postgres.nomad
vault write dbs/config/postgresql @vault-secrets-database/1-connection-postgres.json
vault write dbs/roles/accessdb_pgsql @vault-secrets-database/2-accessdb-role-posgres.json
vault policy write postgres-read-policy vault-secrets-database/3-access-tables-policy-pgsql.hcl
vault read dbs/creds/accessdb_pgsql
```
