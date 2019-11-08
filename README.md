# GitOps examples

## Encrypt & Decrypt

Using sops

```
sops --encrypt --encrypted-suffix stringData -i --pgp <key id> db/config.yaml
```

```
sops --decrypt --encrypted-suffix stringData --pgp <key id> db/config.yaml
```

