# Bascule

Application de fiction interactive en français. Un seul code pour Android et iOS (Expo / React Native),
plus une API Node qui porte le cache et le routage des modèles.

## Démarrer

```bash
npm install
npm start          # puis « a » pour Android, « i » pour iOS, ou scanner le QR code avec Expo Go
```

L'application fonctionne telle quelle, sans serveur : le contenu de l'épisode est embarqué
(`content/rive-gauche-ep3.json`) et sert de mode hors ligne.

Pour lancer l'API :

```bash
cd server && npm install
cp .env.example .env     # renseigner au moins une clé, ou aucune pour rester en mode statique
npm run dev
```

Puis pointer l'app vers le serveur dans `app.json` → `extra.apiUrl`.
Sur un téléphone physique, remplacer `localhost` par l'adresse IP locale du poste
(par exemple `http://192.168.1.20:4000`), sinon le téléphone cherche son propre localhost.

## Construire les binaires

```bash
npm i -g eas-cli && eas login
eas build -p android --profile preview   # APK installable directement
eas build -p ios --profile preview       # nécessite un compte Apple Developer
```

Avant la première construction : renseigner `extra.eas.projectId` (donné par `eas init`)
et vérifier les identifiants `com.ministereapps.bascule`.

## Structure

```
app/                  écrans (expo-router)
  index.tsx           catalogue
  serie/[slug].tsx    fiche série
  lecteur.tsx         lecteur d'épisode
  biblio.tsx          bibliothèque
src/
  components/         Cover, Scene, Bulle, Coulisses, Chrome
  data/               séries et types
  lib/api.ts          client API, cache local, repli hors ligne
  lib/cacheKey.ts     construction de la clé de cache
  store.ts            état du joueur, crédits, progression (persistés via MMKV)
content/              épisodes au format produit par le prompt de découpage
  bible-rive-gauche.json  bible de série, qui alimente le prompt système
server/
  src/index.js        API : catalogue, render, choice
  src/cache.js        clé normalisée, paliers, cache mémoire
  src/routeur.js      trois niveaux de modèles, repli, disjoncteur, validation
  src/moderation.js   filet de sécurité commun à toutes les sorties
  src/journal.js      coût réel par beat et par modèle, taux de cache
  src/llm/            adaptateurs, catalogue de modèles, prompts
  scripts/            aperçu de prompt, test à l'aveugle entre modèles
assets/               icône, splash, polices
```

## Le moteur narratif

```bash
cd server
npm run test:prompt -- b4    # affiche le prompt exact, sans rien appeler
npm run test:modele -- b4    # même beat généré par chaque modèle configuré, en aveugle
```

Le test à l'aveugle mélange et anonymise les sorties : vous notez le naturel du français,
la voix des personnages, la tenue du rythme et le respect de l'objectif, puis le script révèle
quel modèle a écrit quoi. C'est la seule façon fiable de choisir.

Trois niveaux de qualité, déclarés par le beat lui-même (`"qualite": "premium"`) :

| Niveau | Part du volume | Fournisseur par défaut |
|---|---|---|
| `rapide` | 60 à 70 % | modèle open source auto-hébergé en Europe |
| `standard` | 25 à 35 % | Mistral (hébergement UE) |
| `premium` | 5 à 10 % | le meilleur modèle disponible |

Un modèle sans clé est écarté : le routeur descend d'un cran, et si aucun n'est configuré,
le texte écrit à la main est servi. Le joueur n'est jamais bloqué par une panne de fournisseur.

Suivi des coûts : `GET /v1/debug/journal` donne le taux de cache, le coût par modèle et par beat.
En dessous de 70 % de taux de cache, la facture dérape — la clé contient trop de variables,
ou les paliers sont trop fins.

## Ce qui reste à brancher

- **Les visuels.** `src/components/Cover.tsx` et `Scene.tsx` dessinent des décors de substitution en SVG.
  À remplacer par les images et clips produits par le pipeline, servis depuis un CDN.
- **Les achats intégrés.** Les crédits sont locaux. Pour la production : `expo-in-app-purchases`
  ou RevenueCat, avec vérification du reçu côté serveur.
- **Le panneau « Coulisses »** est un outil de développement. À masquer avant publication
  (ou à réserver à un compte interne).

## Conformité

- Les contenus générés par IA doivent être signalés comme tels (AI Act).
- Classification 16+ sur les deux stores, avec filtres de modération côté serveur.
- Mentionner la collecte de données dans la fiche App Store Privacy et le formulaire Play Data Safety.

Le Ministère des Applications — support@ministere-apps.com
