# Descomplicando o Nomad


### Run a VM with nomad installed:
```
vagrant up
```

```
vagrant ssh
```

### Start your server / client
```
sudo nomad agent -dev -bind 0.0.0.0 -log-level INFO
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
vault server -dev -dev-listen-address :8200 &
export VAULT_ADDR='http://10.10.10.10:8200'


```

## Integrate Vault with Nomad
[https://www.nomadproject.io/docs/configuration/vault]
```
curl https://nomadproject.io/data/vault/nomad-server-policy.hcl -O -s -L
vault policy write nomad-server nomad-server-policy-vault.hcl 
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
#note token to add to nomad server - vault stanza

## https://www.nomadproject.io/docs/configuration/vault
# Configuring Nomad Servers
cat <<EOF >> /etc/nomad.d/nomad.hcl
vault {
    enabled = true
    address = "http://10.10.10.10:8200"
    task_token_ttl = "1h"
    create_from_role = "nomad-cluster¨
    token = ""
}
EOF

systemctl restart nomad.service

#Configuring Nomad Clients
cat <<EOF >> /etc/nomad.d/nomad.hcl
vault {
    enabled = true
    address = "http://10.10.10.10:8200"
}
EOF

systemctl restart nomad.service
```


