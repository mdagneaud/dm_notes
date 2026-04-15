# DM_notes

Application web progressive (PWA) de prise de notes et de gestion de tâches. Fonctionne entièrement dans le navigateur, sans serveur, sans compte utilisateur, sans dépendance externe.

**[Ouvrir l'application](https://mdagneaud.github.io/dm_notes/)**

## Fonctionnalités

- Notes texte avec éditeur riche (gras, barré, listes, séparateurs)
- Listes de tâches avec 4 statuts : pas commencée, en attente, en cours, terminée
- Priorisation selon la matrice d'Eisenhower (4 quadrants)
- Matrice Eisenhower interactive et vue tâches transversale
- Rapport d'avancement en Markdown — compatible Teams, Slack, Notion
- Étiquettes colorées filtrables, recherche plein texte
- Export/import chiffré AES-256-GCM
- Mode clair/sombre, installation PWA sur Android et PC
- Fonctionne hors ligne

## Installation

Ouvrir [https://mdagneaud.github.io/dm_notes/](https://mdagneaud.github.io/dm_notes/) dans Chrome.

**Android** — menu ⋮ → Ajouter à l'écran d'accueil

**PC** — icône ⊕ dans la barre d'adresse, ou menu ⋮ → Installer DM_notes

## Données et confidentialité

Toutes les données sont stockées localement dans le navigateur (IndexedDB). Aucune donnée ne transite par un serveur. L'export chiffré utilise AES-256-GCM avec dérivation PBKDF2 (100 000 itérations).

## Licence

[GNU Affero General Public License v3.0](LICENSE)
