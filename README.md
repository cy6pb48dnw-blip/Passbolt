# Passbolt Ergo3 — Documentation d’exploitation

Documentation technique de l’instance **Passbolt Community Edition** utilisée par Ergo3.

> **Sécurité du dépôt**
>
> Ce dépôt est public. Il ne doit contenir **aucun secret** : clé privée WireGuard, clé GPG privée utilisateur, mot de passe, token, clé API OVH, secret SMTP ou contenu exporté de Passbolt. Les chemins et procédures sont documentés, mais les valeurs sensibles restent uniquement sur le serveur.

## État de référence

| Élément | Valeur |
|---|---|
| Application | Passbolt Community Edition |
| FQDN | `vault.ergo3.ch` |
| OS | Debian 13 (Trixie) |
| Installation | paquet Debian `passbolt-ce-server` |
| Passbolt | 5.17.0-1 |
| PHP | 8.4.27-1~deb13u1 |
| Kernel | 6.12.113+deb13-amd64 |
| Web | Nginx |
| PHP runtime | php8.4-fpm |
| Base de données | MariaDB |
| Base Passbolt | `passboltdb` |
| Utilisateur DB | `passboltadmin` |
| IP LAN | `192.168.6.212` |
| IP WireGuard | `10.203.0.37/32` |
| TLS | Let’s Encrypt DNS-01 via API OVH |
| MFA | TOTP activé |

## Architecture actuelle

```text
Utilisateurs internes / VPN
          |
          v
   vault.ergo3.ch
          |
          v
   192.168.6.212
          |
     Nginx :443
          |
      Passbolt CE
          |
       MariaDB

Administration
     |
     v
WireGuard 10.203.0.37/32

Renouvellement TLS
certbot.timer
     |
     v
DNS-01 -> API OVH
     |
     +--> TXT temporaire _acme-challenge.vault.ergo3.ch
     |
     +--> Let's Encrypt
     |
     +--> suppression TXT
     |
     +--> reload Nginx
```

Passbolt n’a plus besoin d’un NAT public entrant sur les ports 80/443 pour renouveler son certificat.

## Documentation

- [Architecture](docs/01-architecture.md)
- [Installation et composants](docs/02-installation.md)
- [Réseau et WireGuard](docs/03-network-wireguard.md)
- [TLS Let’s Encrypt / OVH DNS-01](docs/04-tls-ovh-dns01.md)
- [Exploitation Passbolt](docs/05-passbolt-operations.md)
- [Maintenance et mises à jour](docs/06-maintenance-updates.md)
- [Sauvegarde et restauration](docs/07-backup-restore.md)
- [Sécurité](docs/08-security.md)
- [Dépannage](docs/09-troubleshooting.md)
- [Historique](CHANGELOG.md)

## Scripts de référence

- [passbolt-ovh-dns](scripts/passbolt-ovh-dns)
- [passbolt-certbot-auth](scripts/passbolt-certbot-auth)
- [passbolt-certbot-cleanup](scripts/passbolt-certbot-cleanup)
- [passbolt-nginx deploy hook](scripts/passbolt-nginx)
- [Exemple de configuration OVH](examples/ovh-ergo3.conf.example)

Les scripts présents ici sont des **copies de référence sans secrets**. Les scripts réellement utilisés sont installés dans `/usr/local/sbin/` et le hook Nginx dans `/etc/letsencrypt/renewal-hooks/deploy/`.

## Contrôles rapides

```bash
systemctl is-active nginx php8.4-fpm mariadb
systemctl is-enabled certbot.timer
wg show wg0
certbot certificates

sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"
```

## Sources officielles

- Passbolt — Debian CE: https://www.passbolt.com/docs/hosting/install/ce/debian/
- Passbolt — TLS manuel: https://www.passbolt.com/docs/hosting/tls/ce/debian-manual/
- Passbolt — Backup: https://www.passbolt.com/docs/hosting/maintenance/backup/from-packages/
- Passbolt — TOTP: https://www.passbolt.com/docs/admin/authentication/mfa/totp/
- Let’s Encrypt — DNS-01: https://letsencrypt.org/docs/challenge-types/
- Certbot — utilisation et hooks: https://eff-certbot.readthedocs.io/en/stable/using.html
