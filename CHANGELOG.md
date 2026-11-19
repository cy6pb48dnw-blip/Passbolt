# Changelog

## 2026-11-19

### WireGuard

- ajout de la VM Passbolt comme peer `028 - Ergo3 Passbolt`;
- IP WireGuard : `10.203.0.37/32`;
- handshake validé ;
- ping bidirectionnel validé ;
- service `wg-quick@wg0` configuré pour être persistant.

### TLS

Migration du renouvellement du certificat `vault.ergo3.ch` de HTTP-01 vers DNS-01.

Motif :

- suppression du NAT public 80/443 ;
- maintien du certificat public Let’s Encrypt ;
- renouvellement autonome depuis Passbolt.

Ajout :

- `/etc/vitalink/ovh-ergo3.conf`
- `/usr/local/sbin/passbolt-ovh-dns`
- `/usr/local/sbin/passbolt-certbot-auth`
- `/usr/local/sbin/passbolt-certbot-cleanup`
- `/etc/letsencrypt/renewal-hooks/deploy/passbolt-nginx`

Tests validés :

- création TXT OVH ;
- propagation DNS autoritative ;
- suppression TXT ;
- `certbot --dry-run` ;
- `certbot renew --dry-run --run-deploy-hooks` ;
- Nginx actif ;
- `certbot.timer` actif et enabled.

Certificat actif au moment du changement :

```text
vault.ergo3.ch
ECDSA
expiration: 2026-12-22 08:57:37 UTC
```

### Maintenance système

- Passbolt : 5.17.0-1
- PHP : 8.4.27-1~deb13u1
- kernel : 6.12.113+deb13-amd64
- healthcheck Passbolt final validé.

### DNS local

Ajout / validation de la résolution locale de `vault.ergo3.ch` pour éviter le hairpin vers UniFi lors des healthchecks locaux.

## 2026-10 / état initial documenté

- déploiement Passbolt CE natif sur Debian ;
- Nginx + PHP-FPM + MariaDB ;
- base `passboltdb`;
- MFA TOTP activé ;
- SMTP GMX configuré ;
- certificat Let’s Encrypt initialement obtenu avec exposition HTTP-01.
