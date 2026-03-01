# 🚗 Guide d'intégration complet — AutoPartage
## De zéro à en ligne sur GoDaddy + Supabase + GitHub Pages

---

## Vue d'ensemble de l'architecture

```
┌─────────────────────────────────────────────┐
│  Votre site GoDaddy (intranet résidence)    │
│  → Lien ou iframe vers la plateforme       │
└────────────────────┬────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────┐
│  GitHub Pages                               │
│  → index.html  (l'application)             │
│  → config.js   (clés Supabase)             │
└────────────────────┬────────────────────────┘
                     │  HTTPS sécurisé
                     ▼
┌─────────────────────────────────────────────┐
│  Supabase (cloud gratuit)                   │
│  → Auth : connexion sécurisée              │
│  → Base de données PostgreSQL              │
│  → Row Level Security (RLS)               │
└─────────────────────────────────────────────┘
```

**Durée estimée : 45 à 60 minutes**
**Coût : 0 $ (tout est gratuit)**

---

## PARTIE 1 — SUPABASE (base de données)
### Durée : ~15 minutes

---

### Étape 1 · Créer un compte Supabase

1. Ouvrez **[supabase.com](https://supabase.com)** dans votre navigateur
2. Cliquez **Start your project**
3. Connectez-vous avec votre compte **GitHub** (c'est le plus simple)
4. Vous arrivez sur le tableau de bord Supabase

---

### Étape 2 · Créer votre projet

1. Cliquez le bouton vert **New project**
2. Remplissez le formulaire :
   - **Organization** : laissez la valeur par défaut
   - **Name** : `autopartage` (ou n'importe quel nom)
   - **Database Password** : cliquez **Generate a password**, copiez-le dans un endroit sûr (ex: bloc-notes)
   - **Region** : choisissez **Canada (Central)** ou **US East**
3. Cliquez **Create new project**
4. ⏳ Attendez **2 à 3 minutes** que le projet se mette en place (barre de progression verte)

---

### Étape 3 · Créer les tables (script SQL)

1. Dans le menu gauche, cliquez **SQL Editor** (icône `< >`)
2. Cliquez **New query** (bouton en haut à gauche)
3. Ouvrez le fichier **`setup.sql`** fourni dans un éditeur de texte (Bloc-notes, TextEdit, VS Code…)
4. Sélectionnez **tout le contenu** (Ctrl+A / Cmd+A) et copiez (Ctrl+C / Cmd+C)
5. Collez dans la fenêtre SQL Editor de Supabase (Ctrl+V / Cmd+V)
6. Cliquez le bouton vert **Run** (▶) en bas à droite
7. ✅ Vous devriez voir : `Success. No rows returned`

> **Si vous voyez une erreur** : vérifiez que vous avez bien copié TOUT le contenu du fichier, depuis la première ligne jusqu'à la dernière.

---

### Étape 4 · Créer le premier compte administrateur

1. Toujours dans **SQL Editor**, cliquez **New query**
2. Copiez-collez le bloc ci-dessous en remplaçant les 3 valeurs :

```sql
INSERT INTO auth.users(
  instance_id, id, aud, role, email,
  encrypted_password, email_confirmed_at,
  raw_user_meta_data, created_at, updated_at
) VALUES (
  '00000000-0000-0000-0000-000000000000',
  gen_random_uuid(),
  'authenticated', 'authenticated',
  'VOTRE_EMAIL@example.com',
  crypt('VOTRE_MOT_DE_PASSE', gen_salt('bf')),
  NOW(),
  '{"full_name":"Votre Nom","apt":"Admin","role":"admin"}',
  NOW(), NOW()
);
```

   - Remplacez `VOTRE_EMAIL@example.com` par votre vraie adresse email
   - Remplacez `VOTRE_MOT_DE_PASSE` par un mot de passe fort (min. 8 caractères)
   - Remplacez `Votre Nom` par votre nom

3. Cliquez **Run** (▶)
4. ✅ Résultat attendu : `Success. 1 rows affected`

---

### Étape 5 · Récupérer vos clés API

1. Dans le menu gauche, cliquez **Settings** (icône ⚙️ en bas)
2. Cliquez **API** dans le sous-menu
3. Notez (copiez dans un bloc-notes) les deux valeurs suivantes :
   - **Project URL** : ressemble à `https://abcdefghijkl.supabase.co`
   - **anon / public** (dans la section "Project API keys") : longue chaîne commençant par `eyJ...`

> ⚠️ Ne partagez jamais votre clé **service_role**. Seule la clé **anon** est utilisée ici.

---

## PARTIE 2 — CONFIGURATION DES FICHIERS
### Durée : ~5 minutes

---

### Étape 6 · Modifier config.js

1. Ouvrez le fichier **`config.js`** dans un éditeur de texte
2. Remplacez les deux lignes avec vos vraies valeurs :

```javascript
const SUPABASE_URL     = "https://abcdefghijkl.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
```

3. Sauvegardez le fichier

> **Exemple concret :**
> ```javascript
> const SUPABASE_URL     = "https://xyzxyzxyz.supabase.co";
> const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXV...longue chaine...";
> ```

---

## PARTIE 3 — GITHUB PAGES (hébergement)
### Durée : ~10 minutes

---

### Étape 7 · Créer un dépôt GitHub

1. Ouvrez **[github.com](https://github.com)** — connectez-vous à votre compte
2. Cliquez le **+** en haut à droite → **New repository**
3. Remplissez :
   - **Repository name** : `autopartage` (tout en minuscules, sans espaces)
   - **Visibility** : cochez **Public** *(obligatoire pour GitHub Pages gratuit)*
   - Laissez tout le reste par défaut
4. Cliquez **Create repository**

---

### Étape 8 · Uploader les fichiers

Sur la page de votre nouveau dépôt vide :

1. Cliquez **Add file** → **Upload files**
2. Glissez-déposez (ou cliquez pour sélectionner) les **2 fichiers** :
   - `index.html`
   - `config.js`
3. En bas de la page, dans **Commit changes**, laissez le message par défaut ou écrivez `Initial commit`
4. Cliquez le bouton vert **Commit changes**
5. ✅ Vous voyez maintenant vos 2 fichiers listés dans le dépôt

---

### Étape 9 · Activer GitHub Pages

1. Dans votre dépôt GitHub, cliquez l'onglet **Settings** (en haut, icône ⚙️)
2. Dans le menu gauche, faites défiler jusqu'à **Pages**
3. Sous **Build and deployment** :
   - **Source** : sélectionnez **Deploy from a branch**
   - **Branch** : sélectionnez **main** · **/ (root)**
4. Cliquez **Save**
5. ⏳ Attendez **1 à 2 minutes**
6. Rafraîchissez la page → vous verrez apparaître en vert :
   **"Your site is live at https://votre-nom.github.io/autopartage"**
7. 🎉 Cliquez ce lien pour tester votre application !

---

### Étape 10 · Tester la connexion

1. Ouvrez l'URL GitHub Pages dans votre navigateur
2. L'écran de connexion apparaît
3. Connectez-vous avec l'email et le mot de passe créés à l'**Étape 4**
4. Vous devriez voir le tableau de bord avec les 4 véhicules pré-configurés
5. ✅ Si tout s'affiche correctement, votre plateforme est opérationnelle !

> **Si vous voyez "Connexion à la base de données…" indéfiniment** :
> → Vérifiez le fichier `config.js` : les deux valeurs doivent être entre guillemets et sans espaces supplémentaires.

---

## PARTIE 4 — INTÉGRATION GODADDY
### Durée : ~10 minutes

---

### Étape 11 · Accéder à la zone HTML personnalisée GoDaddy

GoDaddy propose différents types d'hébergement. Voici comment trouver la zone HTML selon votre configuration :

**Si vous utilisez le Website Builder GoDaddy :**
1. Connectez-vous sur **godaddy.com** → **Mes produits**
2. Cliquez **Gérer** sur votre site web
3. Dans l'éditeur, cherchez une section ou un bloc **HTML personnalisé**
   *(cherchez "HTML" dans la barre de recherche des widgets/sections)*
4. Glissez-déposez ce bloc à l'endroit souhaité dans votre page

**Si vous utilisez l'hébergement cPanel (fichiers) :**
1. Accédez au **cPanel** de votre hébergement GoDaddy
2. Ouvrez le **Gestionnaire de fichiers**
3. Naviguez vers le dossier `public_html`
4. Ouvrez le fichier HTML de votre page (ex: `index.html` ou `intranet.html`)
5. Modifiez-le avec l'éditeur intégré

---

### Étape 12 · Ajouter le lien vers AutoPartage

Vous avez **deux options** selon vos préférences :

---

#### ✅ Option A — Bouton lien (recommandée)

La plus fiable. Ouvre AutoPartage dans un nouvel onglet.

Collez ce code dans votre zone HTML GoDaddy :

```html
<div style="text-align:center; margin:30px 0;">

  <p style="font-family:sans-serif; font-size:15px; color:#555; margin-bottom:15px;">
    Réservation de véhicules et covoiturage — Résidence
  </p>

  <a href="https://VOTRE-NOM.github.io/autopartage"
     target="_blank"
     rel="noopener"
     style="
       display:inline-block;
       background:#2d5a3d;
       color:white;
       padding:16px 36px;
       border-radius:10px;
       text-decoration:none;
       font-family:sans-serif;
       font-size:17px;
       font-weight:bold;
       letter-spacing:0.03em;
       box-shadow:0 4px 14px rgba(45,90,61,0.35);
     ">
    🚗 Accéder à AutoPartage
  </a>

  <p style="font-family:sans-serif; font-size:12px; color:#999; margin-top:10px;">
    Connexion requise · Accès réservé aux résidents
  </p>

</div>
```

> ⚠️ Remplacez **`VOTRE-NOM`** par votre nom d'utilisateur GitHub (ex: `jean-tremblay`).

---

#### Option B — Iframe intégrée (plateforme visible dans la page)

La plateforme s'affiche directement dans votre site GoDaddy.

```html
<div style="width:100%; max-width:1280px; margin:20px auto;">

  <h2 style="font-family:sans-serif; text-align:center; margin-bottom:15px; color:#2d5a3d;">
    🚗 AutoPartage — Résidence
  </h2>

  <iframe
    src="https://VOTRE-NOM.github.io/autopartage"
    width="100%"
    height="900px"
    frameborder="0"
    style="
      border-radius:14px;
      box-shadow:0 6px 30px rgba(0,0,0,0.12);
      display:block;
    "
    title="AutoPartage — Réservation véhicules et covoiturage"
    allow="same-origin">
  </iframe>

</div>
```

> ⚠️ Remplacez **`VOTRE-NOM`** par votre nom d'utilisateur GitHub.
>
> Note : certains navigateurs peuvent bloquer les iframes. Si la plateforme n'apparaît pas, utilisez l'Option A.

---

### Étape 13 · Sauvegarder et publier sur GoDaddy

1. Sauvegardez vos modifications dans l'éditeur GoDaddy
2. Cliquez **Publier** (ou l'équivalent selon votre interface)
3. Ouvrez votre site en navigation privée pour tester
4. Cliquez le bouton AutoPartage
5. ✅ La plateforme s'ouvre et vous pouvez vous connecter

---

## PARTIE 5 — GESTION QUOTIDIENNE

---

### Créer un compte résident
**Admin → 👥 Résidents → + Créer un compte**
Remplissez : appartement, nom, email, mot de passe temporaire → communiquez-les au résident en main propre.

### Recharger un solde
**Admin → 💳 Portefeuille → Créditer un résident**
Entrez le montant en $ CA. Le règlement physique se fait hors plateforme.

### Valider un paiement covoiturage
**Admin → ⚙️ Admin → Paiements à valider**
Le conducteur confirme d'abord, puis l'admin clique **✓ Valider** pour créditer le conducteur.

### Ajouter un trajet prédéfini
**Admin → ⚙️ Admin → Trajets prédéfinis → + Ajouter**

### Modifier les tarifs
**Admin → ⚙️ Admin → Tarification → Enregistrer**

### Mettre un véhicule en maintenance
**Admin → 🚗 Véhicules → Maintenance**

---

## PARTIE 6 — DÉPANNAGE

| Problème | Solution |
|----------|----------|
| "Connexion…" bloqué | Vérifiez `config.js` : URL et clé anon sans espaces |
| "Email ou mot de passe incorrect" | Recréez le compte admin (Étape 4) |
| Erreur RLS | Relancez `setup.sql` en entier dans Supabase SQL Editor |
| Page blanche GitHub Pages | Le fichier doit s'appeler exactement `index.html` |
| GitHub Pages inactif | Vérifiez Étape 9 : Settings → Pages → Branch main |
| Premier accès lent | Normal — Supabase gratuit dort après 7j d'inactivité |
| Calcul distance échoue | Réessayez ou créez un trajet prédéfini avec distance manuelle |
| Iframe bloquée | Utilisez l'Option A (lien bouton) |
| Solde insuffisant | Admin doit créditer le résident via Portefeuille |

---

## PARTIE 7 — MISE À JOUR

Pour mettre à jour l'application avec une nouvelle version :

1. **github.com** → votre dépôt → cliquez `index.html`
2. Cliquez le crayon ✏️ **(Edit this file)**
3. **Ctrl+A** pour tout sélectionner → supprimez → collez le nouveau contenu
4. Cliquez **Commit changes**
5. ⏳ Mise à jour en ligne en ~2 minutes automatiquement

> `config.js` n'a jamais besoin d'être modifié après l'Étape 6.

---

## Récapitulatif final

| # | Étape | Durée | Fait ? |
|---|-------|-------|--------|
| 1 | Créer compte Supabase | 2 min | ☐ |
| 2 | Créer projet Supabase | 3 min | ☐ |
| 3 | Exécuter setup.sql | 3 min | ☐ |
| 4 | Créer compte admin SQL | 2 min | ☐ |
| 5 | Copier les clés API | 2 min | ☐ |
| 6 | Modifier config.js | 2 min | ☐ |
| 7 | Créer dépôt GitHub | 3 min | ☐ |
| 8 | Uploader index.html + config.js | 3 min | ☐ |
| 9 | Activer GitHub Pages | 3 min | ☐ |
| 10 | Tester la connexion | 3 min | ☐ |
| 11 | Ouvrir zone HTML GoDaddy | 5 min | ☐ |
| 12 | Coller le code bouton/iframe | 3 min | ☐ |
| 13 | Publier et tester GoDaddy | 3 min | ☐ |

**Total : ~45 minutes**

---

## URLs importantes à conserver

```
Application AutoPartage : https://VOTRE-NOM.github.io/autopartage
Tableau de bord Supabase : https://supabase.com/dashboard
Dépôt GitHub             : https://github.com/VOTRE-NOM/autopartage
```

---

*AutoPartage v2 — Guide GoDaddy · Résidence au Canada*
