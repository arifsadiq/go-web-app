#### Step 1: List the secrets

```bash
kubectl get secret -n argocd
```

#### Step 2: To get the password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd
```

Copy the password and it is encoded with base64. We have to decode it.

#### Step 3: To decode the password

```bash
echo <password> | base64 --decode
```

replace <password> with your encoded password