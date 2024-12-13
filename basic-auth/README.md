# Basic Auth

Ce chart permet de déployer une basic-auth afin d'ajouter une authentification à une application web.  

Il y a une dépendance au chart [haproxy](https://github.com/bitnami/charts/blob/main/bitnami/haproxy/README.md).

### Utilisation

Le fichier `values-example.yaml` décrit un exemple d'utilisation du chart.

Pour la gestion de l'authentification, vous pouvez créer un couple utilisateur/mot de passe, comme indiqué dans l'exemple ci-dessous.

Nous vous conseillons d'utiliser la [documentation](https://www.haproxy.com/documentation/haproxy-configuration-tutorials/authentication/basic-authentication/#hash-passwords-in-the-userlist) afin de générer le hash pour le mot de passe. 

```yaml
basicAuth:
  - user: admin
    password: $5$s6Subz0X7FSX2zON$r94OtF6gOfWlGmySwvn3pDFIAHbIpe6mWneueqtBOm/
```

Ce chart prend également en compte la création de secrets récupérés depuis le [Vault](https://cloud-pi-native.fr/guide/secrets-vault#recuperation-d-un-secret-sur-kubernetes).
