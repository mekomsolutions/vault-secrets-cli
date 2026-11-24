# vault-secrets-cli

Resolves `vault(path#key)` references from an OCD3 instance definition into a Docker Compose `env_file` (typically under `/run/ocd3/<name>/secrets.env`).

## Setup

Example instance definitions live in the OCD3 GitOps repository. Clone it so the examples below can mount a local instance JSON:

## Usage

Pass Vault env vars into the container and mount the instance JSON plus an output directory:

```bash
export VAULT_ADDR=https://vault.example.com
export VAULT_AUTH_METHOD=token
export VAULT_TOKEN=hvs.your-token-here
```

```bash
docker run --rm \
  -e VAULT_ADDR \
  -e VAULT_AUTH_METHOD \
  -e VAULT_TOKEN \
  -v "$(pwd)/instances/dev/oz-c2c-dev.json:/instance.json:ro" \
  -v "$(pwd):/out" \
  mekomsolutions/vault-secrets-cli \
  resolve --instance /instance.json --output /out/secrets.env
```

### AppRole

```bash
export VAULT_ADDR=https://vault.example.com
export VAULT_AUTH_METHOD=approle
export VAULT_ROLE_ID=...
export VAULT_SECRET_ID=...
# optional: export VAULT_APPROLE_PATH=approle
```

```bash
docker run --rm \
  -e VAULT_ADDR \
  -e VAULT_AUTH_METHOD \
  -e VAULT_ROLE_ID \
  -e VAULT_SECRET_ID \
  -e VAULT_APPROLE_PATH \
  -v "$(pwd)/instances/dev/oz-c2c-dev.json:/instance.json:ro" \
  -v "$(pwd)/out:/out" \
  mekomsolutions/vault-secrets-cli \
  resolve --instance /instance.json --output /out/secrets.env
```

### Inline instance JSON

```bash
docker run --rm \
  -e VAULT_ADDR \
  -e VAULT_AUTH_METHOD \
  -e VAULT_TOKEN \
  -v "$(pwd)/out:/out" \
  mekomsolutions/vault-secrets-cli \
  resolve --instance-json '{"name":"demo","envVars":{"DB_PASSWORD":"vault(secret/c2c/dev#db_password)"}}' \
  --output /out/secrets.env
```

## Build

```bash
# Run tests locally (optional)
npm install
npm test

# Build the image
docker build -t mekomsolutions/vault-secrets-cli .
```

Multi-arch example:

```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t mekomsolutions/vault-secrets-cli:latest \
  --push .
```
