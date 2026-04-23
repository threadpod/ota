# ThreadPod OTA

Distribution des mises à jour firmware signées pour les modules ThreadPod
(TPD, TPP, TPG, TPH, TPS).

## Comment ça marche

Chaque module embarque un client OTA qui, sur demande de l'utilisateur depuis
le dashboard, interroge le manifest de son channel actif :

```
https://threadpod.github.io/ota/<code>/<channel>.json
```

Exemples :
- `https://threadpod.github.io/ota/tpd/stable.json`
- `https://threadpod.github.io/ota/tph/latest.json`

Si une version plus récente est disponible **et** que le module est dans la
fenêtre de rollout, il télécharge le binaire depuis la GitHub Release
correspondante, vérifie SHA-256 puis **signature ECDSA P-256** avant
commutation de partition.

## Channels

| Channel | Usage |
| --- | --- |
| `stable` | Par défaut pour tous les modules en production. Tag `<code>-vX.Y.Z`. |
| `latest` | Nightlies / beta. Optionnel, activable depuis le dashboard. Prerelease. |

## Format du manifest

```json
{
  "module":          "tpd",
  "channel":         "stable",
  "version":         "1.0.6",
  "build":           48,
  "date":            "2026-04-20",
  "url":             "https://github.com/threadpod/ota/releases/download/tpd-v1.0.6/tpd.bin",
  "size":            1398272,
  "sha256":          "f3a1b2…",
  "sig_ecdsa_p256":  "3045022100…",
  "min_version":     "1.0.0",
  "rollout_percent": 100,
  "notes":           "WS2812 v2, QA script expand"
}
```

## Signature & transparence

Les binaires sont signés par la clé privée détenue par ThreadPod SAS.
La **clé publique** correspondante est publiée ici pour vérification
indépendante : [`pubkey.pem`](pubkey.pem).

Empreinte SHA-256 de la clé publique (à comparer avec celle embarquée
dans le firmware) : voir [`pubkey.sha256`](pubkey.sha256).

## Sécurité

- Ce repo est **public** par conception : les binaires distribués sont
  destinés à être exécutés sur les modules ; la signature garantit
  l'authenticité.
- Flash Encryption + Secure Boot V2 sont activés sur les modules de
  production (eFuses), ce qui empêche un binaire légitime de tourner sur
  un module tiers.
- Une brèche de la clé privée nécessiterait une rotation coordonnée avec
  une version intermédiaire du firmware embarquant la nouvelle clé.

## Canary rollout

Le champ `rollout_percent` permet un déploiement progressif :

- Chaque module calcule un bucket stable (FNV-1a de son MAC WiFi, modulo 100).
- Seuls les modules dont le bucket est < `rollout_percent` installent.
- Procédure typique : 10 % → 50 % → 100 % sur 48 h.

## Licence

Les binaires sont sous licence ThreadPod. La clé publique et les manifests
sont sous CC-BY-SA 4.0.
