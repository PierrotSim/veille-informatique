# Veille informatique

Site statique (HTML, CSS, JavaScript) destiné à GitHub Pages. Aucun serveur web, framework ou module npm n'est nécessaire.

## Déploiement

1. Décompresser l'archive ZIP.
2. Dans le dépôt GitHub, cliquer sur **Add file > Upload files**.
3. Déposer tous les fichiers extraits à la racine du dépôt (et non dans un sous-dossier). Valider avec **Commit changes**.
4. Dans **Settings > Pages**, choisir **Deploy from a branch**, branche `main` (ou la branche par défaut du dépôt), dossier `/ (root)`, puis **Save**.
5. Une fois le déploiement terminé, utiliser l'URL indiquée dans **Settings > Pages**.

Pour tester localement : depuis le dossier, lancer `python -m http.server 8000` (ou `python3 -m http.server 8000` selon le système), puis ouvrir `http://localhost:8000`. Ouvrir directement le fichier HTML via `file://` ne permet généralement pas de charger les JSON avec fetch.

## Fichiers

- `index.html` : dernière journée de publication et recherche.
- `historique.html` : ensemble des articles archivés, filtrables par source et date.
- `kpi.html` : compteurs calculés à partir des archives et indicateurs déclarés.
- `style.css` et `script.js` : présentation et logique du site.
- `articles.json` : données à actualiser depuis n8n. Initialisé à `[]`.
- `kpi.json` : indicateurs déclarés, modifiables manuellement ou via n8n si les données sont disponibles.
- `.nojekyll` : publication directe des fichiers statiques.

## Contrat des données n8n

`articles.json` doit être un tableau JSON. Exemple de structure (données fictives uniquement pour illustrer le format ; elles ne sont pas incluses dans le site) :

```json
[
  {
    "title": "Titre de l'article",
    "description": "Résumé en français, validé par le workflow.",
    "link": "https://example.com/article",
    "source": "IT-Connect",
    "publishedAt": "2026-11-24T20:00:00+01:00",
    "pubDate": "2026-11-24T14:00:00+01:00"
  }
]
```

`publishedAt` correspond à la date de publication de l'article **sur le site de veille** (date du bulletin n8n), tandis que `pubDate` est la date de l'article original. Le site accepte également `date` ou `pubDate` si `publishedAt` est absent. Fournir de préférence une date ISO 8601 avec fuseau. Les URLs HTTP/HTTPS sont acceptées ; les titres et descriptions sont affichés comme texte, pas comme HTML.

Pour conserver l'historique :

1. Récupérer `articles.json` existant avec l'API GitHub (GET `/repos/{owner}/{repo}/contents/articles.json?ref=main`).
2. Décoder le contenu Base64 de la réponse, puis parser le JSON.
3. Fusionner les anciens articles avec les nouveaux articles validés. Dédupliquer sur leur lien, en conservant la première date de publication sur le site. Ne pas écraser les archives avec uniquement les articles du jour.
4. Écrire le tableau JSON complet dans `articles.json` via le node GitHub (File > Edit) ou l'API Contents de GitHub (PUT avec le SHA actuel du fichier, son contenu encodé en Base64 et un message de commit).
5. Si l'exécution ne produit aucun article, conserver les archives inchangées. Éviter les écritures concurrentes ; en cas de conflit, relire le fichier avant de retenter la fusion.

Le token GitHub doit rester dans les Credentials n8n ; limiter ses droits au dépôt et à Contents en lecture/écriture. Ne jamais publier le token, des webhooks secrets ni des exports contenant des identifiants dans le dépôt. Un dépôt public expose le contenu des fichiers JSON. N'y publier que des données destinées à être publiques.

Le site reste disponible quand le PC est éteint, mais n8n ne publie pas de nouvelles données lorsque son conteneur n'est pas en fonctionnement. GitHub Pages peut nécessiter un délai après chaque commit.

## KPI

Les compteurs d'articles, les sources représentées et la répartition par source sont calculés sur les archives, et non sur l'état réel des six flux RSS. Les taux de fiabilité (95 %) et de consultation (85 %), l'exploitation (2 à 3 articles par jour) et les autres valeurs de `kpi.json` sont les valeurs déclarées du projet. Le site ne mesure pas les clics et ne confirme pas les exécutions n8n. Pour automatiser ces mesures, il faudra collecter les événements pertinents et définir une période et une méthode de calcul.
