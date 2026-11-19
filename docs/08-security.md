# Sécurité

## Principe général

Passbolt contient des secrets critiques. L’architecture privilégie :

- accès réseau privé / VPN ;
- absence de NAT public entrant pour le service ;
- HTTPS obligatoire ;
- MFA TOTP ;
- administration via WireGuard ;
- renouvellement TLS via DNS-01 ;
- séparation stricte entre documentation et secrets.

## Exposition réseau

Le serveur est joignable en interne sur :

```text
192.168.6.212:443
```

L’accès de management WireGuard utilise :

```text
10.203.0.37/32
```

Le renouvellement TLS ne nécessite aucun port entrant public.

## Secrets interdits dans Git

Ne jamais commiter :

- clés privées WireGuard ;
- fichier `/etc/vitalink/ovh-ergo3.conf` réel ;
- application secret / consumer key OVH ;
- mots de passe DB ;
- mots de passe SMTP / app passwords ;
- recovery kits ;
- clés privées utilisateur ;
- exports de mots de passe ;
- dumps MariaDB non chiffrés ;
- clés GPG serveur privées.

## Permissions critiques

OVH :

```bash
chown root:root /etc/vitalink/ovh-ergo3.conf
chmod 600 /etc/vitalink/ovh-ergo3.conf
```

WireGuard :

```bash
chmod 600 /etc/wireguard/wg0.conf
chmod 600 /etc/wireguard/privatekey
```

Scripts TLS :

```bash
chown root:root /usr/local/sbin/passbolt-*
chmod 750 /usr/local/sbin/passbolt-*
```

## API OVH

Le script `passbolt-ovh-dns` est codé pour refuser les domaines autres que `vault.ergo3.ch`.

En complément, les droits réels du consumer key OVH doivent être limités au strict nécessaire pour :

- lire les TXT de la zone `ergo3.ch` ;
- créer un TXT challenge ;
- supprimer ce TXT ;
- rafraîchir la zone.

Éviter un token DNS global si OVH permet de limiter les ACL API au périmètre requis.

## MFA

TOTP est activé.

Exigences :

- conserver HTTPS fonctionnel ;
- synchroniser correctement l’heure du serveur ;
- protéger le recovery kit utilisateur ;
- ne jamais considérer le rôle administrateur comme un moyen de récupérer la clé privée CE.

## Recovery

Community Edition ne fournit pas l’admin-assisted recovery disponible en Pro/Cloud.

Chaque utilisateur doit donc disposer d’un recovery kit correctement protégé.

Source officielle :
https://www.passbolt.com/docs/user/quickstart/browser/admin-assisted-recovery/

## Nginx

Avant reload :

```bash
nginx -t
```

Le hook de déploiement Certbot respecte cette règle avant de recharger Nginx.

## Mises à jour

Maintenir à jour :

- Debian ;
- kernel ;
- Passbolt ;
- PHP ;
- MariaDB ;
- Certbot.

Après patching :

```bash
sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"
```

## Logs utiles

```text
/var/log/letsencrypt/letsencrypt.log
/var/log/nginx/
/var/log/passbolt/
```

Ne jamais copier des logs complets dans un dépôt public sans vérifier qu’ils ne contiennent pas d’adresses, identifiants ou informations sensibles.

## Dépôt GitHub public

Ce repository est public. Les exemples doivent toujours utiliser des placeholders pour les secrets.

La publication d’adresses privées et de clés publiques WireGuard n’accorde pas l’accès au réseau, mais toute information d’architecture doit rester limitée au strict besoin documentaire.
