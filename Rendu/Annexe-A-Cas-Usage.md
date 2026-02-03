# Annexe A : Cas d'Usage Détaillés

*Ce document est un complément détaillé du Cahier des Charges Principal*

---

## Introduction

Cette annexe détaille les **13 cas d'usage majeurs** du système de gestion de restaurant, organisés par acteur et phase du service. Chaque UC comprend les préconditions, flots nominaux, alternatifs et postconditions.

---

## A.1. Phase Réservation

### A.1.1. UC - Gérer les Réservations

**Acteurs** :  Responsable de Salle, Caissier

**Préconditions** :
- Utilisateur authentifié avec rôle approprié
- Application web Dashboard accessible

**Flot Nominal** :
1. Utilisateur accède à l'écran "Réservations"
2. Système affiche planning journalier avec disponibilités
3. Utilisateur sélectionne "Nouvelle Réservation"
4. Système affiche formulaire :
   - Nom client (obligatoire)
   - Téléphone (obligatoire, format validé)
   - Date réservation (obligatoire)
   - Heure arrivée (créneaux 15 min)
   - Nombre personnes (1-20)
   - Demandes spécifiques (optionnel) : Allergies, PMR, Chaise bébé
5. Utilisateur remplit formulaire
6. **Contrainte RGPD** : Système affiche checkbox consentement collecte données
7. Utilisateur valide consentement
8. Système vérifie disponibilité table adaptée
9. Système enregistre réservation avec statut "Confirmée"
10. Système génère ID réservation unique
11. Système affiche confirmation avec ID

**Flots Alternatifs** :

**Alt 1 : Pas de table disponible**
- 8a. Aucune table disponible pour capacité demandée
- 8b. Système affiche message "Complet pour ce créneau"
- 8c. Système propose créneaux alternatifs (±30 min)
- 8d. Retour étape 4

**Alt 2 : Consentement refusé**
- 7a. Utilisateur ne coche pas consentement
- 7b. Système empêche validation
- 7c. Système affiche "Consentement obligatoire pour traitement réservation"

**Alt 3 : Modification réservation**
- 1a. Utilisateur sélectionne réservation existante
- 1b. Système charge données
- 1c. Utilisateur modifie champs
- 1d. Suite flot nominal étape 8

**Alt 4 : Droit à l'oubli (RGPD)**
- 1a. Utilisateur sélectionne "Supprimer réservation"
- 1b. Système affiche confirmation "Données seront anonymisées"
- 1c. Utilisateur confirme
- 1d. Système anonymise données personnelles (Nom → "ANONYME", Tel → "XXXXXXXXXX")
- 1e. Système conserve métadonnées (Date, Heure, Nb personnes) pour statistiques

**Postconditions** :
- Réservation enregistrée en base `RESERVATIONS`
- Consentement RGPD tracé avec timestamp
- Email/SMS confirmation envoyé (si optionnel activé)

**Contraintes Juridiques** :
- **RGPD Art. 6** : Consentement explicite requis
- **RGPD Art. 17** : Droit à l'oubli implémenté (anonymisation)
- **Archivage** : Données anonymisées conservées 2 ans (statistiques CA)

---

## A.2. Phase Accueil

### A.2.1. UC - Accueillir et Assigner Table

**Acteur** : Responsable de Salle

**Préconditions** :
- Responsable authentifié
- Au moins 1 table libre

**Flot Nominal** :
1. Responsable accède écran "Plan de Salle"
2. Système affiche plan 20 tables temps réel :
   - **Vert** : Libre
   - **Rouge** : Occupée
   - **Orange** : Réservée (prochaine 30 min)
3. Client arrive (avec ou sans réservation)
4. Responsable identifie groupe (nombre personnes)
5. Responsable sélectionne table adaptée (capacité ≥ nb personnes)
6. Système vérifie disponibilité table
7. Système affiche liste serveurs disponibles
8. Responsable assigne serveur
9. Système change statut table "Libre" → "Occupée"
10. Système notifie serveur assigné (notification mobile)
11. Système enregistre timestamp ouverture service

**Flots Alternatifs** :

**Alt 1 : Réservation existante**
- 3a. Client annonce nom réservation
- 3b. Responsable recherche réservation dans système
- 3c. Système affiche détails réservation (Nom, Heure, Nb personnes, Demandes spécifiques)
- 3d. Responsable note demandes spécifiques (ex: allergie gluten)
- 3e. Suite flot nominal étape 4

**Alt 2 : Aucune table libre de capacité adaptée**
- 6a. Toutes tables adaptées occupées
- 6b. Système affiche temps attente estimé (moyenne durée repas = 90 min)
- 6c. Responsable propose attente ou retour
- 6d. Si attente : Système crée "fil d'attente" avec priorité

**Alt 3 : Table réservée non réclamée (+15 min retard)**
- 2a. Table statut "Réservée" dépasse 15 min heure réservation
- 2b. Système affiche alerte "Réservation non honorée"
- 2c. Responsable peut libérer table manuellement
- 2d. Système change statut "Réservée" → "Libre"

**Postconditions** :
- Table statut "Occupée"
- Serveur assigné et notifié
- Session service démarrée (timestamp)

**Règles Métier** :
- Table capacité 2 peut accueillir 1-2 personnes
- Table capacité 4 peut accueillir 2-4 personnes (tolérance -50%)
- Interdiction assigner table capacité 2 pour groupe 4 personnes

---

## A.3. Phase Commande et Service

### A.3.1. UC - Prendre Commande

**Acteur** : Serveur (Mobile React Native)

**Préconditions** :
- Serveur authentifié sur application mobile
- Table assignée au serveur (statut "Occupée")
- Connexion réseau (ou mode offline activé)

**Flot Nominal** :
1. Serveur ouvre application mobile
2. Système affiche liste tables assignées
3. Serveur sélectionne table
4. Système affiche écran commande :
   - Nombre couverts (choix 1-12)
   - Catégories menu (Entrées/Plats/Desserts/Vins)
5. Serveur sélectionne catégorie "Plats"
6. **Contrainte Stocks** : Système affiche plats avec disponibilité temps réel (cache Redis 30s, source ERP)
   - Disponible : Vert
   - Stock faible (<5) : Orange
   - Rupture : Grisé
7. Serveur sélectionne plat principal (ex: "Magret de Canard")
8. **Fonctionnalité Recommandation Vin** : Système déclenche algorithme accord mets-vins :
   - Lecture table `WINES` colonne `wine_pairing_id` lié au plat
   - Affichage suggestion : "🍷 Accord recommandé : Cahors AOC 2019"
9. Serveur présente recommandation client
10. Client accepte/refuse
11. **Si accepte** : Serveur ajoute vin commande
12. Serveur répète étapes 5-11 pour chaque couvert
13. Serveur révise commande écran récapitulatif
14. Serveur valide commande
15. Système calcule prix total prévi sionnel
16. **Envoi ERP** : Système envoie commande à ERP cuisine via REST API
    ```json
    POST /api/v2/orders
    {
      "table_number": 5,
      "items": [
        {"dish_id": 42, "quantity": 2, "notes": "Sans oignons"},
        {"dish_id": 15, "quantity": 1}
      ],
      "server_id": "USR-123"
    }
    ```
17. ERP répond `201 Created` avec `erp_order_id`
18. Système enregistre commande en base `ORDERS` avec statut "En préparation"
19. Système affiche confirmation "Commande envoyée cuisine"

**Flots Alternatifs** :

**Alt 1 : Plat en rupture de stock**
- 7a. Serveur sélectionne plat grisé
- 7b. Système affiche popup "Rupture stock momentanée"
- 7c. Système propose alternatives catégorie similaire
- 7d. Retour étape 6

**Alt 2 : Client refuse recommandation vin**
- 10a. Client refuse
- 10b. Serveur peut rechercher manuellement autre vin (catégorie "Vins")
- 10c. Suite flot nominal étape 12

**Alt 3 : Modification commande (avant validation)**
- 13a. Serveur détecte erreur
- 13b. Serveur sélectionne item à modifier/supprimer
- 13c. Système met à jour récapitulatif
- 13d. Retour étape 13

**Alt 4 : Mode Offline (pas de réseau)**
- 16a. Appel ERP échoue (timeout >5s ou pas de réseau)
- 16b. Système bascule mode offline
- 16c. Système enregistre commande dans SQLite local
- 16d. Système ajoute queue synchronisation
- 16e. Système affiche "📶 Mode Offline - Sync auto dès retour réseau"
- 16f. Suite flot nominal étape 18 (local)
- **Voir UC Mode Offline détaillé (A.7)**

**Alt 5 : Erreur ERP (500 Internal Server Error)**
- 16a. ERP répond `500` ou `503`
- 16b. **Circuit Breaker** détecte échec
- 16c. Système retente 2 fois (backoff exponentiel 1s, 2s)
- 16d. Si 3 échecs : Circuit Breaker passe état OPEN (30s)
- 16e. Système bascule mode offline (Alt 4)

**Postconditions** :
- Commande enregistrée en base locale ET ERP
- Statut "En préparation"
- `erp_order_id` tracé pour synchronisation
- Timestamp commande enregistré

**Règles Métier** :
- Prix commande = somme `unit_price` items au moment commande (pas prix menu actuel)
- Recommandation vin : 1 vin max par plat principal
- Cache stocks Redis TTL 30s (réduit appels API Stocks)

**Métriques Performance** :
- Latence API P95 <200ms (cible)
- Affichage menu <1s (cache hit)
- Recommandation vin <50ms (requête SQL indexée)

---

### A.3.2. UC - Consulter Stocks et Menu Temps Réel

**Acteur** : Serveur (Mobile)

**Préconditions** :
- Serveur authentifié
- Application mobile lancée

**Flot Nominal** :
1. Serveur ouvre onglet "Menu"
2. Système requête cache Redis clé `menu:items:*`
3. **Cache HIT** : Redis retourne données (TTL 30s)
4. Système affiche menu avec disponibilité :
   - Nom plat
   - Prix
   - Icône allergie (gluten, lactose...)
   - **Stock** : Disponible/Limité (<5)/Rupture
5. Serveur consulte catégorie (ex: "Desserts")
6. Système filtre affichage

**Flots Alternatifs** :

**Alt 1 : Cache MISS**
- 3a. Redis ne contient pas clé (expirée ou premier accès)
- 3b. Système requête API Stocks externe :
    ```http
    GET /api/stocks/availability
    Headers: X-API-Key: xxx
    ```
- 3c. API Stocks répond JSON :
    ```json
    {
      "items": [
        {"dish_id": 42, "available": true, "stock": 12},
        {"dish_id": 15, "available": false, "stock": 0}
      ]
    }
    ```
- 3d. Système enregistre dans Redis avec `SETEX menu:items:42 30 '{"available":true,"stock":12}'`
- 3e. Suite flot nominal étape 4

**Alt 2 : API Stocks indisponible**
- 3b. API Stocks timeout (>5s) ou erreur 500
- 3c. **Fallback** : Système lit disponibilité depuis dernière synchro base `MENU_ITEMS.last_stock_sync`
- 3d. Système affiche warning "⚠️ Stocks non actualisés (dernière mise à jour: 14:32)"
- 3e. Suite flot nominal étape 4

**Postconditions** :
- Menu affiché avec données fraîches
- Cache Redis alimenté (si MISS)

**Règles Métier** :
- Cache TTL 30s (compromis fraîcheur/performance)
- Fallback base si API externe down
- Stocks <5 : Alerte visuelle serveur (éviter commande dernières unités si gros groupe)

---

### A.3.3. UC - Recevoir Notification "Plat Prêt"

**Acteur** : Serveur (Mobile)

**Préconditions** :
- Serveur a passé commande
- Commande statut "En préparation"
- Application mobile ouverte (foreground ou background)

**Flot Nominal** :
1. **Cuisine** : Chef termine préparation plat
2. Chef marque commande "Prête" dans ERP cuisine
3. **ERP** : Système déclenche webhook vers backend :
    ```http
    POST /api/webhooks/order-ready
    Headers: 
      X-ERP-Signature: HMAC-SHA256(body, secret)
    Body:
    {
      "erp_order_id": "ERP-456",
      "table_number": 5,
      "items_ready": [42, 15],
      "timestamp": "2026-02-02T14:35:00Z"
    }
    ```
4. **Backend** : Système vérifie signature HMAC (sécurité)
5. Système identifie `order_id` local via mapping `erp_order_id`
6. Système identifie serveur assigné table via `ORDERS.server_id`
7. **Pub/Sub** : Système publie événement Redis channel `notifications:server:USR-123`
8. **WebSocket** : Système envoie notification temps réel au serveur :
    ```json
    {
      "type": "ORDER_READY",
      "table_number": 5,
      "items": ["Magret de Canard x2", "Tarte Tatin x1"],
      "timestamp": "2026-02-02T14:35:00Z"
    }
    ```
9. **Mobile** : Application affiche notification push :
   - Son : Bip discret
   - Banner : "🍽️ Table 5 : 3 plats prêts"
   - Badge app : Incrémente compteur
10. Serveur consulte détails notification
11. Système affiche liste plats prêts
12. Serveur se rend en cuisine
13. Serveur marque "Récupéré" dans app
14. Système met à jour statut commande "Servie"

**Flots Alternatifs** :

**Alt 1 : Application mobile fermée (background)**
- 9a. Application en arrière-plan Android
- 9b. Système envoie notification push via Firebase Cloud Messaging (FCM)
- 9c. Android affiche notification système
- 9d. Serveur tape notification
- 9e. Application s'ouvre sur détails commande

**Alt 2 : Serveur pas de réseau (offline)**
- 8a. WebSocket déconnecté (serveur hors zone WiFi)
- 8b. Backend enregistre notification dans Redis List `pending_notifications:USR-123`
- 8c. **Reconnexion** : Serveur revient en zone WiFi
- 8d. WebSocket se reconnecte automatiquement (Socket.io auto-reconnect)
- 8e. Backend vide Redis List et envoie toutes notifications en attente
- 8f. Suite flot nominal étape 9

**Alt 3 : Signature HMAC invalide**
- 4a. Signature webhook ne correspond pas
- 4b. Système rejette requête (401 Unauthorized)
- 4c. Système log tentative intrusion dans `AUDIT_LOGS`
- 4d. Alerte sécurité envoyée admin

**Postconditions** :
- Notification reçue par serveur
- Statut commande mis à jour "Servie"
- Temps préparation enregistré (métrique cuisine)

**Métriques Performance** :
- Latence notification <100ms (ERP → Mobile)
- WebSocket reconnexion <2s (si déconnexion)

**Règles Métier** :
- 1 notification par commande (pas de spam si plusieurs plats)
- Notification expire 30 min (si serveur ne consulte jamais)

---

## A.4. Phase Encaissement

### A.4.1. UC - Gérer Addition et Split Bill

**Acteur** : Caissier (Web React.js, poste fixe)

**Préconditions** :
- Caissier authentifié
- Table avec commandes statut "Servie"
- Clients demandent addition

**Flot Nominal** :
1. Caissier accède Dashboard "Encaissements"
2. Système affiche liste tables occupées avec totaux
3. Caissier sélectionne table (ex: Table 5)
4. Système affiche addition détaillée :
   - Liste items commandés (Plat, Quantité, Prix unitaire)
   - Sous-total HT
   - TVA (5.5% / 10% / 20% selon type plat)
   - **Total TTC**
   - Nombre couverts (ex: 4)
5. **Cas Split Bill** : Clients demandent paiement divisé par couvert
6. Caissier sélectionne "Paiement Divisé"
7. Système affiche écran :
   - 4 onglets : "Couvert 1", "Couvert 2", "Couvert 3", "Couvert 4"
   - Liste items commande
8. Caissier assigne chaque item à couvert :
   - Drag & drop item → Onglet couvert
   - OU : Saisie manuelle "Item X → Couvert Y"
9. **Items partagés** (ex: Bouteille vin partagée 4 personnes) :
   - Caissier sélectionne "Diviser item"
   - Système répartit prix équitablement : Prix / Nb couverts
   - Ex: Bouteille 40€ → 10€ par couvert
10. Système calcule sous-total par couvert :
    - Couvert 1 : 25€
    - Couvert 2 : 32€
    - Couvert 3 : 28€
    - Couvert 4 : 30€
    - **TOTAL** : 115€ (= total addition)
11. Système vérifie cohérence : Somme couverts = Total addition
12. Caissier valide répartition
13. Système affiche "Split Bill validé - Procéder paiements individuels"

**Flots Alternatifs** :

**Alt 1 : Paiement global (1 personne paie tout)**
- 5a. Client paie addition complète
- 5b. Caissier sélectionne "Paiement Global"
- 5c. Suite UC Encaisser (A.4.2)

**Alt 2 : Paiement tiers (1 personne paie pour X autres)**
- 5a. Client A paie pour lui + Client B
- 5b. Caissier groupe couverts :
   - Groupe 1 : Couvert 1 + Couvert 2 (Client A paie)
   - Groupe 2 : Couvert 3 (Client C paie)
   - Groupe 3 : Couvert 4 (Client D paie)
- 5c. Système calcule sous-totaux groupes
- 5d. Suite flot nominal étape 13

**Alt 3 : Erreur répartition (somme ≠ total)**
- 11a. Somme couverts ≠ Total addition (ex: oubli item)
- 11b. Système affiche erreur "Répartition incohérente : -5€ manquant"
- 11c. Système highlight items non assignés en rouge
- 11d. Retour étape 8

**Alt 4 : Modification commande après split (ajout dessert)**
- 1a. Serveur ajoute item après split validé
- 1b. Système invalide split existant
- 1c. Système affiche "⚠️ Commande modifiée - Refaire split"
- 1d. Caissier refait split (étapes 6-12)

**Postconditions** :
- Addition calculée avec TVA correcte
- Split bill validé (si applicable)
- Prêt pour encaissements individuels

**Règles Métier** :
- **Contrainte UK** : `PAYMENTS(order_id, cover_number)` UNIQUE (interdiction doublon paiement même couvert)
- Items partagés : Division équitable automatique
- TVA applicable : 5.5% (aliments), 10% (boissons non alcoolisées), 20% (alcools)

---

### A.4.2. UC - Encaisser Paiements

**Acteur** : Caissier (Web)

**Préconditions** :
- Addition validée (avec ou sans split)
- TPE bancaire allumé (si paiement CB)

**Flot Nominal - Paiement CB** :
1. Caissier sélectionne couvert à encaisser (ex: Couvert 1, 25€)
2. Caissier sélectionne mode "Carte Bancaire"
3. Système affiche montant sur écran TPE
4. Client insère carte dans TPE
5. **TPE** : Terminal dialogue avec banque (réseau VLAN 10 isolé, PCI DSS)
6. Client saisit code PIN
7. **TPE** : Terminal envoie requête autorisation banque
8. **Banque** : Répond "Autorisé" avec `approval_code` unique
9. TPE affiche "Paiement accepté"
10. TPE renvoie données au système :
     ```json
     {
       "status": "approved",
       "approval_code": "AUTH-789456",
       "card_last4": "1234",
       "amount": 25.00,
       "timestamp": "2026-02-02T15:10:00Z"
     }
     ```
11. Système enregistre paiement en base `PAYMENTS` :
     ```sql
     INSERT INTO payments (
       order_id, cover_number, method, amount, approval_code, card_last4
     ) VALUES (
       'ORD-123', 1, 'CB', 25.00, 'AUTH-789456', '1234'
     );
     ```
12. **Contrainte NF525** : Système enregistre transaction dans `AUDIT_LOGS` (immuable)
13. Système imprime ticket caisse CB
14. Système affiche "✅ Couvert 1 : Payé (CB)"

**Flot Nominal - Paiement Espèces** :
1. Caissier sélectionne couvert (25€)
2. Caissier sélectionne mode "Espèces"
3. Système affiche montant
4. Client donne espèces (ex: 30€)
5. Caissier saisit montant donné : 30€
6. Système calcule rendu monnaie : 5€
7. Système affiche "Rendu monnaie : 5€"
8. Caissier rend monnaie
9. Caissier valide encaissement
10. Système enregistre paiement `PAYMENTS` (method='CASH')
11. **NF525** : Transaction enregistrée `AUDIT_LOGS`
12. Système imprime ticket caisse Espèces
13. Système affiche "✅ Couvert 1 : Payé (Espèces)"

**Flot Nominal - Paiement Titre Restaurant (TR)** :
1. Caissier sélectionne mode "Titre Restaurant"
2. Système affiche montant
3. Client remet chèques TR
4. Caissier compte montant TR (ex: 2x 10€ = 20€)
5. Caissier saisit : "20€ TR"
6. **Complément** : Si montant TR < total (25€) :
   - Système calcule reste à payer : 5€
   - Caissier encaisse complément (CB ou Espèces)
   - Système enregistre 2 paiements distincts :
     - Paiement 1 : 20€ TR
     - Paiement 2 : 5€ CB
7. Système valide paiement complet
8. Suite étapes 11-13 (enregistrement + ticket)

**Flots Alternatifs** :

**Alt 1 : Paiement CB refusé**
- 8a. Banque répond "Refusé" (solde insuffisant, carte invalide...)
- 8b. TPE affiche "Paiement refusé"
- 8c. Système affiche "❌ Paiement échoué - Choisir autre mode"
- 8d. Retour étape 2

**Alt 2 : TPE hors ligne (panne réseau)**
- 5a. TPE ne répond pas (timeout >10s)
- 5b. Système détecte timeout
- 5c. Caissier affiche "⚠️ TPE indisponible"
- 5d. **Fallback** : Proposition paiement Espèces temporaire
- 5e. OU : Attente rétablissement connexion (max 30 min politique restaurant)

**Alt 3 : Tous couverts payés (fin table)**
- 14a. Caissier a encaissé tous 4 couverts
- 14b. Système vérifie `all_covers_paid()` trigger
- 14c. Trigger détecte : Somme paiements = Total addition
- 14d. Système change statut commande "Payée"
- 14e. Système libère table : Statut "Occupée" → "Libre"
- 14f. Système affiche "Table 5 libérée"

**Postconditions** :
- Paiement(s) enregistré(s) `PAYMENTS`
- Transaction(s) NF525 immutable `AUDIT_LOGS`
- Ticket(s) caisse imprimé(s)
- Table libérée si tous couverts payés

**Règles Métier** :
- **Interdiction** : Chèques bancaires refusés (politique restaurant)
- **Paiement CB** : `approval_code` obligatoire (contrainte `NOT NULL IF method='CB'`)
- **Paiement complet** : Trigger vérifie `SUM(payments.amount) = order.total`
- **Masquage données** : Logs contiennent `card_last4` seulement (RGPD), pas numéro complet

**Contraintes Sécurité** :
- TPE isolé VLAN 10 (firewall bloque VLAN 20 → VLAN 10)
- Communication TPE chiffrée TLS 1.3
- Données carte jamais stockées backend (PCI DSS compliance)

---

### A.4.3. UC - Imprimer Ticket Caisse / Note de Frais

**Acteur** : Caissier

**Préconditions** :
- Paiement(s) enregistré(s)
- Imprimante thermique allumée

**Flot Nominal** :
1. Système détecte paiement validé
2. Système génère ticket caisse automatiquement :
   ```
   ═══════════════════════════════════
        RESTAURANT LA BELLE TABLE
          123 Rue de Paris
           75001 Paris, France
      Tel: 01 23 45 67 89
   ═══════════════════════════════════
   
   Date: 02/02/2026 15:10:32
   Table: 5 | Serveur: Jean D.
   Caissier: Marie L.
   
   ───────────────────────────────────
   ADDITION
   ───────────────────────────────────
   1x Magret de Canard      18.50 €
   1x Cahors AOC 2019        12.00 €
   1x Café                    2.50 €
   ───────────────────────────────────
   Sous-total HT:            29.20 €
   TVA 5.5% (aliments):       1.02 €
   TVA 20% (alcool):          2.40 €
   ───────────────────────────────────
   TOTAL TTC:                32.62 €
   
   MODE PAIEMENT: CB ****1234
   AUTORISATION: AUTH-789456
   
   ═══════════════════════════════════
       ** Certifié NF525 **
   Hash: a3f5...d8e2
   Signature: RSA-2048
   Numéro Ticket: 2026-02-02-00542
   ═══════════════════════════════════
   
      Merci de votre visite !
       À très bientôt.
   ```
3. Système envoie ticket à imprimante thermique
4. Imprimante imprime ticket (3-5 secondes)
5. Caissier remet ticket client

**Flot Alternatif** :

**Alt 1 : Note de frais demandée**
- 1a. Client demande "Note de frais détaillée"
- 1b. Caissier sélectionne "Imprimer Note Frais"
- 1c. Système affiche formulaire :
   - Raison sociale entreprise (obligatoire)
   - Adresse entreprise
   - N° SIRET
- 1d. Caissier remplit formulaire
- 1e. Système génère note frais format légal :
   - Mentions obligatoires : SIRET restaurant, TVA détaillée, Date, Heure
   - **Numéro facture** unique : FACT-2026-00542
- 1f. Système imprime note frais
- 1g. **Archivage** : Système enregistre PDF note frais (`invoices/2026/02/FACT-2026-00542.pdf`)

**Alt 2 : Imprimante hors service**
- 4a. Imprimante papier vide ou bourrage
- 4b. Système détecte erreur imprimante
- 4c. Système affiche "⚠️ Imprimante HS"
- 4d. **Fallback** : Système envoie ticket par email (si client donne email)
- 4e. OU : Caissier note "Ticket envoyé ultérieurement"

**Postconditions** :
- Ticket imprimé et remis
- Note frais archivée (si demandée)
- Numéro ticket NF525 séquentiel enregistré

**Contrainte Juridique NF525** :
- Ticket contient **obligatoirement** :
  - Hash transaction
  - Signature électronique RSA
  - Numéro séquentiel unique
  - Mention "Certifié NF525"
- Archivage tickets **6 ans minimum**

---

## A.5. Phase Clôture Journalière

### A.5.1. UC - Clôture Journalière NF525 (Ticket Z)

**Acteur** : Caissier (fin service 23h)

**Préconditions** :
- Fin de journée (toutes tables payées)
- Caissier rôle "admin" ou "caissier"

**Flot Nominal** :
1. Caissier accède Dashboard "Clôture Journalière"
2. Système affiche résumé journée :
   - Nombre transactions : 87
   - CA Total TTC : 4523.50 €
   - Répartition modes paiement :
     - CB : 72 transactions, 3890.00 €
     - Espèces : 12 transactions, 520.50 €
     - TR : 3 transactions, 113.00 €
3. Caissier vérifie cohérence (compte caisse vs système)
4. Caissier sélectionne "Clôturer Journée"
5. **Calcul Hash NF525** : Système exécute :
   - Récupère hash clôture veille : `hash_previous` (depuis `AUDIT_LOGS.closure_hash` date = J-1)
   - Concatène données journée :
     ```
     data = "2026-02-02|87|4523.50|" + hash_previous
     ```
   - Calcule SHA-256 déterministe :
     ```javascript
     const crypto = require('crypto');
     const hash = crypto.createHash('sha256')
       .update(data)
       .digest('hex');
     // hash = "a3f5d8e2..."
     ```
6. **Signature RSA** : Système signe hash avec clé privée NF525 :
   ```javascript
   const signature = crypto.sign(
     'sha256',
     Buffer.from(hash),
     privateKey
   );
   // signature = "RSA-2048-SIGNATURE..."
   ```
7. **Immuabilité** : Système enregistre clôture dans `AUDIT_LOGS` :
   ```sql
   INSERT INTO audit_logs (
     event_type, closure_date, total_amount, 
     nb_transactions, closure_hash, signature, hash_previous
   ) VALUES (
     'DAILY_CLOSURE', '2026-02-02', 4523.50, 
     87, 'a3f5d8e2...', 'RSA-SIG...', 'b2e7c9f1...'
   );
   ```
8. **Trigger Immuabilité** : Trigger PostgreSQL vérifie interdiction UPDATE/DELETE sur `AUDIT_LOGS`
9. Système génère **Ticket Z** (PDF) :
   - Récapitulatif CA jour
   - Hash chaîné
   - Signature RSA
   - Graphique CA (optionnel)
10. Système imprime Ticket Z (2 exemplaires : caisse + archive)
11. Système affiche "✅ Clôture validée - Ticket Z imprimé"
12. **Archivage** : Système sauvegarde Ticket Z (`closures/2026/02/Z-2026-02-02.pdf`)

**Flots Alternatifs** :

**Alt 1 : Première clôture (J1 restaurant)**
- 5a. Aucun `hash_previous` (table `AUDIT_LOGS` vide)
- 5b. Système utilise `hash_previous = "GENESIS"` (valeur bootstrap)
- 5c. Suite flot nominal étape 6

**Alt 2 : Incohérence caisse physique vs système**
- 3a. Caissier compte caisse espèces : 500€
- 3b. Système affiche attendu : 520.50€
- 3c. Écart : -20.50€
- 3d. Système affiche alerte "⚠️ Écart caisse : -20.50€"
- 3e. Caissier saisit justification (ex: "Erreur rendu monnaie Table 12")
- 3f. Système enregistre écart + justification
- 3g. Suite flot nominal étape 4

**Alt 3 : Clôture déjà effectuée (double clôture interdite)**
- 1a. Caissier tente clôturer 2ème fois même jour
- 1b. Système détecte clôture existante date = aujourd'hui
- 1c. Système affiche "❌ Clôture déjà effectuée aujourd'hui"
- 1d. Système propose "Consulter Ticket Z existant"

**Postconditions** :
- Clôture enregistrée `AUDIT_LOGS` (IMMUTABLE)
- Hash chaîné calculé et signé RSA
- Ticket Z imprimé et archivé 6 ans
- Journée comptable fermée

**Règles Métier NF525** :
- Hash **déterministe** : Même input → Même hash (reproductibilité contrôle fiscal)
- Trigger **interdiction** : `BEFORE UPDATE/DELETE AUDIT_LOGS` → Exception
- Signature RSA : Clé privée sécurisée (HSM ou fichier chiffré PGP)
- Ticket Z : Obligatoire chaque jour ouverture restaurant

**Tests Unitaires Critiques** :
- Test hash déterminisme : 10 exécutions → 10 hash identiques
- Test trigger immuabilité : `UPDATE audit_logs` → Exception levée
- Test signature RSA : Vérification clé publique réussit

---

## A.6. Administration Système

### A.6.1. UC - Gérer Utilisateurs (CRUD)

**Acteur** : Administrateur

**Préconditions** :
- Admin authentifié avec rôle `ROLE_ADMIN`

**Flot Nominal - Création Utilisateur** :
1. Admin accède Dashboard "Gestion Utilisateurs"
2. Système affiche liste utilisateurs existants (serveurs, caissiers, admins)
3. Admin sélectionne "Créer Utilisateur"
4. Système affiche formulaire :
   - Nom complet (obligatoire)
   - Username (unique, obligatoire)
   - Email (format validé)
   - Rôle (dropdown : Serveur/Caissier/Admin)
   - Mot de passe temporaire (auto-généré)
5. Admin remplit formulaire
6. Système valide unicité username (contrainte UK `USERS.username`)
7. **Sécurité** : Système hash mot de passe avec Bcrypt (salt rounds=10)
8. Système enregistre utilisateur :
   ```sql
   INSERT INTO users (username, password_hash, role, email)
   VALUES ('jean.dupont', '$2b$10$...', 'serveur', 'jean@resto.fr');
   ```
9. Système envoie email bienvenue avec mot de passe temporaire
10. Système affiche "✅ Utilisateur créé - Email envoyé"

**Flot Nominal - Modification Utilisateur** :
1. Admin sélectionne utilisateur dans liste
2. Système affiche détails utilisateur
3. Admin modifie champs (ex: changement rôle "Serveur" → "Caissier")
4. Admin valide modification
5. Système met à jour base
6. **RBAC** : Système révoque ancien token JWT (force re-login)
7. Système affiche "✅ Utilisateur modifié"

**Flot Nominal - Suppression Utilisateur (Soft Delete)** :
1. Admin sélectionne utilisateur
2. Admin sélectionne "Désactiver Utilisateur"
3. Système affiche confirmation "Désactiver jean.dupont ?"
4. Admin confirme
5. Système effectue **soft delete** :
   ```sql
   UPDATE users SET is_active = false WHERE id = 'USR-123';
   ```
6. Utilisateur ne peut plus se connecter
7. Données historiques conservées (commandes, paiements passés liés à cet utilisateur)

**Postconditions** :
- Utilisateur créé/modifié/désactivé
- Hash mot de passe sécurisé (Bcrypt)
- Logs modifications enregistrés `AUDIT_LOGS`

**Règles Métier** :
- Soft delete (jamais `DELETE FROM users`) : Préserve intégrité référentielle
- Mot de passe : Minimum 8 caractères, 1 maj, 1 chiffre (validation Joi)
- Email unique (contrainte UK optionnelle)

---

### A.6.2. UC - Consulter Rapports CA

**Acteur** : Administrateur, Gérant

**Préconditions** :
- Utilisateur rôle `ROLE_ADMIN` ou `ROLE_MANAGER`
- Données historiques disponibles

**Flot Nominal** :
1. Utilisateur accède "Rapports & Statistiques"
2. Système affiche dashboard :
   - Filtres : Période (Jour/Semaine/Mois/Année)
   - Métriques : CA, Nb transactions, Ticket moyen, Top plats
3. Utilisateur sélectionne période "Mois de Janvier 2026"
4. Système exécute requête SQL agrégée :
   ```sql
   SELECT 
     DATE(created_at) as date,
     SUM(total_amount) as ca_jour,
     COUNT(*) as nb_orders,
     AVG(total_amount) as ticket_moyen
   FROM orders
   WHERE created_at BETWEEN '2026-01-01' AND '2026-01-31'
     AND status = 'paid'
   GROUP BY DATE(created_at)
   ORDER BY date;
   ```
5. Système affiche graphique CA évolution 31 jours
6. Système calcule métriques :
   - **CA Total mois** : 142 350.00 €
   - **Ca moyen/jour** : 4 591.00 €
   - **Nb transactions** : 2 687
   - **Ticket moyen** : 53.00 €
7. Utilisateur consulte "Top 10 Plats"
8. Système exécute requête :
   ```sql
   SELECT 
     menu_items.name,
     COUNT(*) as nb_commandes,
     SUM(order_items.unit_price * order_items.quantity) as ca_plat
   FROM order_items
   JOIN menu_items ON order_items.menu_item_id = menu_items.id
   WHERE order_items.created_at BETWEEN '2026-01-01' AND '2026-01-31'
   GROUP BY menu_items.name
   ORDER BY nb_commandes DESC
   LIMIT 10;
   ```
9. Système affiche tableau :
   | Plat | Nb Commandes | CA Plat |
   |:-----|:------------:|:-------:|
   | Magret de Canard | 456 | 8 436.00 € |
   | Burger Maison | 392 | 5 880.00 € |
   | ... | ... | ... |
10. Utilisateur exporte rapport (button "Export Excel")
11. Système génère fichier Excel avec graphiques
12. Système télécharge `Rapport_CA_Janvier_2026.xlsx`

**Postconditions** :
- Rapport CA généré avec métriques exactes
- Export Excel disponible

**Règles Métier** :
- Requêtes agrégées optimisées (index `created_at`, `status`)
- Données temps réel (pas cache pour rapports admin)

---

## A.7. Mode Offline et Synchronisation

### A.7.1. UC - Fonctionnement Mode Offline

**Acteur** : Serveur (Mobile)

**Préconditions** :
- Application mobile installée
- Pas de réseau WiFi disponible

**Flot Nominal** :
1. Serveur prend commande (UC A.3.1)
2. Système détecte réseau indisponible (étape 16 UC A.3.1)
3. **Bascule Offline** : Système active mode offline :
   - Icône 📶 affichée dans app
   - Message "Mode Offline - Commandes enregistrées localement"
4. Système enregistre commande dans **SQLite local** :
   ```sql
   -- Base SQLite mobile
   INSERT INTO offline_orders (
     id, table_number, items, server_id, created_at, sync_status
   ) VALUES (
     'offline-1675348800-a3f5', 5, 
     '[{"dish_id":42,"quantity":2}]', 
     'USR-123', '2026-02-02T14:30:00Z', 'pending'
   );
   ```
5. Système ajoute commande à **queue synchronisation** :
   ```sql
   INSERT INTO sync_queue (order_id, action, retry_count)
   VALUES ('offline-1675348800-a3f5', 'create_order', 0);
   ```
6. Serveur continue prendre autres commandes (toutes enregistrées SQLite)
7. **Retour réseau** : Serveur revient en zone WiFi (15 min plus tard)
8. Système détecte réseau disponible (ping backend réussit)
9. **Synchronisation auto** : Système lance processus sync :
   - Lecture `sync_queue` : 3 commandes pending
   - Pour chaque commande :
     - Requête POST `/api/orders` avec données SQLite
     - Backend enregistre commande PostgreSQL + ERP
     - Backend répond `201 Created` avec `server_order_id`
     - Système met à jour SQLite : `sync_status = 'synced'`, `server_order_id` enregistré
10. **Détection conflits** : Si commande existe déjà serveur (doublon) :
    - Backend répond `409 Conflict`
    - Système marque `sync_status = 'conflict'`
    - **Stratégie** : Système garde version serveur (authoritative)
    - Système supprime doublon local SQLite
11. Synchronisation terminée : 3/3 réussies
12. Système affiche "✅ Synchronisation complète - 3 commandes envoyées"
13. Système supprime données SQLite synchronisées (cleanup)

**Flots Alternatifs** :

**Alt 1 : Échec synchronisation (serveur down)**
- 9a. POST `/api/orders` timeout ou 500
- 9b. Système incrémente `retry_count`
- 9c. **Backoff exponentiel** : Attend 2^retry_count secondes (1s, 2s, 4s, 8s...)
- 9d. Si retry_count ≥ 5 : Abandon, admin notifié
- 9e. Données restent SQLite local (safe)

**Alt 2 : Conflit détecté (commande modifiée entre-temps)**
- 10a. Backend détecte commande même `table_number` + `created_at` proche (<1 min)
- 10b. Backend compare items :
   - Local : 2 plats
   - Serveur : 3 plats (serveur a ajouté dessert entre-temps)
- 10c. **Stratégie** : Merge automatique impossible
- 10d. Backend répond `409 Conflict` avec détails
- 10e. Système affiche notification serveur : "⚠️ Conflit Table 5 - Vérifier commande"
- 10f. Serveur consulte commande serveur (authoritative)
- 10g. Serveur corrige manuellement si besoin

**Alt 3 : Premier lancement app (pas de données SQLite)**
- 1a. Application jamais utilisée
- 1b. Système crée schéma SQLite :
   ```sql
   CREATE TABLE offline_orders (...);
   CREATE TABLE sync_queue (...);
   CREATE INDEX idx_sync_status ON offline_orders(sync_status);
   ```
- 1c. Suite flot nominal

**Postconditions** :
- Commandes offline synchronisées vers serveur
- Données SQLite nettoyées (si sync réussie)
- Mode online rétabli

**Règles Métier** :
- UUID offline format : `offline-{timestamp}-{random}` (évite collision)
- Serveur = authoritative (priorité en cas conflit)
- Synchronisation automatique (pas action manuelle serveur)
- Retry max 5 fois (évite boucle

 infinie)

**Métriques Performance** :
- Synchronisation 3 commandes <5s (cible)
- Détection réseau <2s (ping backend)

---

