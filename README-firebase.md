# Atelion — Firebase : connexion Google + sauvegarde cloud

Ce document explique comment relier `atelion.html` à **votre** projet Firebase
pour que vos plans soient enregistrés dans le cloud et accessibles depuis n'importe quel appareil.

> ⚠️ **À savoir avant de commencer** : la connexion Google **ne fonctionne pas** quand la page
> est ouverte en double-cliquant le fichier (`file://`). Google exige une adresse `http(s)`.
> Deux solutions : un petit serveur local (`http://localhost`) pour travailler,
> et **Firebase Hosting** pour avoir une vraie adresse à ouvrir partout.

---

## 1. Créer le projet Firebase (5 minutes)

1. Va sur <https://console.firebase.google.com> et clique **Ajouter un projet**.
   Donne-lui un nom (ex. `atelion`). Google Analytics n'est pas nécessaire.
2. Dans le menu de gauche : **Authentication** → *Commencer* → onglet **Sign-in method** →
   active **Google** (choisis un e-mail de support) → **Enregistrer**.
3. Toujours à gauche : **Firestore Database** → *Créer une base de données* →
   mode **Production** → emplacement (ex. `eur3 (Europe)`) → *Activer*.
4. **Paramètres du projet** (roue dentée en haut à gauche) → onglet **Général** →
   descends jusqu'à **Vos applications** → **`</>`** (Web) → donne un surnom → **Enregistrer**.
   Firebase affiche un bloc `firebaseConfig` : **garde-le sous la main**.

## 2. Coller la configuration dans l'application

Ouvre `atelion.html` dans un éditeur de texte, descends tout en bas, et repère :

```html
<script id="firebase-config" type="application/json">
{
  "apiKey": "REMPLACER",
  "authDomain": "REMPLACER.firebaseapp.com",
  "projectId": "REMPLACER",
  "storageBucket": "REMPLACER.appspot.com",
  "messagingSenderId": "REMPLACER",
  "appId": "REMPLACER"
}
</script>
```

Remplace les valeurs par celles de ta console (garde les guillemets), par exemple :

```json
{
  "apiKey": "AIzaSyC...",
  "authDomain": "atelion-5b5f1.firebaseapp.com",
  "projectId": "atelion-5b5f1",
  "storageBucket": "atelion-5b5f1.firebasestorage.app",
  "messagingSenderId": "123456789012",
  "appId": "1:123456789012:web:abcdef123456"
}
```

Enregistre, recharge la page : le bouton **Cloud** en haut ouvre le panneau de connexion.

## 3. Autoriser le domaine

Dans **Authentication → Paramètres → Domaines autorisés**, `localhost` y est déjà.
Si tu déploies sur Firebase Hosting, ton domaine `xxx.web.app` est ajouté automatiquement.
Si tu utilises un autre hébergeur (GitHub Pages…), ajoute son domaine à la main.

## 4. Servir la page en http(s)

### Option A — en local (le plus rapide pour tester)

Dans le dossier qui contient `atelion.html` :

```bash
npx serve .            # ou : python -m http.server 8080
```

Puis ouvre **http://localhost:8080/atelion.html** (et non le fichier directement).
La connexion Google fonctionne ; les données partent dans Firestore.

### Option B — en ligne avec Firebase Hosting (adresse fixe, partageable)

```bash
npm install -g firebase-tools
firebase login
cd <dossier contenant atelion.html et firebase.json>
firebase use --add          # choisir le projet créé à l'étape 1
firebase deploy --only hosting
```

Firebase affiche une adresse du type `https://atelion-5b5f1.web.app`.
Le fichier `firebase.json` fourni est déjà prêt (il sert `atelion.html`).

## 5. Règles Firestore (important)

Par défaut, Firestore **refuse** toutes les lectures/écritures. Dans
**Firestore Database → Règles**, colle le contenu de `firestore.rules` :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid}/projets/{projetId} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

Chaque utilisateur ne peut lire et écrire que dans son propre dossier `projets`.

---

## Comment ça marche dans l'application

- Bouton **Cloud** (barre du haut) : se connecter avec Google, voir l'état de synchronisation,
  enregistrer immédiatement, lister/ouvrir/supprimer ses projets.
- Une fois connecté, **chaque modification est envoyée automatiquement** (1,6 s après la dernière
  action) dans `users/{uid}/projets/{id}`.
- Le projet garde un identifiant cloud (`cloudId`) : les sauvegardes suivantes mettent à jour
  la même fiche, tu ne crées pas de doublons.
- Le bouton **Enregistrer** (fichier `.json`) et la sauvegarde locale du navigateur continuent
  de fonctionner : tu peux travailler hors ligne, tout reste sur le poste.

### Structure d'un document

```
users/{uid}/projets/{projetId}
  name        "Séjour Duval"
  project     { name, client, date, cloudId }
  room        { w, d, h }
  items       [ ... ]
  view, view2d, cam, toggles
  schema      1
  updatedAt   <timestamp serveur>
```

## Dépannage

| Symptôme | Cause / solution |
|---|---|
| « Connexion Google impossible depuis un fichier local » | La page est ouverte en `file://` → lance un serveur local (étape 4A) ou déploie (4B). |
| `auth/unauthorized-domain` | Ajoute le domaine dans **Authentication → Paramètres → Domaines autorisés**. |
| `auth/popup-blocked` | Autorise les pop-ups pour ce site, ou réessaie (la popup doit s'ouvrir après le clic). |
| `permission-denied` à la sauvegarde | Les règles Firestore ne sont pas celles de l'étape 5. |
| `Firebase n'a pas pu démarrer` | Configuration incomplète (guillemets/valeurs), ou réseau bloquant `gstatic.com`. |
| Rien ne se sauvegarde | Vérifie que le bouton affiche **Cloud ✓** (connecté) et que la console (F12) ne montre pas d'erreur. |

## Sécurité

- La `apiKey` Firebase n'est **pas** un secret : elle identifie le projet, elle ne donne aucun droit.
  Ce sont les **règles Firestore** qui protègent les données (étape 5).
- La configuration est visible dans le fichier : c'est normal pour une application web Firebase.
- Cette clé est en plus **restreinte** (Google Cloud → *API et services → Identifiants → Browser key*) :

  | Type de restriction | Valeur |
  |---|---|
  | Restrictions relatives aux applications | **Sites Web** |
  | Référents autorisés | `http://localhost:8000/*` et `https://atelion-5b5f1.web.app/*` |

  ⚠️ **Si tu ajoutes un jour un autre domaine** (GitHub Pages, nom de domaine perso…), il faut
  l'ajouter dans cette liste, sinon les appels Firebase seront refusés (`requests from referer <> are blocked`).
  Et pour le développement local, ouvre l'appli via `http://localhost:8000/...` et non `http://127.0.0.1:8000/...`
  (ce sont deux origines différentes pour Google).
- GitHub peut signaler la clé comme « secret exposé » dans un dépôt public : c'est un faux positif
  pour Firebase. Il suffit de classer l'alerte (onglet *Security → Secret scanning*).

