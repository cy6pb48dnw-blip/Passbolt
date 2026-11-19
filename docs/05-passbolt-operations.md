# Exploitation Passbolt

## Edition

Instance actuelle :

```text
Passbolt Community Edition
```

Version de référence :

```text
5.17.0-1
```

## MFA

TOTP est activé au niveau de l’organisation.

Passbolt CE supporte TOTP. La configuration administrateur se trouve dans :

```text
Administration -> Multi Factor Authentication
```

Chaque utilisateur configure ensuite son authenticator dans son compte.

TLS/HTTPS est requis pour le MFA.

## Partage et droits

Le rôle administrateur ne donne pas automatiquement accès au contenu de tous les secrets.

Le modèle attendu est :

- créer les groupes appropriés ;
- partager explicitement les ressources avec les utilisateurs ou groupes ;
- attribuer les permissions nécessaires ;
- ne pas supposer qu’un administrateur peut lire tous les mots de passe.

## Recovery kits

Chaque utilisateur doit conserver son **recovery kit** dans un emplacement sécurisé et chiffré.

Ce recovery kit contient la clé privée utilisateur et est critique pour récupérer un compte sur un nouveau navigateur ou appareil.

## Limitation Community Edition

L’**admin-assisted account recovery** n’est pas disponible dans Passbolt Community Edition. Cette fonction est réservée aux éditions Pro et Cloud.

Conséquence opérationnelle :

- sans recovery kit / clé privée valide et sans session récupérable, l’administrateur CE ne peut pas recréer cryptographiquement l’accès aux secrets privés de l’utilisateur ;
- les secrets partagés doivent rester accessibles à d’autres propriétaires/utilisateurs autorisés pour éviter un point de perte unique ;
- avant suppression/recréation d’un utilisateur, contrôler les propriétaires et les partages.

Documentation officielle :
https://www.passbolt.com/docs/user/quickstart/browser/admin-assisted-recovery/

## SMTP

Le SMTP sert notamment aux invitations et notifications.

État actuel :

- fournisseur : GMX
- serveur : `mail.gmx.net`
- port : `587`
- TLS : activé
- authentification : mot de passe applicatif

Ne jamais commiter le compte ni le mot de passe applicatif.

## Healthcheck

```bash
sudo -H -u www-data /bin/bash -c \
  "/usr/share/php/passbolt/bin/cake passbolt healthcheck"
```

À exécuter après :

- mise à jour Passbolt ;
- mise à jour PHP importante ;
- changement TLS ;
- changement MariaDB ;
- restauration ;
- modification de la configuration Passbolt.

## Vérifications de base

```bash
systemctl is-active nginx
systemctl is-active php8.4-fpm
systemctl is-active mariadb

curl -Ik https://vault.ergo3.ch/
certbot certificates
```

Depuis la VM, `vault.ergo3.ch` doit être résolu localement de façon cohérente avec le vhost Nginx.

## Ajout d’un utilisateur

Principes :

1. créer/inviter l’utilisateur ;
2. vérifier la réception de l’email ;
3. laisser l’utilisateur générer/importer sa clé ;
4. imposer la conservation du recovery kit ;
5. configurer TOTP ;
6. ajouter l’utilisateur aux groupes nécessaires ;
7. partager explicitement les secrets nécessaires.

## Suppression / remplacement d’un utilisateur

Avant toute suppression :

1. identifier les ressources dont l’utilisateur est seul propriétaire ;
2. transférer les droits/partages nécessaires ;
3. vérifier l’existence d’au moins un autre utilisateur autorisé ;
4. seulement ensuite suspendre ou supprimer le compte selon le besoin.

## Sources officielles

- TOTP: https://www.passbolt.com/docs/admin/authentication/mfa/totp/
- Recovery kit: https://www.passbolt.com/docs/user/basic-features/browser/download-recovery-kit/
- Admin-assisted recovery: https://www.passbolt.com/docs/user/quickstart/browser/admin-assisted-recovery/
