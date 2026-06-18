# 🎵 Playlist Vote

Un mini-site collaboratif pour proposer des chansons et voter pour créer une playlist ensemble — sans création de compte.

## Fonctionnalités

- 🎵 **Proposer** une chanson (titre + artiste)
- 👑 **Voter** pour ses préférées (1 vote par chanson, mémorisé localement)
- 🔄 **Mise à jour en temps réel** — tous les participants voient les votes instantanément
- 📱 **Responsive** mobile

---

## Mise en place pas à pas

### Étape 1 — Créer un projet Firebase

1. Aller sur [console.firebase.google.com](https://console.firebase.google.com/)
2. Cliquer sur **Ajouter un projet**
3. Donner un nom au projet (ex : `playlist-vote`)
4. Désactiver Google Analytics si inutile → **Créer le projet**
5. Une fois dans le tableau de bord, cliquer sur l'icône **Web** (`</>`) pour ajouter une application web
6. Donner un surnom (ex : `playlist-vote-web`), ne pas cocher Firebase Hosting
7. **Copier l'objet `firebaseConfig`** qui s'affiche — tu en auras besoin à l'étape 5

---

### Étape 2 — Activer Cloud Firestore

1. Menu gauche : **Build › Firestore Database**
2. Cliquer sur **Créer une base de données**
3. Choisir **Démarrer en mode test** *(on sécurisera les accès à l'étape suivante)*
4. Sélectionner la région la plus proche (ex : `europe-west3` pour Paris) → **Activer**

---

### Étape 3 — Configurer les règles de sécurité

Dans **Firestore › Règles**, remplacer tout le contenu par :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /songs/{songId} {

      // Lecture publique
      allow read: if true;

      // Création : champs requis, votes initiaux à 0, titre et artiste non vides
      allow create: if request.resource.data.keys().hasAll(['title','artist','votes','addedAt'])
                    && request.resource.data.votes == 0
                    && request.resource.data.title is string
                    && request.resource.data.artist is string
                    && request.resource.data.title.size() > 0
                    && request.resource.data.artist.size() > 0;

      // Mise à jour : uniquement incrémenter les votes de 1
      allow update: if request.resource.data.diff(resource.data).affectedKeys().hasOnly(['votes'])
                    && request.resource.data.votes == resource.data.votes + 1;

      // Suppression interdite
      allow delete: if false;
    }
  }
}
```

Cliquer sur **Publier**.

---

### Étape 4 — Créer l'index Firestore (tri par votes + date)

Le tri combiné `votes décroissant / date croissante` requiert un index composite.

**Option A — automatique (recommandée)**
Ouvre le site dans le navigateur après l'étape 6. Si l'index manque, Firestore affiche une erreur dans la **console du navigateur** (touche F12) avec un lien direct pour créer l'index en un clic. Attends ~2 minutes puis recharge la page.

**Option B — manuelle**
1. Aller dans **Firestore › Index › Index composites**
2. Cliquer sur **Créer un index**
3. Remplir comme suit :

| Champ    | Ordre       |
|----------|-------------|
| `votes`  | Décroissant |
| `addedAt`| Croissant   |

4. Collection : `songs` — Portée : Collection
5. Cliquer sur **Créer** (2 minutes environ)

---

### Étape 5 — Configurer le site

Ouvrir `index.html`, trouver le bloc `firebaseConfig` (lignes ~135-143) et remplacer chaque valeur `YOUR_*` :

```js
const firebaseConfig = {
  apiKey:            "AIzaSy...",
  authDomain:        "mon-projet.firebaseapp.com",
  projectId:         "mon-projet",
  storageBucket:     "mon-projet.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123456789:web:abc123"
};
```

---

### Étape 6 — Déployer sur GitHub Pages

1. Créer un dépôt GitHub (public ou privé)
2. Pousser les fichiers `index.html` et `README.md` sur la branche `main`
3. Aller dans **Settings › Pages** du dépôt
4. Sous **Source** : branche `main`, dossier `/ (root)`
5. Cliquer sur **Save**
6. Ton site sera disponible après ~30 secondes sur :
   ```
   https://<ton-pseudo>.github.io/<nom-du-repo>/
   ```

---

## Sécuriser la clé API (optionnel mais recommandé)

La clé API Firebase visible dans le code source est normale pour un projet web client. Pour limiter son usage :

1. Aller dans [Google Cloud Console › API Keys](https://console.cloud.google.com/apis/credentials)
2. Cliquer sur ta clé (`Browser key (auto created by Firebase)`)
3. Sous **Restrictions d'application**, choisir **Référents HTTP**
4. Ajouter `https://<ton-pseudo>.github.io/*`
5. **Enregistrer**

---

## Comment fonctionnent les votes

Sans authentification, les votes sont mémorisés dans le **`localStorage`** du navigateur (`pv_voted`). Un utilisateur ne peut voter qu'une fois par chanson depuis un même navigateur. Vider le cache navigateur permet de voter à nouveau — c'est un choix délibéré pour simplifier l'expérience dans un cadre informel.

---

## Structure du projet

```
.
├── index.html    ← l'application complète (HTML + CSS + JS)
└── README.md     ← ce fichier
```

## Structure Firestore

```
songs  (collection)
└── {songId}  (document auto-généré)
    ├── title     : string     — titre de la chanson
    ├── artist    : string     — nom de l'artiste
    ├── votes     : number     — nombre de votes (0 à la création)
    └── addedAt   : timestamp  — date d'ajout (pour le tri à égalité)
```
