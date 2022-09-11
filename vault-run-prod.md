# Configure Production Environment

## Create Data Directories

```
VAULT_HOME_DIR="/etc/vault"
VAULT_CONFIG_FILE="$VAULT_HOME_DIR/config.hcl"
VAULT_LIB_DIR="/var/lib/vault"
VAULT_LIB_DIR_DATA="/var/lib/vault/data"
sudo mkdir -p $VAULT_HOME_DIR
sudo mkdir -p $VAULT_LIB_DIR
```

## Create User

```
sudo useradd --system --home $VAULT_HOME_DIR --shell /bin/false vault
sudo chown -R vault:vault $VAULT_HOME_DIR $VAULT_LIB_DIR
```

## Create a Vault Service file

```
cat <<EOF | sudo tee /etc/systemd/system/vault.service
[Unit]
Description="HashiCorp Vault - A tool for managing secrets"
Documentation=https://www.vaultproject.io/docs/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault/config.hcl

[Service]
User=vault
Group=vault
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
NoNewPrivileges=yes
ExecStart=/usr/local/bin/vault server -config=/etc/vault/config.hcl
ExecReload=/bin/kill --signal HUP
KillMode=process
KillSignal=SIGINT
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
StartLimitBurst=3
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

## Create Vault /etc/vault/config.hcl file

```
touch $VAULT_CONFIG_FILE
```

## Add basic configuration settings for Vault to config.hcl file

```
cat <<EOF | sudo tee $VAULT_CONFIG_FILE
disable_cache = true
disable_mlock = true
ui = true
listener "tcp" {
   address          = "0.0.0.0:8200"
   tls_disable      = 1
}
storage "file" {
   path  = "$VAULT_LIB_DIR_DATA"
}
api_addr         = "http://0.0.0.0:8200"
max_lease_ttl         = "10h"
default_lease_ttl    = "10h"
cluster_name         = "vault"
raw_storage_endpoint     = true
disable_sealwrap     = true
disable_printable_check = true
EOF
```


## Start and enable vault service to start on system boot

```
sudo systemctl daemon-reload
sudo systemctl enable --now vault
systemctl status vault
```

# Export VAULT_ADDR environment variable before you initialize Vault server - Replace 127.0.0.1 with Vault Server IP address
```
export VAULT_ADDR=http://127.0.0.1:8200
echo "export VAULT_ADDR=http://$SERVER_IP:8200" >> ~/.bashrc
```

# Start initialization with the default options by running the command below

```
sudo rm -rf $VAULT_LIB_DIR_DATA/*
vault operator init > $VAULT_HOME_DIR/init.file
```
