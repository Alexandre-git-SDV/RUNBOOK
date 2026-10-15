# RUNBOOK

Runbook de gestion des incidents cyber pour une application e-commerce fictive (NextJS, FastAPI, PostgreSQL).

👉 **[Ouvrir le runbook sécurité](./RUNBOOK.md)**

## Contenu

Le runbook indique **qui appeler** et **quoi faire** en cas d'incident :

- **Contacts et astreintes :** les rôles et l'ordre d'escalade
- **Accès d'urgence :** les comptes break-glass et les règles d'usage
- **Secrets :** où ils sont, qui en est responsable, comment les révoquer et les renouveler
- **Rollback et kill switch :** revenir en arrière ou couper le service et les paiements
- **Journaux et sauvegardes :** les sources de logs, la rétention, les restaurations
- **Réponse à incident :** une checklist de la détection au post-mortem

> ⚠️ Aucun secret réel ne doit être versionné dans ce dépôt : il contient des procédures, pas des identifiants.
