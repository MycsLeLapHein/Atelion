# Atelion

**Plans 2D / 3D et fiches de débit pour la menuiserie** — une application web, un seul fichier HTML, toutes les cotes en centimètres.

> Conçue pour l'atelier : on pose un meuble, on tape ses cotes, on voit le plan et la vue 3D, et on sort la liste de panneaux à débiter.

---

## Fonctionnalités

### Plans
- **Plan 2D** coté, avec grille et règles graduées en cm, accrochage au cm / aux murs / aux arêtes des autres meubles.
- **Vue 3D** temps réel (moteur maison, aucune librairie 3D), ombres portées, caméra orbitale, vues Iso / Dessus / Face.
- **2D + 3D côte à côte** pour contrôler en travaillant.
- Poignées de déplacement, redimensionnement et rotation (pas de 5°).

### 20 formes prêtes à l'emploi, modifiables au cm
Placard 2 portes · Bibliothèque · Commode · Meuble TV · Étagère murale · Table · Table ronde · Bureau · Chaise · Banc · Meuble bas + plan de travail · Îlot central · Meuble haut · Lit · Escalier · Bloc · Planche · Colonne — plus un **meuble sur mesure** assemblé **panneau par panneau** (joues, dessus, dessous, fond, tablettes, portes, pieds) avec orientation Face / Côté / À plat.

### Débit & sorties
- **Fiche de débit** : longueur, largeur, épaisseur, quantité, m², métrage de chant, volume, détail par élément.
- **Impression** d'une planche A4 (plan + vue 3D + fiche de débit + cartouche).
- Export **PNG** (plan et 3D), export **CSV** du débit, **enregistrement** du projet en `.json`.

### Confort
- Sauvegarde automatique dans le navigateur, **annuler/rétablir**, raccourcis clavier complets.
- **Version tactile** : tiroirs, zoom par pincement, cibles agrandies.
- **Compte Google + sauvegarde cloud** via Firebase (optionnel, voir [README-firebase.md](README-firebase.md)).

---

## Démarrage rapide

L'application doit être servie en `http(s)` : la connexion Google et Firebase refusent `file://`.

```bash
# dans ce dossier
npx serve .
# ou
python -m http.server 8000
```

Puis ouvrir **http://localhost:8000/atelion.html**

## Déploiement

### Firebase Hosting (recommandé)

```bash
npm install -g firebase-tools
firebase login
firebase use --add        # choisir le projet atelion-5b5f1
firebase deploy --only hosting
```

→ `https://atelion-5b5f1.web.app`

### GitHub Pages

Le fichier `index.html` redirige vers `atelion.html` : activez **Settings → Pages → Deploy from branch → main / (root)**.
⚠️ Ajoutez ensuite le domaine `votrepseudo.github.io` dans **Firebase → Authentication → Paramètres → Domaines autorisés**, sinon la connexion Google sera refusée.

---

## Structure

```
atelion.html          application complète (HTML + CSS + JS, sans dépendance)
index.html            redirection vers atelion.html (GitHub Pages)
atelion-logo-512.png  sigle (512×512)
firebase.json         configuration Hosting
firestore.rules       règles de sécurité Firestore
README-firebase.md    mise en place de la connexion Google + cloud
```

L'application est **autonome** : le fichier `atelion.html` fonctionne seul, sans build ni dépendance. Seule la partie « Cloud » charge le SDK Firebase depuis le CDN Google, et uniquement si la configuration est renseignée.

## Données & confidentialité

Tout est calculé et stocké **dans le navigateur** (localStorage) par défaut.
Si vous activez le cloud, vos plans sont enregistrés dans **votre** projet Firebase (`users/{uid}/projets/{id}`), protégés par les règles fournies : chaque compte ne voit que ses propres projets.

---

© Atelion — menuiserie & agencement. Code fourni pour un usage personnel ; pas de licence de redistribution.
