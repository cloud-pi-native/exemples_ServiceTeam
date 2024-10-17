# Code d'exemple

Ce dépôt contient plusieurs exemples de chart helm utilisable sur la plateforme Cloud-Pi Native.

Résumé des exemples:

## Monitoring

- [prometheus_community_blackbox](monitoring/prometheus_community_blackbox/README.md): Chart Helm pour prometheus blackbox exporter (sonde d'URL).
- [prometheus_blackbox_custom](monitoring/prometheus_blackbox_custom/README.md): Installation et configuration d'un blackbox exporter (sonde d'URL) à la main.

## Logs

- [archive to S3](logs/archive_to_S3/README.md): Archiver ses logs sur un bucket S3 grâce à [vector](https://vector.dev/)
- [view with victoria logs](logs/view-with-victoria-logs/README.md): Voir ses logs grâce à [VictoriaLogs](https://docs.victoriametrics.com/victorialogs/)

## Secrets

- [SOPS](secrets/sops/README.md): Gestion des secrets dans un chart helm via [SOPS](https://github.com/isindir/sops-secrets-operator)

## Misc

- [pull_images_from_harbor](/misc/pull_images_from_harbor/README.md): Utilisation du secret registry-pull-secret et de la propriété imagePullSecrets sur un déploiement

## SSO

- [Keycloak](sso/keycloak/README.md): Déployer un fournisseur d'identité via Helm pour mettre à disposition un serice de [Single Sign-On](https://www.keycloak.org/getting-started/getting-started-openshift)

## APIM

- [Krakend](apim/krakend/README.md): Déployer une API Gateway afin de sécuriser les endpoints d'une API. Configurer l'authentification auprès d'un fournisseur d'identité et les règles d'accès aux différents endpoints (RBAC/ABAC) grâce à [Krakend](https://www.krakend.io/docs/overview/) 
