1. Setting up the Vault Server

```bash
touch docker-compose.yml
mkdir -p volumes/{config,file,logs}
```

2. Populate the vault config vault.json. (As you can see the config is local, in the next couple of posts, I will show how to persist this config to Amazon S3)

```bash
cat > volumes/config/vault.json << EOF
{
  "backend": {
    "file": {
      "path": "/vault/file"
    }
  },
  "listener": {
    "tcp":{
      "address": "0.0.0.0:8200",
      "tls_disable": 1
    }
  },
  "ui": true
}
EOF
```

3. Populate the docker-compose.yml:

```bash
cat > docker-compose.yml << EOF
version: '2'
services:
  vault:
    image: vault
    container_name: vault
    ports:
      - "8200:8200"
    restart: always
    volumes:
      - ./volumes/logs:/vault/logs
      - ./volumes/file:/vault/file
      - ./volumes/config:/vault/config
    cap_add:
      - IPC_LOCK
    entrypoint: vault server -config=/vault/config/vault.json
EOF
```

4. Start the Vault Server:

```bash
docker-compose up -d
```

Interacting with the Vault CLI (vault cli required)

5. Set environment variables:

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
```

6. Initialize new vault cluster with 6 key shares:

```bash
vault operator init -key-shares=6 -key-threshold=3
```

7. In order to unseal the vault cluster, we need to supply it with 3 key shares:

```bash
vault operator unseal key1
vault operator unseal key2
vault operator unseal key3
```

8. Ensure the vault is unsealed:

```bash
vault status -format=json
```

9. Authenticate against the vault:

```bash
vault login rootpassword
```

10. Enable the secret kv engine:

```bash
vault secrets enable -version=1 -path=secret kv
```

[Source](https://blog.ruanbekker.com/blog/2019/05/06/setup-hashicorp-vault-server-on-docker-and-cli-guide/)