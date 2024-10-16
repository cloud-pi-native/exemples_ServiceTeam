# CNPG PGADMIN

Exemple d'utilisation d'un cluster CNPG (Cloud Native PostGreSQL) ainsi que l'utilisation optionnel d'un pgadmin.

## Utilisation

Ce chart défini une base pour l'utilisation de CNPG, avec un script d'initialisation SQL est joué au démarrage de la base.

### Initialisation de la base via un backup CNPG

Il est aussi possible de repartir d'un backup généré par CNPG, un exemple est disponible [ici](exemples/cnpg/cluster-from-backup.yaml)

Pour plus de de documentation [https://cloudnative-pg.io/documentation/1.24/bootstrap/](https://cloudnative-pg.io/documentation/1.24/bootstrap/)

### Initilisation de la base via un backup stocké sur un S3

Ce chart comprend un script dataloader, permettant de récupérer un fichier sql depuis un S3 et de le restaurer dans la base voulue.

Attention, cela demande de construire l'image docker associée, voir les fichiers [ici](exemples/dataloader/)

### Rétention des WAL

Il est possible de paramétrer 2 volumes différents pour le stockage des données et celui des WAL:

```yaml
  storage:
    size: 8Gi
  walStorage:
    size: 5Gi
```

De plus il est bon de spécifier à postgresql la taille maximum du stockage des WAL (par défaut à -1 pour illimité):

```yaml
  postgres:
    parameters:
      max_slot_wal_keep_size: 5GB
```

Attention aux unités pour postgresql: B, kB, MB, GB, TB.
