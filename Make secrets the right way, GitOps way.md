# Make secrets the right way, GitOps way

1. Structure of the repository
Inside gitops repo, make directory which will hold the secret. If it is helm repo, then it should be **templates** directory. Secred already inside is the one we are creating in this process.

```
.
├── Chart.yaml
├── README.md
├── templates
│   └── es-sealed-secret.yaml
└── values.yaml
```

2. Secret file
In order to create sealed secred in Kubernetes cluster through GitOps procedure, we have to ensure we have plain secret content. In this scenario, it is **.json** key file in another repo:

```
mlp-backup-gcloud-token.json
```

3. Kubeseal installed and configured
Requirement for the whole process is of course having kubeseal installed and configure on your cluster
4. Cert file for sealing
In order to be able to seal the secret on the cluster, we have to retrieve the cert file which the cluster uses for it. Name of the file **master.key** can be whatever. For encrypting the file, leave only first tls.crt. Of course, don't leave it like that - it has to be base64 decoded:

```
kubectl get secret -n kubeseal -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml > master.key
```


5. Create kubernetes secret file, but do not apply on the cluster
If you want to output it to the file, do so:
```
kubectl create secret generic gcp-backup-key -n clearml-dev --from-file=gcs.client.default.credentials_file=./mlp-backup-gcloud-token.json --dry-run=client -o yaml >> plain_secret.yaml
```

6. Sealing the secret

```
kubeseal --cert <path_do_master_key_cert> --format=yaml -f <path_do_secret_filea_kojeg_zelis_sealat>
```

DONE

## Editing existing secrets

In case your service already uses secret, but it has to change for some reason, there is a way to do it. In this case, we are changing **key** inside sealed secret

1. Position inside **/templates** or other repo where secrets are placed
2. Make backup of existing secret, just in case:
```
cp kira-data-mongo-secret.yaml-sealed.yaml ../../../microblink/.
```

3. Get unsealed secret from cluster locally
```
k get secret kira-data-mongo-secret -n kira-data-dev -o yaml > kira-data-mongo-secret.yaml
```

4. Replace desired part of the secret
```
sed -i '' 's/mongodb-password/mongodb-passwords/g' kira-data-mongo-secret.yaml
```

5. Seal the secret again, thus making new secret
```
kubeseal --cert ~/smh/Microblink-repos/res-mle-ops/mlkube.crt --format=yaml -f kira-data-mongo-secret.yaml > kira-data-mongo-secret-sealed.yaml
```

6. Remove old secret and rename the new one so it has same name as the old one

DONE
