# Dépannage

## 1. Healthcheck SSL échoue depuis la VM

### Symptôme observé

Le FQDN public `vault.ergo3.ch` résolvait vers une adresse publique et, depuis la VM, le trafic revenait sur l’équipement UniFi. Le certificat présenté était alors un certificat auto-signé `unifi.local` au lieu du certificat Let’s Encrypt Passbolt.

### Correction appliquée

Résolution locale :

```text
127.0.0.1 vault.ergo3.ch
```

Contrôler :

```bash
getent hosts vault.ergo3.ch

openssl s_client \
  -connect vault.ergo3.ch:443 \
  -servername vault.ergo3.ch \
  </dev/null 2>/dev/null |
openssl x509 -noout -subject -issuer -dates
```

Le certificat doit être celui de `vault.ergo3.ch`.

## 2. Certificat ne se renouvelle plus après suppression du NAT

### Cause

L’ancien mécanisme HTTP-01 dépendait d’un accès public entrant.

### Solution actuelle

DNS-01 via API OVH.

Test :

```bash
certbot renew \
  --cert-name vault.ergo3.ch \
  --dry-run \
  --run-deploy-hooks
```

## 3. Vérifier le TXT DNS-01

```bash
NS="$(dig +short NS ergo3.ch | head -1)"

dig +short TXT \
  _acme-challenge.vault.ergo3.ch \
  @"$NS"
```

En dehors d’un challenge, la réponse doit normalement être vide.

## 4. API OVH

Test contrôlé :

```bash
TEST="vitalink-passbolt-test-$(date +%s)"

/usr/local/sbin/passbolt-ovh-dns \
  txt-presenter vault.ergo3.ch "$TEST"

NS="$(dig +short NS ergo3.ch | head -1)"
dig +short TXT _acme-challenge.vault.ergo3.ch @"$NS"

/usr/local/sbin/passbolt-ovh-dns \
  txt-retirer vault.ergo3.ch "$TEST"
```

Ne pas oublier la suppression du token de test.

## 5. Nginx

```bash
nginx -t
systemctl status nginx --no-pager
journalctl -u nginx -n 100 --no-pager
```

Certificat configuré :

```bash
nginx -T 2>/dev/null |
grep -nE 'server_name|ssl_certificate|ssl_certificate_key'
```

## 6. Certbot

```bash
certbot certificates
systemctl status certbot.timer --no-pager
journalctl -u certbot.service -n 100 --no-pager
tail -100 /var/log/letsencrypt/letsencrypt.log
```

Configuration de renewal :

```bash
grep -E \
  '^(authenticator|pref_challs|manual_auth_hook|manual_cleanup_hook)' \
  /etc/letsencrypt/renewal/vault.ergo3.ch.conf
```

Attendu :

```text
authenticator = manual
pref_challs = dns-01,
manual_auth_hook = /usr/local/sbin/passbolt-certbot-auth
manual_cleanup_hook = /usr/local/sbin/passbolt-certbot-cleanup
```

## 7. WireGuard

```bash
ip -br a show wg0
wg show wg0
ping -c 3 10.203.0.1
```

Attendu :

- adresse `10.203.0.37/32`;
- handshake récent ;
- transfert RX/TX ;
- ping vers `10.203.0.1`.

Service :

```bash
systemctl status wg-quick@wg0 --no-pager
```

## 8. Passbolt

```bash
sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"
```

Services :

```bash
systemctl is-active nginx php8.4-fpm mariadb
```

## 9. Après reboot

```bash
systemctl --failed
systemctl is-active nginx php8.4-fpm mariadb
systemctl is-active wg-quick@wg0
systemctl is-active certbot.timer
wg show wg0
certbot certificates
```

## 10. Deploy hook affiche "error output" alors que nginx -t réussit

`nginx -t` écrit ses messages de succès sur stderr. Certbot peut donc afficher un bloc "error output" même si le test est réussi.

Le hook actuel capture stdout/stderr et n’affiche le contenu que si `nginx -t` retourne une erreur réelle.
