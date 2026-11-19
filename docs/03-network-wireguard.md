# Réseau et WireGuard

## Rôle

La VM Passbolt est connectée au backbone WireGuard VitaLink pour l’administration sécurisée.

WireGuard n’est pas utilisé pour le challenge Let’s Encrypt : le renouvellement TLS est désormais réalisé directement depuis Passbolt via l’API DNS OVH.

## Peer Passbolt

| Paramètre | Valeur |
|---|---|
| Client ID | 028 |
| Nom | Ergo3 Passbolt |
| IP WireGuard | `10.203.0.37/32` |
| Clé publique Passbolt | `ry910oC25wh9aFsuknqScXRhKsrQbXu8QnuCEDhGPU4=` |
| Hub WG | `10.203.0.1` |
| Clé publique hub | `gytL99X0RG1vp6c0NcHznRw3Q/1hOTRqiVGn0q0bSX8=` |
| Port hub | `51820/UDP` |

L’adresse publique exacte du hub n’est volontairement pas stockée dans ce dépôt public.

## Configuration Passbolt

Fichier :

```text
/etc/wireguard/wg0.conf
```

Structure :

```ini
[Interface]
Address = 10.203.0.37/32
PrivateKey = <SECRET_LOCAL>

[Peer]
PublicKey = gytL99X0RG1vp6c0NcHznRw3Q/1hOTRqiVGn0q0bSX8=
Endpoint = <HUB_PUBLIC_ENDPOINT>:51820
AllowedIPs = 10.203.0.1/32
PersistentKeepalive = 25
```

La clé privée réelle ne doit jamais sortir de la VM.

## Configuration hub

Le hub contient le peer :

```ini
# 028 - Ergo3 Passbolt
[Peer]
PublicKey = ry910oC25wh9aFsuknqScXRhKsrQbXu8QnuCEDhGPU4=
AllowedIPs = 10.203.0.37/32
```

## Persistance

Sur Passbolt :

```bash
systemctl enable --now wg-quick@wg0
```

Contrôler :

```bash
systemctl is-enabled wg-quick@wg0
systemctl is-active wg-quick@wg0
ip -br a show wg0
wg show wg0
```

## Tests validés

Depuis Passbolt :

```bash
ping -c 3 10.203.0.1
```

Depuis le hub :

```bash
ping -c 3 10.203.0.37
```

Le 19 novembre 2026 :

- handshake WireGuard : OK
- ping Passbolt -> hub : OK
- ping hub -> Passbolt : OK
- latence observée : environ 14–15 ms

## Règles

- Ne pas attribuer de réseau virtuel `10.210.x.0/24` à Passbolt pour ce besoin.
- Passbolt utilise seulement son adresse WireGuard de management `10.203.0.37/32`.
- Ne jamais commiter `/etc/wireguard/privatekey`.
- Toute modification WireGuard doit être rendue persistante dans `wg0.conf` et vérifiée après redémarrage.
