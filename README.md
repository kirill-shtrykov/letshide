# Let's Hide!
A Bash application that monitors the renewal of Let's Encrypt certificates 
and uploads the updated certificates to Hashicorp Vault. 
Ensure your certificates are always up-to-date and securely stored with automated synchronization.

# Installation
## Create Vault Policy
The application should have rights to renew self token and write to the path of the certificates storage.
For example we create `letsencrypt` policy and use the mount point `kv` 
and the storage path `ssl` in the root of the mount point.
```shell
tee letsencrypt-policy.hcl <<EOF
# Allow checking the capabilities of our own token. This is used to validate the
# token upon startup.
path "sys/capabilities-self" {
  capabilities = ["update"]
}

# Allow our own token to be renewed.
path "auth/token/renew-self" {
  capabilities = ["update"]
}

# Allow to create and update records for the certificates
path "kv/data/ssl" {
  capabilities = ["update", "create"]
}
EOF
```

Write a policy called `letsencrypt` using the `letsencrypt-policy.hcl` file.
```shell
vault policy write letsencrypt letsencrypt-policy.hcl
```

## Create Vault Token
For the application to function correctly, you will need to create a
[Periodic service token](https://developer.hashicorp.com/vault/tutorials/tokens/tokens#periodic-service-tokens).
For example with Vault policy `letsencrypt` and period `72h`
```shell
vault token create -policy letsencrypt -period 72h
```

## Copy executable
Put `letshide` to `/usr/local/bin` and set executable bit
```shell
sudo cp letshide /usr/local/bin
sudo chmod a+x /usr/local/bin/letshide
```
 
## Create environment variables file `/etc/default/letshide`
You could copy example from [etc/default/letshide](etc/default/letshide).
- `VAULT_ADDR` - Address of the Vault server. The default is https://127.0.0.1:8200.
- `VAULT_TOKEN` - The Vault Token. Required.
- `VAULT_MOUNT` - The Vault mount point. The default is `kv`. 
- `VAULT_PATH` - The path to store certificates. The default is `ssl` in root of mount point.
- `CERT_DIR` - The path to `live` directory of Let's Encrypt. Ex.: `/etc/letsencrypt/live/example.com`. Required.

## Create SystemD unit file
You could copy example from [etc/systemd/system/letshide.service](etc/systemd/system/letshide.service)

## Create SystemD timer file
You could copy example from [etc/systemd/system/letshide.timer](etc/systemd/system/letshide.timer)

## Enable and start timer
```shell
systemctl daemon-reload
systemctl enable letshide.timer
systemctl start letshide.timer
```
