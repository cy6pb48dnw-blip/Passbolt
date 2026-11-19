# Scripts

Copies de référence des scripts installés sur la VM Passbolt.

## Installation

```bash
install -o root -g root -m 750 \
  scripts/passbolt-ovh-dns \
  /usr/local/sbin/passbolt-ovh-dns

install -o root -g root -m 750 \
  scripts/passbolt-certbot-auth \
  /usr/local/sbin/passbolt-certbot-auth

install -o root -g root -m 750 \
  scripts/passbolt-certbot-cleanup \
  /usr/local/sbin/passbolt-certbot-cleanup

install -o root -g root -m 750 \
  scripts/passbolt-nginx \
  /etc/letsencrypt/renewal-hooks/deploy/passbolt-nginx
```

Le fichier de credentials OVH doit être créé séparément :

```text
/etc/vitalink/ovh-ergo3.conf
```

Permissions obligatoires :

```bash
chown root:root /etc/vitalink/ovh-ergo3.conf
chmod 600 /etc/vitalink/ovh-ergo3.conf
```

Ne jamais copier le fichier réel dans ce dépôt.

## Test

```bash
certbot renew \
  --cert-name vault.ergo3.ch \
  --dry-run \
  --run-deploy-hooks
```
