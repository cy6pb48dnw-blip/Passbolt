# Architecture

## Objectif

Cette instance héberge le coffre de mots de passe Ergo3 sur **Passbolt Community Edition**. L’objectif est de conserver le service accessible uniquement depuis les réseaux autorisés et les accès VPN, tout en permettant un renouvellement TLS automatique sans exposition publique du serveur web.

## Composants

| Composant | Rôle |
|---|---|
| Debian 13 | OS de la VM |
| Passbolt CE | coffre de secrets |
| Nginx | terminaison HTTPS |
| PHP 8.4 FPM | runtime applicatif |
| MariaDB | base de données Passbolt |
| Certbot | émission et renouvellement TLS |
| API OVH | écriture/suppression des TXT DNS-01 |
| WireGuard | administration sécurisée de la VM |

## Adressage

- FQDN : `vault.ergo3.ch`
- IP LAN de la VM : `192.168.6.212`
- IP WireGuard de la VM : `10.203.0.37/32`
- Hub WireGuard : `10.203.0.1`

Le DNS interne résout `vault.ergo3.ch` vers `192.168.6.212`. L’accès à la VM est prévu depuis le LAN ou via les VPN autorisés.

## Flux applicatif

```text
Client
  |
  | HTTPS / 443
  v
vault.ergo3.ch
  |
  v
Nginx
  |
  v
Passbolt CE
  |
  v
MariaDB
```

## Flux d’administration

```text
Administrateur
  |
  v
Hub WireGuard
  |
  v
10.203.0.37/32
  |
  +--> SSH / maintenance Passbolt
```

WireGuard est un **plan de management**. Il n’est plus nécessaire au mécanisme TLS lui-même.

## Flux de renouvellement TLS

```text
certbot.timer
     |
     v
certbot renew
     |
     v
passbolt-certbot-auth
     |
     v
passbolt-ovh-dns
     |
     v
API OVH
     |
     +--> crée _acme-challenge.vault.ergo3.ch TXT
     |
     v
Let's Encrypt valide DNS-01
     |
     v
passbolt-certbot-cleanup
     |
     +--> supprime le TXT temporaire
     |
     v
nouveau certificat
     |
     v
deploy hook -> nginx -t -> reload nginx
```

Aucun NAT entrant 80/443 n’est requis pour le renouvellement.

## Principe de split DNS

Le nom `vault.ergo3.ch` est utilisé partout, mais il peut être résolu vers l’adresse privée `192.168.6.212` à l’intérieur des réseaux autorisés.

Le challenge DNS-01 ne dépend pas de l’adresse A du serveur. Let’s Encrypt vérifie uniquement le TXT public sous :

```text
_acme-challenge.vault.ergo3.ch
```

## Données sensibles

Ne jamais stocker dans ce dépôt :

- `/etc/vitalink/ovh-ergo3.conf`
- `/etc/wireguard/privatekey`
- clé privée GPG d’un utilisateur
- mots de passe MariaDB
- secret SMTP / mot de passe applicatif
- exports de secrets Passbolt
- recovery kits utilisateurs

## Sources officielles

- Passbolt Debian CE: https://www.passbolt.com/docs/hosting/install/ce/debian/
- Passbolt TLS manuel: https://www.passbolt.com/docs/hosting/tls/ce/debian-manual/
- Let’s Encrypt DNS-01: https://letsencrypt.org/docs/challenge-types/
