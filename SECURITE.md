# Analyse de sécurité de DM_notes

Version analysée : 3.6.0. Corrections livrées : 3.7.0. Date : 29 septembre 2026.

## Périmètre et modèle de menace

DM_notes est une application web monopage (un fichier `index.html`), servie par GitHub Pages et installée comme PWA. Il n'y a ni serveur applicatif, ni compte, ni synchronisation : les données vivent dans le navigateur (IndexedDB, avec une copie dans localStorage). La surface d'attaque est donc réduite, mais pas nulle. Trois scénarios comptent réellement. Le premier est un fichier d'import piégé : quelqu'un transmet un export JSON dont le contenu exécute du code à l'ouverture. Le deuxième est la chaîne d'approvisionnement : le dépôt GitHub, le compte qui le contrôle et les ressources chargées depuis des CDN tiers ont tous la capacité de modifier le code qui lit l'intégralité des notes. Le troisième est l'accès physique ou logiciel à l'appareil, où les données sont stockées en clair.

## Constats et traitement

### 1. Injection de code via les données (XSS stockée) : critique, corrigé

Les titres, textes de tâches, noms d'étiquettes, identifiants et couleurs étaient insérés dans la page via `innerHTML` sans échappement. Le contenu des notes, qui est du HTML, était réinjecté tel quel dans l'éditeur. Un fichier importé contenant par exemple un titre `<img src=x onerror=...>` exécutait donc du JavaScript avec un accès complet aux notes, y compris la capacité de les envoyer à l'extérieur. Le même défaut rendait fragile la saisie courante (un texte de tâche contenant `<b>` était interprété).

Correction : une fonction `esc()` échappe toute donnée affichée ; `sanitizeHtml()` n'autorise dans le contenu des notes qu'une liste blanche de balises de mise en forme, sans aucun attribut ; `sanitizeData()` normalise chaque import, restauration et chargement (identifiants limités à `[A-Za-z0-9_-]`, couleurs au format `#RRGGBB`, dates au format `AAAA-MM-JJ`, criticités, priorités et statuts limités aux valeurs connues). L'aperçu des cartes est calculé par `DOMParser`, qui n'exécute rien. Le correctif a été vérifié avec un fichier volontairement piégé : aucune des cinq charges d'injection ne s'exécute.

### 2. Absence de politique de sécurité du contenu : élevé, corrigé en partie

GitHub Pages ignore le fichier `_headers` (il sert à Netlify et Cloudflare Pages), donc aucun en-tête de sécurité n'était envoyé. Une balise `<meta http-equiv="Content-Security-Policy">` limite désormais les connexions sortantes au domaine de l'application, bloque les objets, les cadres, les formulaires et la balise `base`. Le script inline impose encore `'unsafe-inline'` pour les scripts : la politique ne suffit pas à elle seule contre une injection, mais elle empêche une injection éventuelle d'exfiltrer les données par `fetch` ou par une image distante. Pour aller plus loin, il faudrait sortir le JavaScript dans un fichier `app.js` et retirer `'unsafe-inline'` de `script-src`. Les directives `frame-ancestors` (protection contre le clickjacking) ne fonctionnent qu'en en-tête HTTP, pas en balise meta.

### 3. Ressources tierces : moyen, corrigé en partie

La feuille d'icônes Bootstrap venait de jsDelivr sans empreinte d'intégrité : une compromission du CDN aurait modifié la page. L'attribut `integrity` (SHA-384) est ajouté. Google Fonts ne peut pas recevoir d'empreinte car la réponse varie selon le navigateur. Surtout, son chargement transmet l'adresse IP de l'utilisateur à Google, ce qu'un tribunal allemand a jugé contraire au RGPD en 2022. Recommandation : héberger la police Anek Latin et les icônes dans le dépôt, ce qui supprime à la fois le risque tiers et la dépendance réseau.

### 4. Chiffrement des exports : moyen, corrigé

Le schéma (AES-256-GCM, sel de 16 octets et vecteur d'initialisation de 12 octets aléatoires, clé dérivée par PBKDF2-SHA256) est correct. En revanche 100 000 itérations sont en dessous de la recommandation OWASP actuelle (600 000), et aucun mot de passe n'était refusé, même d'un caractère. Les nouveaux exports utilisent 600 000 itérations, inscrites dans le fichier (`iter`), et exigent 12 caractères. Les anciens exports restent lisibles. À l'import, le mot de passe était saisi dans une fenêtre `prompt()` en clair ; il passe désormais par un champ masqué, effacé du formulaire après usage.

### 5. Données en clair sur l'appareil : moyen, non corrigé

IndexedDB et la sauvegarde `notes-backup` de localStorage contiennent toutes les notes en clair. Toute personne ou extension ayant accès au profil du navigateur peut les lire. C'est le fonctionnement normal d'une application locale ; un chiffrement au repos supposerait de saisir un mot de passe à chaque ouverture. À réserver au cas où des informations sensibles (données personnelles de tiers, informations professionnelles confidentielles) y seraient stockées. Dans l'intervalle, le verrouillage de l'appareil et du profil navigateur est la protection effective.

### 6. Chaîne d'approvisionnement GitHub : moyen, action hors code

Quiconque peut pousser sur la branche publiée peut livrer un code qui lit toutes les notes des appareils qui chargent l'application. Recommandations : activer la double authentification sur le compte GitHub, protéger la branche publiée, et privilégier les propositions de modification (pull requests) plutôt que les dépôts directs de fichiers.

### 7. Saisie vocale : faible, information

Dans Chrome, l'API Web Speech envoie l'audio aux serveurs de Google pour la transcription. À éviter pour dicter des contenus confidentiels.

### 8. Import : faible, recommandation

L'import remplace toutes les données après une simple confirmation, sans copie préalable. Un export automatique avant remplacement éviterait une perte irréversible en cas de mauvais fichier.

### 9. Points vérifiés sans défaut

Pas de gestionnaire d'événement inline dans le HTML, pas d'`eval`, pas de service worker (donc pas de risque de cache empoisonné, mais pas non plus de fonctionnement hors ligne réel, contrairement à ce qu'indique le README), vérification de mise à jour limitée au fichier `version.json` du même domaine, rapport Markdown produit via `textContent`.

## Anomalie constatée au passage

Le `version.json` publié annonçait 3.3.0 alors que l'application était en 3.6.0 : le bandeau « nouvelle version » proposait donc une version plus ancienne. Les deux sont alignés en 3.7.0.
