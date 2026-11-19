# Sauvegarde et restauration

## État actuel

La procédure de sauvegarde est documentée ici, mais **aucun job automatique de sauvegarde Passbolt n’a été confirmé dans l’état de référence de ce dépôt**.

Il faut donc considérer l’automatisation des backups et les tests de restauration comme une tâche d’exploitation à vérifier / mettre en place.

## Éléments indispensables

La documentation officielle Passbolt demande de sauvegarder au minimum :

1. la base de données ;
2. les clés GPG serveur ;
3. la configuration Passbolt ;
4. les éléments TLS si nécessaires à l’architecture.

Pour cette instance, il faut également préserver la configuration DNS-01 OVH.

## 1. Export de la base

Commande officielle recommandée pour une installation par paquet Debian :

```bash
sudo su -s /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt sql_export" \
  www-data
```

Conserver le fichier SQL produit dans un emplacement de backup chiffré hors de la VM.

## 2. Clés GPG serveur

Répertoire :

```text
/etc/passbolt/gpg/
```

Fichiers critiques :

```text
/etc/passbolt/gpg/serverkey.asc
/etc/passbolt/gpg/serverkey_private.asc
```

La clé privée serveur est sensible.

## 3. Configuration Passbolt

Fichier principal :

```text
/etc/passbolt/passbolt.php
```

Ce fichier peut contenir des informations sensibles, notamment les accès DB.

## 4. TLS / Certbot / OVH

Sauvegarder dans un backup chiffré :

```text
/etc/letsencrypt/
/etc/vitalink/ovh-ergo3.conf
/usr/local/sbin/passbolt-ovh-dns
/usr/local/sbin/passbolt-certbot-auth
/usr/local/sbin/passbolt-certbot-cleanup
/etc/letsencrypt/renewal-hooks/deploy/passbolt-nginx
```

Le fichier `/etc/vitalink/ovh-ergo3.conf` ne doit jamais être envoyé dans Git.

## 5. WireGuard

Sauvegarder de manière chiffrée :

```text
/etc/wireguard/wg0.conf
/etc/wireguard/privatekey
/etc/wireguard/publickey
```

La clé privée WireGuard ne doit pas être stockée dans ce dépôt.

## Exemple de structure de backup

```text
passbolt-backup-YYYYMMDD-HHMM/
├── database/
│   └── passbolt-backup.sql
├── passbolt/
│   ├── passbolt.php
│   └── gpg/
├── letsencrypt/
├── vitalink/
│   └── ovh-ergo3.conf
└── wireguard/
```

Chiffrer l’archive avant export hors du serveur.

## Restauration DB officielle

Passbolt fournit `mysql_import` :

```bash
sudo su -s /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt mysql_import \
   --dir /path/to/backups \
   --file passbolt-backup.sql" \
  www-data
```

## Restauration GPG

Après restauration des fichiers GPG, les clés doivent aussi être présentes dans le keyring du compte web. Suivre la procédure officielle Passbolt et valider ensuite :

```bash
sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck --gpg"
```

## Contrôle final après restauration

```bash
systemctl is-active nginx php8.4-fpm mariadb
nginx -t

sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"

certbot certificates
wg show wg0
```

Puis vérifier une connexion utilisateur avec un compte de test / administrateur disposant de son recovery kit.

## Politique recommandée

- backup régulier adapté au volume de changements ;
- copie hors de la VM ;
- stockage chiffré ;
- rétention multi-génération ;
- test périodique de restauration ;
- sauvegarde avant toute montée de version importante.

## Sources officielles

- Backup package Passbolt: https://www.passbolt.com/docs/hosting/maintenance/backup/from-packages/
- Backup général: https://www.passbolt.com/docs/hosting/maintenance/backup/
