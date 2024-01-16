# SOPS
Ce projet fournit un exemple d'utilisation de [SOPS](https://github.com/getsops/sops) pour chiffrer ses secrets et les injecter dans sa chart helm.

Cloud PI utilise l'algorithme [AGE](https://age-encryption.org/) pour chiffrer/déchiffrer les données sensibles et la CRD [SopsSecret](https://github.com/isindir/sops-secrets-operator)

## Arborescence
Exemple d'arborescence permettant de gérer des secrets différents par plateforme/environnement.

```
.
├── Chart.yaml
├── README.md
├── charts
├── conf                         : dossier contenant les secrets en clairs (non versionnés)/chiffrés. Dossier non lu par helm
│   ├── decrypt.sh               : script pour déchiffrer un fichier chiffré
│   ├── encrypt.sh               : script pour chiffrer un/des fichiers en clairs
│   ├── mi                       : dossier représentant la plateforme mi
│   │   ├── preprod              : sous-dossier représentant l'environnement preprod
│   │   │   ├── secrets.yml.dec  : fichier secret en clair
│   │   │   └── secrets.yml.enc  : fichier chiffré
│   │   └── prod                 : sous-dossier représentant l'environnement prod
│   │       ├── secrets.yml.dec
│   │       └── secrets.yml.enc
│   └── ovh                      : dossier représentant la plateforme ovh
│       ├── dev                  : sous-dossier représentant l'environnement dev
│       │   ├── secrets.yml.dec
│       │   └── secrets.yml.enc
│       └── qualif               : sous-dossier représentant l'environnement qualif
│           ├── secrets.yml.dec
│           └── secrets.yml.enc
├── templates                    : dossier contenant les différents fichiers yaml que helm va utiliser
│   ├── deployment.yml           : exemple d'un déploiement utilisant les secrets créés via SopsSecret
│   └── secrets.yml              : fichier permettant d'injecter les bons secrets selon la plateforme et l'environnement    
├── values-mi-preprod.yaml       : surchage du values pour la plateforme mi et l'environnement preprod
├── values-mi-prod.yaml          : surchage du values pour la plateforme mi et l'environnement prod
├── values-ovh-dev.yaml          : surchage du values pour la plateforme ovh et l'environnement dev
├── values-ovh-qualif.yaml       : surchage du values pour la plateforme ovh et l'environnement qualif
└── values.yaml                  : valeur par défaut pour le chart helm
```

## SopsSecret
Pour pouvoir utiliser SOPS avec kubernetes, il faut définir un `SopsSecret`.

Ce `SopsSecret` sera responsable de créer les différents `Secret` (il est possible de définir plusieurs `Secret` par `SopsSecret`)

Exemple d'un fichier en clair, qui sera par la suite chiffré:
```yaml
apiVersion: isindir.github.com/v1alpha3
kind: SopsSecret
metadata:
  # Nom du SopsSecret, n'est pas réellement utile
  name: example-sopssecret-mi-preprod
spec:
  # Tableau contenant les différents secrets
  secretTemplates:
      # Nom du secret qui sera utilisé plus tard dans l'application
    - name: my-secret-name-1
       
      # Labels du secret
      labels:
        label1: value1

      # Annotations du secret
      annotations:
        key1: value1

      # Données du secret sous forme clé/valeur
      # la différence entre stringData et data, est que les valeurs dans le dictionnaire data doivent être encodées en base64
      stringData:
        data-name0: data-value0
      data:
        data-name1: ZGF0YS12YWx1ZTE=
    - name: jenkins-secret
      labels:
        "jenkins.io/credentials-type": "usernamePassword"
      annotations:
        "jenkins.io/credentials-description": "credentials from Kubernetes"
      stringData:
        username: myUsername
        password: 'Pa$$word'
    - name: some-token
      stringData:
        token: Wb4ziZdELkdUf6m6KtNd7iRjjQRvSeJno5meH4NAGHFmpqJyEsekZ2WjX232s4Gj
    - name: docker-login
      type: 'kubernetes.io/dockerconfigjson'
      stringData:
        .dockerconfigjson: '{"auths":{"index.docker.io":{"username":"imyuser","password":"mypass","email":"myuser@abc.com","auth":"aW15dXNlcjpteXBhc3M="}}}'
```

## Scripts
### Chiffrement
Ce projet propose un script permettant de chiffrer via l'utilitaire SOPS (à installer séparément) et l'algorithme AGE différents fichiers YAML.

Il contient déjà les différentes clés publiques utilisées sur Cloud PI.

Le script parcourt le dossier courant et sous-dossiers pour trouver des fichiers avec l'extension .dec

Les fichiers chiffrés sont sauvegardés avec l'extension .enc au lieu de .dec

Il est possible de passer un argument pour changer le dossier courant, voir de donner un chemin vers un fichier
```
.
├── encrypt.sh
├── mi
│   ├── preprod
│   │   ├── secrets.yml.dec
│   │   └── secrets.yml.enc
│   └── prod
│       ├── secrets.yml.dec
│       └── secrets.yml.enc
└── ovh
    ├── dev
    │   ├── secrets.yml.dec
    │   └── secrets.yml.enc
    └── qualif
        ├── secrets.yml.dec
        └── secrets.yml.enc
```

Ci-dessous un tableau reprenant les fichiers concernés selon l'argument fourni au script:
| Argument                | Fichiers concernés                                                                                   |
|-------------------------|------------------------------------------------------------------------------------------------------|
| <aucun>                 | mi/preprod/secrets.yml.dec, mi/prod/secrets.yml.dec, ovh/dev/secrets.yml, ovh/qualif/secrets.yml.dec |
| ovh                     | ovh/dev/secrets.yml, ovh/qualif/secrets.yml.dec                                                      |
| ovh/dev/secrets.yml.dec | ovh/dev/secrets.yml                                                                                  |


## Divers
### Clé publique
AGE est un algorithme à clé publique/clé privée.

Pour avoir les clés publiques, s'adresser à la ServiceTeam


### Git
Ne surtout pas oublier de ne pas versionner les fichiers en clair dans le dépôt git.

Ajouter cette ligne au `.gitignore`:
```
*.dec
```

