# TLS Let’s Encrypt via OVH DNS-01

## Décision

Le certificat de `vault.ergo3.ch` est renouvelé **directement par la VM Passbolt**.

L’ancien modèle HTTP-01 nécessitait que les ports 80/443 soient publiquement NATés vers Passbolt. Ce NAT a été supprimé.

Le nouveau modèle utilise DNS-01 et l’API OVH.

## Principe

À chaque challenge :

1. Certbot génère une valeur de validation.
2. `passbolt-certbot-auth` appelle `passbolt-ovh-dns`.
3. Le script crée temporairement :
   `_acme-challenge.vault.ergo3.ch TXT "<validation>"`.
4. Le hook attend que le TXT soit visible sur les serveurs DNS autoritatifs.
5. Let’s Encrypt valide le challenge DNS-01.
6. `passbolt-certbot-cleanup` supprime le TXT.
7. Certbot écrit le certificat renouvelé dans la lineage existante.
8. Le deploy hook valide Nginx puis le recharge.

## Fichiers

```text
/etc/vitalink/ovh-ergo3.conf
/usr/local/sbin/passbolt-ovh-dns
/usr/local/sbin/passbolt-certbot-auth
/usr/local/sbin/passbolt-certbot-cleanup
/etc/letsencrypt/renewal/vault.ergo3.ch.conf
/etc/letsencrypt/renewal-hooks/deploy/passbolt-nginx
```

Le fichier OVH doit être :

```bash
chown root:root /etc/vitalink/ovh-ergo3.conf
chmod 600 /etc/vitalink/ovh-ergo3.conf
```

## Format du fichier OVH

Ne jamais commiter les valeurs réelles.

```ini
OVH_ENDPOINT=https://eu.api.ovh.com/1.0
OVH_APPLICATION_KEY=<SECRET>
OVH_APPLICATION_SECRET=<SECRET>
OVH_CONSUMER_KEY=<SECRET>
```

## Protection intégrée

Le script dédié Passbolt refuse toute demande qui ne concerne pas :

```text
vault.ergo3.ch
```

La zone est fixée à :

```text
ergo3.ch
```

Le sous-domaine TXT est fixé à :

```text
_acme-challenge.vault
```

Ceci limite les erreurs opérationnelles du script même si les credentials OVH disposent de droits plus larges.

## Configuration Certbot enregistrée

Extrait attendu :

```ini
authenticator = manual
pref_challs = dns-01,
manual_auth_hook = /usr/local/sbin/passbolt-certbot-auth
manual_cleanup_hook = /usr/local/sbin/passbolt-certbot-cleanup
```

Contrôle :

```bash
grep -E '^(authenticator|pref_challs|manual_auth_hook|manual_cleanup_hook)' \
  /etc/letsencrypt/renewal/vault.ergo3.ch.conf
```

## Timer

```bash
systemctl status certbot.timer --no-pager
systemctl list-timers --all | grep certbot
```

État validé :

```text
enabled
active (waiting)
```

Le timer Debian lance le contrôle Certbot deux fois par jour. Certbot ne renouvelle le certificat que lorsqu’il entre dans sa fenêtre de renouvellement.

## Deploy hook Nginx

Fichier :

```text
/etc/letsencrypt/renewal-hooks/deploy/passbolt-nginx
```

Comportement :

- ne traite que `vault.ergo3.ch`;
- exécute `nginx -t`;
- si la configuration est valide, exécute `systemctl reload nginx`;
- en cas d’erreur Nginx, le hook retourne une erreur et n’effectue pas le reload.

## Test API OVH validé

Test manuel réalisé :

```bash
TEST="vitalink-passbolt-test-$(date +%s)"

/usr/local/sbin/passbolt-ovh-dns \
  txt-presenter vault.ergo3.ch "$TEST"

NS="$(dig +short NS ergo3.ch | head -1)"
dig +short TXT _acme-challenge.vault.ergo3.ch @"$NS"

/usr/local/sbin/passbolt-ovh-dns \
  txt-retirer vault.ergo3.ch "$TEST"
```

Résultat validé :

- création TXT : OK
- visibilité autoritative : OK
- suppression : OK
- aucun TXT résiduel : OK

## Test Certbot complet validé

```bash
certbot renew \
  --cert-name vault.ergo3.ch \
  --dry-run \
  --run-deploy-hooks
```

Résultat le 19 novembre 2026 :

```text
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/vault.ergo3.ch/fullchain.pem (success)
```

Le hook d’authentification a créé le TXT, attendu sa propagation, puis le hook cleanup l’a supprimé.

## Certificat actif au 19 novembre 2026

```text
Certificate Name: vault.ergo3.ch
Key Type: ECDSA
Domains: vault.ergo3.ch
Expiry: 2026-12-22 08:57:37 UTC
```

Chemins :

```text
/etc/letsencrypt/live/vault.ergo3.ch/fullchain.pem
/etc/letsencrypt/live/vault.ergo3.ch/privkey.pem
```

## Contrôles

```bash
certbot certificates
nginx -t
systemctl is-active nginx
systemctl is-enabled certbot.timer

NS="$(dig +short NS ergo3.ch | head -1)"
dig +short TXT _acme-challenge.vault.ergo3.ch @"$NS"
```

En fonctionnement normal, le dernier `dig` doit être vide en dehors d’un challenge en cours.

## Sources officielles

- Let’s Encrypt DNS-01: https://letsencrypt.org/docs/challenge-types/
- Certbot hooks: https://eff-certbot.readthedocs.io/en/stable/using.html
- Passbolt TLS manuel: https://www.passbolt.com/docs/hosting/tls/ce/debian-manual/
