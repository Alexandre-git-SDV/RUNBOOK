# 🛡️ Runbook sécurité — E-commerce client

> **Cas fictif** · Maintenance d'une application e-commerce
> **Stack :** NextJS (front) · FastAPI (back) · PostgreSQL (BDD)

---

> [!CAUTION]
> **Incident en cours ?**
> 1. Appeler l'**astreinte**.
> 2. En cas de **fuite de secret**, prévenir **Jean Claude** immédiatement.
> 3. Contenir avec le [kill switch](#4-rollback-et-kill-switch), puis suivre la [checklist](#8-réponse-à-incident).

## Sommaire

1. [Contacts et astreintes](#1-contacts-et-astreintes)
2. [Accès d'urgence](#2-accès-durgence)
3. [Secrets](#3-secrets)
4. [Rollback et kill switch](#4-rollback-et-kill-switch)
5. [Journaux](#5-journaux)
6. [Sauvegardes](#6-sauvegardes)
7. [Modèles de messages](#7-modèles-de-messages)
8. [Réponse à incident](#8-réponse-à-incident)

---

## 1. Contacts et astreintes

| Rôle | Nom | Quand l'appeler |
|---|---|---|
| 🔑 DSI, responsable des secrets | **Jean Claude** | **En priorité** pour toute fuite de secret |
| 🏗️ Architecte développement | **Michel Astra** | Incident applicatif, rollback, correctif |
| 🚨 Astreinte technique | Rotation hebdomadaire | Premier intervenant, 24/7 |



## 2. Accès d'urgence

- **Comptes break-glass** (admin cloud, BDD) : dans le coffre de mots de passe, accessibles au DSI et à l'architecte uniquement.
- **Serveurs :** VPN + SSH par clé, MFA obligatoire.
- **Après usage :** journaliser l'accès, changer le mot de passe et le remettre dans le coffre.

## 3. Secrets

| Secret | Stockage | Responsable | Révocation / renouvellement |
|---|---|---|---|
| Clés API **Stripe** | Coffre + variables d'env. back | Jean Claude | Dashboard Stripe → *Roll key* → coffre → redéploiement |
| Clés API **PayPal** | Coffre + variables d'env. back | Jean Claude | Console dev PayPal → régénérer → coffre → redéploiement |
| Secrets **webhooks** | Coffre + variables d'env. back | Jean Claude | Régénérer le *signing secret* → mettre à jour le back → vérifier la réception |
| Identifiants **PostgreSQL** | Coffre | Michel Astra | `ALTER ROLE … PASSWORD …` → config → redémarrer l'API |
| Secret **JWT / sessions** | Coffre | Michel Astra | Changer le secret → redéployer *(toutes les sessions sont invalidées)* |

> [!IMPORTANT]
> **En cas de fuite :** prévenir Jean Claude → **révoquer** → **renouveler** → redéployer → analyser l'usage du secret dans les logs.
> Si le secret a été commité, le purger aussi de l'historique git.

## 4. Rollback et kill switch

| Action | Procédure |
|---|---|
| **Rollback applicatif** | Jenkins → relancer le job de déploiement sur le dernier tag stable (front et back) |
| **Rollback BDD** | Migration descendante testée, sinon [restauration de la sauvegarde](#6-sauvegardes) |
| **Kill switch** | Mode maintenance (page statique) + coupure des paiements par feature flag |
| **Dernier recours** | Désactiver les clés de paiement chez Stripe / PayPal |

## 5. Journaux

| Outil | Contenu |
|---|---|
| **Logger** (FastAPI / NextJS) | Erreurs, authentification, paiements |
| **Jenkins** | Historique des déploiements (qui, quoi, quand) |
| **Uptime Kuma** | Disponibilité du site et alertes |

Conserver les logs au moins **90 jours** et ne rien supprimer pendant un incident.

## 6. Sauvegardes

| | |
|---|---|
| **Fréquence** | Dump PostgreSQL journalier |
| **Stockage** | Chiffré, hors du serveur de production |
| **Rétention** | 7 journalières + 4 hebdomadaires |
| **Test** | Restauration mensuelle en recette |

La dernière sauvegarde a été réalisée à 3h UTC+2.

## 7. Modèles de messages

> Remplacer les champs `[…]`. Ne jamais inclure de secret dans un message.

### 💾 Sauvegardes

**Sauvegarde réussie**
```
✅ [SAUVEGARDE] Sauvegarde PostgreSQL réalisée le JJ/MM/AAAA à HH:mm — taille : [X] Go — stockée chiffrée hors prod.
```

**Échec de sauvegarde**
```
❌ [SAUVEGARDE] Échec du dump PostgreSQL du JJ/MM/AAAA à HH:mm — erreur : [message].
Dernière sauvegarde valide : JJ/MM/AAAA HH:mm. Relance en cours par [nom].
```

**Restauration**
```
♻️ [RESTAURATION] Restauration de la sauvegarde du JJ/MM/AAAA HH:mm sur [prod/recette] lancée à HH:mm par [nom].
Perte de données possible depuis cette date. Fin estimée : HH:mm.
```

### 🔑 Fuite de secret *(interne)*

```
🚨 [FUITE SECRET] Secret exposé : [Stripe / PayPal / BDD / JWT…] — détecté le JJ/MM/AAAA à HH:mm via [source].
@Jean Claude prévenu. Révocation : [en cours / faite]. Ne pas réutiliser l'ancien secret.
```

### ⚠️ Incident en cours *(client)*

```
Bonjour, nous constatons depuis HH:mm un incident affectant [service / paiements].
Nos équipes sont mobilisées. Prochain point à HH:mm.
```

### 🛑 Kill switch / maintenance

```
🛑 [KILL SWITCH] Site en maintenance et paiements coupés depuis HH:mm par [nom]. Motif : [raison].
```

### ↩️ Rollback

```
↩️ [ROLLBACK] Retour à la version [tag] effectué à HH:mm par [nom]. Service vérifié : [oui/non].
```

### ✅ Résolution

```
✅ [RÉSOLU] Incident du JJ/MM/AAAA résolu à HH:mm. Cause : [résumé]. Post-mortem prévu le JJ/MM/AAAA.
```

### 🔒 Violation de données personnelles *(client / utilisateurs)*

```
Le JJ/MM/AAAA, nous avons détecté un accès non autorisé à [type de données].
Mesures prises : [actions]. La CNIL a été notifiée. Contact : [email].
```

## 8. Réponse à incident

- [ ] **Détecter :** alerte Uptime Kuma, logs ou signalement
- [ ] **Alerter :** astreinte, puis [escalade](#1-contacts-et-astreintes)
- [ ] **Contenir :** kill switch ou révocation des secrets
- [ ] **Corriger :** rollback ou correctif
- [ ] **Restaurer :** vérifier le service et les paiements
- [ ] **Notifier :** le client, et la **CNIL sous 72 h** si des données personnelles sont touchées
- [ ] **Post-mortem :** sous 5 jours ouvrés
