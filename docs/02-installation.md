# Installation et composants

## VM

État de référence :

- Debian 13 (Trixie), installation minimale
- 2 vCPU
- 4 Go RAM
- disque 40 Go
- hostname : `passbolt`
- IP LAN : `192.168.6.212`

Les ressources sont supérieures au minimum officiel Passbolt pour une petite instance.

## Installation Passbolt

L’instance utilise le paquet Debian officiel :

```bash
apt install passbolt-ce-server
```

Paquet installé :

```text
passbolt-ce-server
```

Version de référence au 19 novembre 2026 :

```text
5.17.0-1
```

## Services

Services principaux :

```bash
systemctl status nginx
systemctl status php8.4-fpm
systemctl status mariadb
```

État attendu : `active (running)`.

## PHP

Version de référence :

```text
PHP 8.4.27-1~deb13u1
```

Cette version a remplacé PHP 8.4.24 lors de la maintenance de novembre 2026.

## Kernel

Version de référence :

```text
6.12.113+deb13-amd64
```

## MariaDB

Base applicative :

```text
database: passboltdb
user:     passboltadmin
```

Le mot de passe de la base n’est pas stocké dans Git.

La configuration applicative principale est sous :

```text
/etc/passbolt/passbolt.php
```

## Nginx / HTTPS

Le service est publié localement en HTTPS sur :

```text
https://vault.ergo3.ch
```

Les certificats actifs sont sous :

```text
/etc/letsencrypt/live/vault.ergo3.ch/fullchain.pem
/etc/letsencrypt/live/vault.ergo3.ch/privkey.pem
```

Contrôle :

```bash
nginx -t
openssl x509 \
  -in /etc/letsencrypt/live/vault.ergo3.ch/fullchain.pem \
  -noout -subject -issuer -dates
```

## SMTP

L’instance utilise un relais SMTP GMX avec :

- serveur : `mail.gmx.net`
- port : `587`
- chiffrement : STARTTLS/TLS
- authentification par mot de passe applicatif

Le compte SMTP et son secret ne doivent pas être documentés dans un dépôt public.

## Healthcheck

Commande de référence :

```bash
sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"
```

Après la maintenance de novembre 2026, le healthcheck était entièrement fonctionnel, avec uniquement des avertissements non bloquants liés aux contrôles SMTP/MX/notifications.

## Résolution locale du FQDN

Pour éviter qu’un healthcheck local sorte vers le DNS public et rencontre l’équipement UniFi au lieu du vhost Nginx local, l’instance dispose d’une résolution locale :

```text
127.0.0.1 vault.ergo3.ch
```

Vérifier :

```bash
grep vault.ergo3.ch /etc/hosts
getent hosts vault.ergo3.ch
```

## Sources officielles

- Installation Debian 13 CE: https://www.passbolt.com/docs/hosting/install/ce/debian/
- Prérequis: https://www.passbolt.com/docs/hosting/requirements/
