---
tags: ["vault"]
---

# vault基本使用及部署

```shell
docker run -d --restart=always  \
  --name vault-dev \
  -p 8200:8200 \
  -e VAULT_DEV_ROOT_TOKEN_ID=root \
  -e VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200 \
  hashicorp/vault:1.15
```

## 检测服务

```shell
curl http://127.0.0.1:8200/v1/sys/health
```

## 存secret

```shell
curl 'http://127.0.0.1:8200/v1/secret/data/demo1' \
  -H 'X-Vault-Token: root' \
  --data-raw '{"data":{"111":"222"}}'
```

## 取secret

```shell
curl 'http://127.0.0.1:8200/v1/secret/data/demo1' \
    -H 'X-Vault-Token: root'
```
