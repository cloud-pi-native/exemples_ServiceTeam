# Secrets Vault

Ce chart présente l'utilisation du kind VaultStaticSecret afin de retrouver un secret sur [vault](https://www.vaultproject.io/) via [VaultSecretOperator](https://developer.hashicorp.com/vault/tutorials/kubernetes/vault-secrets-operator)

## Hiérarchie des secrets

Pour cet exemple, il faut créer des secrets pour avoir cette structure:

```txt
mi-demovault/
├─ dev/
│  ├─ dockerconfig: username=monsuer / password=monmotdepasse / registry=my-registry.com
│  ├─ db: username=app / password=azerty1234
├─ foo: bar=baz
```

```shell
vault kv put -mount=mi-demovault dev/db username=app password=azerty1234
vault kv put -mount=mi-demovault dev/dockerconfig username=monsuer password=monmotdepasse registry=my-registry.com
vault kv put -mount=mi-demovault foo bar=baz
```

## Récupération d'un secret

Pour chaque projet créé dans la console, des objets de type `VaultConnection` et `VaultAuth` sont automatiquement créés afin de pouvoir authentifié le projet auprès du vault.

Pour récupérer un secret, créer le manifest suivant:

- Organisation: mi
- Nom du projet: demovault

```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: db-secrets
spec:
  vaultAuthRef: vault-auth # Nom du VaultAuth, toujours vault-auth dans notre cas
  mount: mi-demovault # Nom du kv dans vault (<organisation>-<nom du projet>)
  path: db # Chemin vers le secret
  type: kv-v2 # Type du store de secret (toujours kv-v2 dans notre cas)
  refreshAfter: 10m # L'opérateur est capable de détecter les changements du secret sur vault pour les rafraichir dans kubernetes
  destination: # Nom du secret kubernetes à créer
    name: db-secrets
    create: true
    labels:
```

### Templetisation d'un secret

L'opérateur VSO permet de templatiser un secret.

A partir d'un username/password récupéré dans vault et d'une annotation sur le manifest de type VaultStaticSecret, il est possible de construire une chaîne de caractères pour se connecter à une base de données postgresql:

```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: db-secrets
  annotations:
    myapp.config/postgres-host: postgres-postgresql.postgres.svc.cluster.local:5432
spec:
  [...]
  destination:
    [...]
    transformation:
      excludes:
      - .*
      templates:
        url:
          text: |
            {{`{{- $host := get .Annotations "myapp.config/postgres-host" -}}
            {{- printf "postgresql://%s:%s@%s/postgres?sslmode=disable" (get .Secrets "username") (get .Secrets "password") $host -}}`}}
```

---

A partir d'un secret vault (contenant les clés username, password et registry), construire un secret de type dockerconfigjson:

```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: docker-secret
spec:
  vaultAuthRef: vault-auth
  mount: {{ .Values.vaultSecretMount }}
  path: dev/dockerconfig
  type: {{ .Values.vaultSecretType }}
  refreshAfter: {{ .Values.vaultRefreshAfter }}
  destination:
    name: demo-pull-secret
    create: true
    type: kubernetes.io/dockerconfigjson
    transformation:
      excludeRaw: true
      excludes:
        - .*
      templates: 
        ".dockerconfigjson":
          text: |
            {{`
              {{- $registry := get .Secrets "registry" -}}
              {{- $username := get .Secrets "username" -}}
              {{- $password := get .Secrets "password" -}}
              {{- $config := (list $username ":" $password | join "" | b64enc)  -}}
              {{- dict "username" $username
                        "password" $password
                        "auths" (dict $registry $config) 
                        | mustToJson -}}
            `}}
```

Il est ainsi possible d'utiliser ce secret dans un déploiement:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
  labels:
    app: busybox
spec:
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      [...]
      imagePullSecrets:
      - name: demo-pull-secret
```

### Redémarrer une instance après la création/modification d'un secret

L'opérateur est capable de redémarrer votre instance (Deployment, DaemonSet, StatefulSet) dès qu'il détecte un changement dans le secret:

```yaml
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: db-secrets
spec:
  [...]
  rolloutRestartTargets:
  - kind: Deployment # Type d'objet
    name: busybox # Nom de l'objet

