# Cas d'utilisation — TogoMarket

---

## 1. Diagramme d'architecture

```mermaid
graph TB
    subgraph Client
        MOB[📱 App Mobile<br/>React Native / Expo]
        WEB[🖥️ App Web<br/>React / Vite]
    end

    subgraph Backend
        API[⚙️ API Symfony 7<br/>API Platform 4]
        DB[(🗄️ PostgreSQL 16)]
        UPLOAD[📁 Uploads<br/>VichUploader]
    end

    subgraph Services
        STRIPE[💳 Stripe<br/>Checkout Sessions]
        MAIL[📧 MailCatcher<br/>Notifications]
    end

    MOB -->|JWT + REST| API
    WEB -->|JWT + REST| API
    API --> DB
    API --> UPLOAD
    API -->|Checkout| STRIPE
    STRIPE -->|Webhook| API
    API -->|SMTP| MAIL
```

---

## 2. Diagramme des cas d'utilisation global

```mermaid
graph LR
    subgraph Visiteur
        V1[Consulter les produits]
        V2[Rechercher / filtrer]
        V3[Voir le detail d'un produit]
        V4[S'inscrire]
        V5[Se connecter]
    end

    subgraph Acheteur
        A1[Ajouter au panier<br/>avec couleur/taille]
        A2[Gerer le panier]
        A3[Passer commande<br/>paiement Stripe]
        A4[Suivre ses commandes]
        A5[Voir le code de livraison]
        A6[Gerer ses adresses<br/>+ GPS]
        A7[Voir la boutique<br/>d'un vendeur]
        A8[Modifier son profil]
    end

    subgraph Vendeur
        S1[Gerer ses produits<br/>couleurs/tailles/images]
        S2[Voir son tableau de bord]
        S3[Avancer le statut<br/>des commandes]
        S4[Confirmer livraison<br/>avec code client]
    end

    subgraph Admin
        AD1[Statistiques globales]
        AD2[Gerer les utilisateurs]
        AD3[Valider les vendeurs]
        AD4[Marquer commande payee]
    end
```

---

## 3. Cas d'utilisation par acteur

### 3.1 Visiteur (non connecte)

| # | Cas d'utilisation | Description | Preconditions | Postconditions |
|---|-------------------|-------------|---------------|----------------|
| CU-V01 | Consulter la page d'accueil | Le visiteur accede a la page d'accueil avec les categories, produits recents et recherche | Aucune | Page affichee |
| CU-V02 | Rechercher un produit | Le visiteur saisit un mot-cle dans la barre de recherche | Aucune | Liste filtree affichee |
| CU-V03 | Voir le detail d'un produit | Le visiteur clique sur un produit pour voir : galerie, prix, couleurs, tailles, vendeur, avis | Produit existant | Page detail affichee |
| CU-V04 | S'inscrire | Le visiteur cree un compte avec prenom, nom, email, mot de passe | Aucune | Compte ROLE_BUYER cree, connexion auto |
| CU-V05 | Se connecter | Le visiteur se connecte avec email et mot de passe | Compte existant et actif | Token JWT, acces a l'app |

---

### 3.2 Acheteur (ROLE_BUYER)

| # | Cas d'utilisation | Description | Preconditions | Postconditions |
|---|-------------------|-------------|---------------|----------------|
| CU-A01 | Selectionner les variantes | L'acheteur choisit la couleur (dot colore) et la taille (chip) sur la page detail | Produit avec variantes | Variantes selectionnees |
| CU-A02 | Ajouter au panier | L'acheteur ajoute un produit avec quantite, couleur et taille | Connecte, variantes selectionnees (si dispo), stock suffisant | Article ajoute au panier |
| CU-A03 | Gerer le panier | L'acheteur modifie les quantites ou supprime des articles | Panier non vide | Panier mis a jour |
| CU-A04 | Gerer ses adresses | L'acheteur ajoute/modifie/supprime des adresses de livraison | Connecte | Adresse sauvegardee |
| CU-A05 | Utiliser la geolocalisation | L'acheteur clique sur le bouton GPS pour remplir automatiquement rue, ville, pays | Permission localisation | Champs remplis automatiquement |
| CU-A06 | Passer commande | L'acheteur selectionne une adresse et paye via Stripe | Panier non vide, adresse existante | Commande(s) creee(s), panier vide |
| CU-A07 | Consulter ses commandes | L'acheteur voit la liste de ses commandes avec statut | Connecte | Liste affichee |
| CU-A08 | Voir le detail d'une commande | L'acheteur voit la timeline, les articles, l'adresse et le total | Commande existante | Detail affiche |
| CU-A09 | Voir le code de livraison | Quand sa commande est expediee, l'acheteur voit le code a 6 chiffres | Commande en statut "shipped" | Code affiche |
| CU-A10 | Communiquer le code au livreur | L'acheteur donne le code au livreur/vendeur pour confirmer la reception | Code visible | Livraison confirmable |
| CU-A11 | Voir la boutique d'un vendeur | L'acheteur clique sur le vendeur dans la page detail pour voir tous ses produits | Page detail produit | Page boutique affichee |
| CU-A12 | Modifier son profil | L'acheteur modifie prenom, nom, telephone | Connecte | Profil mis a jour |

---

### 3.3 Vendeur (ROLE_SELLER)

| # | Cas d'utilisation | Description | Preconditions | Postconditions |
|---|-------------------|-------------|---------------|----------------|
| CU-S01 | Voir le tableau de bord | Le vendeur consulte ses stats : produits, commandes, revenus | Connecte vendeur | Dashboard affiche |
| CU-S02 | Creer un produit | Le vendeur remplit : titre, description, prix (FCFA), stock, categorie, couleurs, tailles, images | Connecte vendeur | Produit cree, slug auto-genere |
| CU-S03 | Modifier un produit | Le vendeur modifie un de ses produits | Proprietaire du produit | Produit mis a jour |
| CU-S04 | Supprimer un produit | Le vendeur supprime un produit apres confirmation | Proprietaire du produit | Produit et images supprimes |
| CU-S05 | Uploader des images | Le vendeur ajoute des photos depuis la galerie ou la camera | Proprietaire du produit | Images uploadees |
| CU-S06 | Gerer les couleurs | Le vendeur selectionne les couleurs disponibles parmi 12 predefinies | Creation/edition produit | Couleurs enregistrees |
| CU-S07 | Gerer les tailles | Le vendeur selectionne les tailles disponibles (vetements ou chaussures) | Creation/edition produit | Tailles enregistrees |
| CU-S08 | Voir ses commandes | Le vendeur consulte les commandes contenant ses produits | Connecte vendeur | Liste filtree par seller |
| CU-S09 | Lancer la preparation | Le vendeur passe une commande payee en "En preparation" | Commande en statut "paid" | Statut → processing |
| CU-S10 | Marquer comme expediee | Le vendeur passe en "Expediee" | Statut "processing" | Statut → shipped, **code livraison genere** |
| CU-S11 | Confirmer la livraison | Le vendeur saisit le code a 6 chiffres donne par le client | Statut "shipped" | Si code correct → statut "delivered" |

---

### 3.4 Administrateur (ROLE_ADMIN)

| # | Cas d'utilisation | Description | Preconditions | Postconditions |
|---|-------------------|-------------|---------------|----------------|
| CU-AD01 | Voir les statistiques | L'admin consulte les stats globales : utilisateurs, produits, commandes, revenus | Connecte admin | Dashboard affiche |
| CU-AD02 | Gerer les utilisateurs | L'admin active/desactive des comptes | Connecte admin | Compte modifie |
| CU-AD03 | Valider un vendeur | L'admin approuve ou refuse un profil vendeur | Profil en attente | Statut mis a jour |
| CU-AD04 | Gerer les categories | L'admin cree/modifie/supprime des categories | Connecte admin | Categorie modifiee |
| CU-AD05 | Marquer commande payee | L'admin passe une commande "pending" en "paid" (paiement hors ligne) | Commande en attente | Statut → paid |

---

## 4. Flux de commande (processus complet)

```mermaid
sequenceDiagram
    actor Client as 🛒 Acheteur
    participant Cart as Panier
    participant API as API Backend
    participant Stripe as 💳 Stripe
    actor Vendeur as 🏪 Vendeur

    Note over Client,Vendeur: PHASE 1 — Achat

    Client->>Cart: Ajoute produits<br/>(couleur, taille, quantite)
    Client->>API: POST /checkout/create-session
    Note over API: Split panier par vendeur<br/>→ N commandes (status: pending)
    API->>Stripe: Cree Checkout Session<br/>(1 paiement unique)
    Stripe-->>Client: Page de paiement
    Client->>Stripe: Paye
    Stripe->>API: Webhook: payment confirmed
    Note over API: Toutes commandes → paid<br/>Stock decremente<br/>Panier vide

    Note over Client,Vendeur: PHASE 2 — Preparation

    Vendeur->>API: PATCH /orders/{id}<br/>status: processing
    Note over Vendeur: "Lancer preparation"

    Note over Client,Vendeur: PHASE 3 — Expedition

    Vendeur->>API: PATCH /orders/{id}<br/>status: shipped
    Note over API: Code livraison 6 chiffres<br/>genere automatiquement
    Client->>API: GET /orders/{id}/delivery-code
    Note over Client: Voit le code: 472831

    Note over Client,Vendeur: PHASE 4 — Livraison

    Client-->>Vendeur: Communique le code: 472831
    Vendeur->>API: POST /orders/{id}/confirm-delivery<br/>code: 472831
    Note over API: Code verifie ✅<br/>Status → delivered
```

---

## 5. Flux de commande multi-vendeurs

```mermaid
sequenceDiagram
    actor Sophie as 🛒 Sophie
    participant API as API Backend
    participant Stripe as 💳 Stripe
    actor Kofi as 🏪 Kofi<br/>(TechLome)
    actor Ama as 🏪 Ama<br/>(AfroStyle)

    Sophie->>API: Panier: iPhone (Kofi) + Robe (Ama)
    Sophie->>API: POST /checkout/create-session

    Note over API: Split par vendeur:<br/>Commande #1 → Kofi (iPhone)<br/>Commande #2 → Ama (Robe)

    API->>Stripe: 1 seul paiement (total)
    Stripe-->>Sophie: Paye une seule fois
    Stripe->>API: Webhook → les 2 commandes passent en "paid"

    Note over Kofi,Ama: Chaque vendeur gere independamment

    Kofi->>API: Commande #1 → processing → shipped
    Note over API: Code livraison: 384721
    Sophie->>Kofi: Code: 384721
    Kofi->>API: confirm-delivery(384721) ✅

    Note over Ama: 3 jours plus tard...

    Ama->>API: Commande #2 → processing → shipped
    Note over API: Code livraison: 159462
    Sophie->>Ama: Code: 159462
    Ama->>API: confirm-delivery(159462) ✅
```

---

## 6. Scenarios detailles

### CU-V04 : S'inscrire

| Element | Detail |
|---------|--------|
| **Acteur** | Visiteur |
| **Precondition** | Aucune |
| **Scenario principal** | 1. Le visiteur clique sur "Creer un compte" 2. Il remplit : prenom, nom, email, mot de passe 3. Le systeme verifie l'unicite de l'email 4. Le compte est cree avec ROLE_BUYER 5. Le systeme connecte automatiquement l'utilisateur 6. Redirection vers l'accueil |
| **Scenario alternatif** | 3a. L'email existe deja → message d'erreur |
| **Postcondition** | Compte cree, utilisateur connecte |

### CU-A02 : Ajouter au panier

| Element | Detail |
|---------|--------|
| **Acteur** | Acheteur |
| **Precondition** | Connecte, page detail produit |
| **Scenario principal** | 1. L'acheteur selectionne une couleur (si disponible) 2. Il selectionne une taille (si disponible) 3. Il choisit la quantite 4. Il clique sur "Ajouter" 5. Le systeme verifie le stock 6. L'article est ajoute au panier avec les variantes |
| **Scenario alternatif** | 1a. Couleur requise mais non selectionnee → alerte "Veuillez choisir une couleur" 2a. Taille requise mais non selectionnee → alerte "Veuillez choisir une taille" 5a. Stock insuffisant → erreur "Stock insuffisant" 6a. Meme produit+couleur+taille deja dans le panier → quantites fusionnees |
| **Postcondition** | Article dans le panier |

### CU-A06 : Passer commande (multi-vendeurs)

| Element | Detail |
|---------|--------|
| **Acteur** | Acheteur |
| **Precondition** | Panier non vide, au moins une adresse |
| **Scenario principal** | 1. L'acheteur va au checkout 2. Il selectionne une adresse de livraison 3. Il clique sur "Payer" 4. Le systeme groupe les articles par vendeur 5. 1 commande par vendeur est creee (status: pending) 6. 1 session Stripe est creee avec le total 7. L'acheteur est redirige vers la page Stripe 8. Apres paiement, le webhook met toutes les commandes en "paid" 9. Le stock est decremente, le panier vide |
| **Scenario alternatif** | 5a. Stock insuffisant pour un produit → erreur avec nom du produit |
| **Postcondition** | N commandes creees, panier vide |

### CU-S11 : Confirmer la livraison avec code

| Element | Detail |
|---------|--------|
| **Acteur** | Vendeur |
| **Precondition** | Commande en statut "shipped" |
| **Scenario principal** | 1. Le vendeur clique sur "Confirmer livraison" 2. Un modal s'ouvre avec un champ de saisie 6 chiffres 3. Le vendeur demande le code au client 4. Il saisit le code et clique "Confirmer" 5. Le systeme verifie le code cote serveur 6. Si correct : commande → "delivered", code efface |
| **Scenario alternatif** | 5a. Code incorrect → erreur "Code de livraison incorrect" 4a. Code incomplet (moins de 6 chiffres) → bouton desactive |
| **Postcondition** | Commande livree, code supprime |

### CU-AD05 : Marquer commande payee (admin)

| Element | Detail |
|---------|--------|
| **Acteur** | Administrateur |
| **Precondition** | Connecte admin, commande en statut "pending" |
| **Scenario principal** | 1. L'admin voit une commande en attente de paiement 2. Il clique sur "Marquer payee" 3. Confirmation demandee 4. Le statut passe a "paid" |
| **Cas d'usage** | Paiement en especes, virement bancaire, mobile money — le paiement est confirme hors Stripe |
| **Postcondition** | Commande payee, vendeur peut commencer la preparation |

---

## 7. Diagramme d'etats — Commande

```mermaid
stateDiagram-v2
    [*] --> pending : Commande creee
    pending --> paid : Webhook Stripe /<br/>Admin (CU-AD05)
    pending --> cancelled : Timeout / annulation
    paid --> processing : Vendeur (CU-S09)
    processing --> shipped : Vendeur (CU-S10)<br/>→ Code livraison genere
    shipped --> delivered : Vendeur + Code client (CU-S11)
    delivered --> [*]
    cancelled --> [*]
```

---

## 8. Diagramme d'etats — Code de livraison

```mermaid
stateDiagram-v2
    [*] --> Inexistant : Commande creee
    Inexistant --> Genere : Status → shipped<br/>(6 chiffres aleatoires)
    Genere --> Visible_acheteur : GET /delivery-code<br/>(seul le proprietaire)
    Visible_acheteur --> Communique : Acheteur → Livreur<br/>(oral / SMS)
    Communique --> Verifie : POST /confirm-delivery
    Verifie --> Valide : Code correct ✅<br/>→ Status delivered
    Verifie --> Refuse : Code incorrect ❌<br/>→ Nouvelle tentative
    Refuse --> Communique
    Valide --> Supprime : Code efface de la BDD
    Supprime --> [*]
```
