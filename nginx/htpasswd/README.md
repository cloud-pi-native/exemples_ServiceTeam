# Ajout d'un basic auth sur un ingress

## Etape 1 création du htpasswd

Récupérer ou créer un fichier htpasswd :

```
$ htpasswd -c auth foo
New password:
Re-type new password:
Adding password for user foo
```

Ceci crée un fichier auth dont voici un exemple de contenu :
```
more auth
foo:$apr1$.rVNB7kq$eI1Ve4ShnwiDDs7bZo6OW.
```
Et la version encodé en base64 :

```
cat auth | base64
Zm9vOiRhcHIxJC5yVk5CN2txJGVJMVZlNFNobndpRERzN2JabzZPVy4K
```

## Création du secret Kubernetes

Pour cela utilisez SOPS ou Vault. Pour l'exemple, le contenu est laissé en clair ce qui n'est pas une bonne pratique mais facilite la démonstration.

voir le contenu du templates/secret.yaml

```yaml
apiVersion: v1
data:
  auth: Zm9vOiRhcHIxJC5yVk5CN2txJGVJMVZlNFNobndpRERzN2JabzZPVy4K
kind: Secret
metadata:
  name: basic-auth
type: Opaque
```

## Configuration de l'ingress

Sur l'ingress ajouter les annotations suivantes :

```yaml
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
```

Accédez à l'url de l'ingress avec le mot de passe correspondant.