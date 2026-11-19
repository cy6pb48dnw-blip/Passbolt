# Maintenance et mises à jour

## État de référence — 19 novembre 2026

| Composant | Version |
|---|---|
| Passbolt CE | `5.17.0-1` |
| PHP | `8.4.27-1~deb13u1` |
| Kernel | `6.12.113+deb13-amd64` |
| OS | Debian 13 Trixie |

Lors de la maintenance de novembre 2026 :

- Passbolt a été mis à jour de 5.16 vers 5.17 ;
- PHP a été mis à jour de 8.4.24 vers 8.4.27 ;
- le kernel Debian a été mis à jour ;
- le healthcheck Passbolt final était fonctionnel ;
- Nginx, PHP-FPM et MariaDB étaient actifs.

## Avant une maintenance importante

1. Vérifier qu’une sauvegarde récente et restaurable existe.
2. Si disponible dans l’hyperviseur, prendre un snapshot temporaire cohérent.
3. Vérifier l’espace disque.
4. Vérifier l’état des services.
5. Noter les versions actuelles.

```bash
df -h
systemctl --failed
systemctl is-active nginx php8.4-fpm mariadb
dpkg -l | grep passbolt
php -v
uname -r
certbot certificates
wg show wg0
```

## Mise à jour Debian / Passbolt

```bash
apt update
apt list --upgradable
apt upgrade
```

Pour une opération majeure, lire les notes de version Passbolt et Debian avant d’utiliser `full-upgrade`.

Contrôler après mise à jour :

```bash
dpkg -l | grep passbolt
php -v
uname -r

nginx -t
systemctl is-active nginx
systemctl is-active php8.4-fpm
systemctl is-active mariadb
```

## Healthcheck Passbolt

```bash
sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"
```

Le healthcheck doit être considéré comme une étape obligatoire après une mise à jour Passbolt ou une modification importante de PHP, MariaDB, GPG ou TLS.

## Reboot

Si un nouveau kernel est installé :

```bash
reboot
```

Après redémarrage :

```bash
uname -r
systemctl --failed
systemctl is-active nginx php8.4-fpm mariadb
systemctl is-active wg-quick@wg0
systemctl is-active certbot.timer

wg show wg0
curl -Ik https://vault.ergo3.ch/
```

## Vérification TLS après maintenance

```bash
certbot certificates

certbot renew \
  --cert-name vault.ergo3.ch \
  --dry-run \
  --run-deploy-hooks
```

Le dry-run ne doit pas laisser de TXT résiduel :

```bash
NS="$(dig +short NS ergo3.ch | head -1)"
dig +short TXT _acme-challenge.vault.ergo3.ch @"$NS"
```

## Vérification WireGuard

```bash
ip -br a show wg0
wg show wg0
ping -c 3 10.203.0.1
```

La configuration doit rester persistante via :

```text
/etc/wireguard/wg0.conf
systemctl enable wg-quick@wg0
```

## Principes

- ne jamais considérer une modification réseau comme terminée tant qu’elle n’est pas persistante ;
- ne jamais supprimer un ancien backup avant validation du nouveau ;
- ne jamais modifier les secrets directement depuis Git ;
- tester Nginx avant tout reload ;
- exécuter le healthcheck Passbolt après chaque changement important.

## Sources officielles

- Passbolt maintenance / backup: https://www.passbolt.com/docs/hosting/maintenance/backup/
- Installation Debian: https://www.passbolt.com/docs/hosting/install/ce/debian/
