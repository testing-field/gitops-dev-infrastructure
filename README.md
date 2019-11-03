# GitOps examples

## Encrypt & Decrypt

Using sops

```
sops --encrypt --encrypted-suffix stringData -i db/config.yaml
```

```
sops --decrypt --encrypted-suffix stringData db/config.yaml
```

