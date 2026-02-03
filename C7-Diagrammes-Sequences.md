# C7 - Diagrammes de Séquence des Scénarios Majeurs

## Objectif
Représenter **plusieurs diagrammes de séquence majeurs** modélisant les flux d'interactions critiques du système, couvrant les scénarios principaux identifiés dans les itérations IT1 à IT4.

---

## 1. Diagrammes de Séquence des Scénarios Majeurs

### 1.1. Scénario 1 : Prise de Commande avec Recommandation Vin (IT1 - UC Majeur)

**Contexte** : Un serveur prend une commande incluant un plat principal, le système suggère automatiquement un vin adapté.

**Acteurs** :
- **Serveur** (via app mobile React Native)
- **Backend API** (Node.js)
- **PostgreSQL** (SGBD)
- **Redis** (Cache)
- **ERP QuiCuisineIci** (système externe)

**Flux nominal** :

```mermaid
sequenceDiagram
    actor Serveur
    participant Mobile as Mobile App<br/>(React Native)
    participant SQLite as SQLite<br/>(Offline Cache)
    participant API as Backend API<br/>(Node.js)
    participant Redis as Redis<br/>(Cache)
    participant DB as PostgreSQL<br/>(BDD)
    participant ERP as ERP Cuisine<br/>(QuiCuisineIci)

    Note over Serveur,ERP: UC1.2 + UC2.1 : Commande + Recommandation Vin

    %% Phase 1: Consultation Menu
    Serveur->>+Mobile: Ouvre écran Menu
    Mobile->>+SQLite: SELECT menu_items (cache local)
    
    alt Cache local disponible (offline/récent)
        SQLite-->>Mobile: Retour menu local
        Mobile->>Serveur: Affichage menu (immédiat)
    else Cache expiré ou vide
        Mobile->>+API: GET /api/menu
        API->>+Redis: GET menu:full
        
        alt Cache hit Redis
            Redis-->>API: Menu JSON (TTL 5min)
            API-->>Mobile: 200 OK + Menu
        else Cache miss
            API->>+DB: SELECT menu_items<br/>JOIN wines ON wine_pairing_id
            DB-->>-API: Résultat (50 plats + vins associés)
            API->>Redis: SET menu:full (TTL 5min)
            API-->>-Mobile: 200 OK + Menu
        end
        
        Mobile->>SQLite: UPDATE menu cache local
        SQLite-->>-Mobile: Cache mis à jour
        Mobile->>Serveur: Affichage menu + suggestions vin
    end

    %% Phase 2: Sélection Plat + Recommandation
    Serveur->>Mobile: Sélectionne "Magret de canard"
    Mobile->>Mobile: Détection catégorie="plat principal"
    
    Note over Mobile: Logique métier locale:<br/>Si plat principal → Afficher vin recommandé
    
    Mobile->>Serveur: Affiche carte avec:<br/>🍷 "Cahors 2018" suggéré<br/>(accord mets-vin automatique)

    %% Phase 3: Validation Commande
    Serveur->>Mobile: Ajoute magret + vin au panier
    Serveur->>Mobile: Valide commande (Table 5)
    
    Mobile->>+API: POST /api/orders<br/>Body: {table_id: 5,<br/>   items: [{dish_id: 12, qty: 1},<br/>           {wine_id: 34, qty: 1}]}
    
    %% Authentification & Validation
    API->>API: Middleware: JWT validation
    API->>API: Middleware: RBAC check (role=serveur)
    API->>API: Middleware: Joi schema validation
    
    %% Business Logic
    API->>+DB: BEGIN TRANSACTION
    API->>DB: INSERT INTO orders<br/>(table_id, status, total_amount)
    DB-->>API: order_id = 456
    
    API->>DB: INSERT INTO order_items<br/>(order_id, menu_item_id, quantity)
    DB-->>API: 2 lignes insérées
    
    API->>DB: UPDATE tables SET status='occupied'
    DB-->>-API: OK
    
    %% Envoi ERP
    API->>+ERP: POST /orders<br/>Body: {order_id: 456,<br/>   items: [{dish: "Magret", qty: 1}]}
    
    alt ERP disponible
        ERP-->>API: 201 Created<br/>{erp_order_id: "K789"}
        API->>DB: UPDATE orders<br/>SET erp_id='K789', status='sent'
    else ERP indisponible (Circuit Breaker)
        ERP--xAPI: Timeout (5s)
        Note over API: Circuit Breaker OPEN<br/>(Opossum)
        API->>DB: UPDATE orders<br/>SET status='pending_erp'
        API->>API: Queue retry (exponential backoff)
    end
    
    %% Invalidation Cache
    API->>Redis: DEL tables:status<br/>(invalidation cache plan salle)
    
    API->>DB: COMMIT
    API-->>-Mobile: 201 Created<br/>{order_id: 456, status: "sent"}
    
    Mobile->>SQLite: INSERT order (offline backup)
    Mobile->>Serveur: ✅ Commande validée<br/>Table 5 : Magret + Cahors

    Note over Serveur,ERP: Temps total: ~150ms (cache hit)<br/>ou ~400ms (cache miss + ERP)
```

**Points Clés** :
- ✅ **Cache multi-niveaux** : SQLite (offline) → Redis → PostgreSQL
- ✅ **Recommandation vin automatique** : JOIN SQL `menu_items.wine_pairing_id`
- ✅ **Résilience ERP** : Circuit breaker Opossum (retry exponential backoff)
- ✅ **Sécurité** : JWT + RBAC middleware pipeline
- ✅ **Performance** : P50 ~150ms (cache hit), P95 ~400ms (cache miss)

---

### 1.2. Scénario 2 : Split Bill / Paiement Divisé (IT1 - UC Critique)

**Contexte** : Une table de 4 personnes souhaite payer individuellement (chacun sa part).

**Acteurs** :
- **Caissier** (interface web React.js)
- **Backend API**
- **PostgreSQL** (transactions ACID)
- **TPE Bancaire** (système externe isolé VLAN)

**Flux nominal** :

```mermaid
sequenceDiagram
    actor Caissier
    participant Web as Interface Caisse<br/>(React.js)
    participant API as Backend API<br/>(Node.js)
    participant DB as PostgreSQL<br/>(BDD)
    participant TPE as TPE Bancaire<br/>(VLAN 10 isolé)

    Note over Caissier,TPE: UC4.3 : Split Bill par Couvert

    %% Phase 1: Consultation Addition
    Caissier->>+Web: Ouvre addition Table 12
    Web->>+API: GET /api/orders/789/bill
    
    API->>+DB: SELECT orders, order_items<br/>WHERE order_id=789<br/>FOR UPDATE (row-level lock)
    DB-->>-API: Order details:<br/>{total: 102€,<br/> covers: 4,<br/> items: [...]}
    
    API-->>-Web: 200 OK + Bill details
    Web->>Caissier: Affiche:<br/>Total: 102€<br/>4 couverts

    %% Phase 2: Calcul Split
    Caissier->>Web: Clique "Split Bill"
    Web->>Web: UI Interactive:<br/>Sélection mode division
    Caissier->>Web: Choisit "Par personne" (égal)
    
    Web->>Web: Calcul local (SplitBillService):<br/>102€ / 4 = 25.50€/personne
    
    Web->>Caissier: Affiche 4 boutons:<br/>💳 Personne 1: 25.50€<br/>💳 Personne 2: 25.50€<br/>💳 Personne 3: 25.50€<br/>💳 Personne 4: 25.50€

    %% Phase 3: Paiement Individuel (Personne 1)
    Caissier->>Web: Clique "Payer Personne 1"
    Web->>+API: POST /api/payments/split<br/>Body: {order_id: 789,<br/>   cover_number: 1,<br/>   amount: 25.50,<br/>   method: "CB"}
    
    API->>API: JWT + RBAC check (role=caissier)
    API->>API: Joi validation (amount > 0)
    
    %% Transaction ACID
    API->>+DB: BEGIN TRANSACTION ISOLATION SERIALIZABLE
    
    %% Vérification double paiement
    API->>DB: SELECT * FROM payments<br/>WHERE order_id=789<br/>AND cover_number=1<br/>FOR UPDATE
    
    alt Déjà payé
        DB-->>API: Row exists (status='paid')
        API->>DB: ROLLBACK
        API-->>Web: 409 Conflict<br/>"Personne 1 déjà payée"
        Web->>Caissier: ❌ Erreur: Déjà payé
    else Pas encore payé
        DB-->>API: No rows
        
        %% Insertion paiement
        API->>DB: INSERT INTO payments<br/>(order_id, cover_number,<br/> amount, method, status)<br/>VALUES (789, 1, 25.50,<br/> 'CB', 'pending')
        DB-->>API: payment_id = 1234
        
        %% Appel TPE
        API->>+TPE: VLAN 10 Request:<br/>Montant: 25.50€<br/>Transaction ID: 1234
        
        alt TPE Succès
            TPE-->>API: Approval Code: ABC123<br/>Card: ****1234
            API->>DB: UPDATE payments<br/>SET status='paid',<br/>    approval_code='ABC123'
        else TPE Refusé/Timeout
            TPE--xAPI: Declined / Timeout
            API->>DB: UPDATE payments<br/>SET status='failed'
            API->>DB: ROLLBACK
            API-->>Web: 402 Payment Required<br/>"Carte refusée"
            Web->>Caissier: ❌ Paiement refusé
        end
        
        %% Vérification si addition complète
        API->>DB: SELECT COUNT(*)<br/>FROM payments<br/>WHERE order_id=789<br/>AND status='paid'
        DB-->>API: count = 1 (sur 4 attendus)
        
        Note over API: Logique: 1/4 payé<br/>→ Commande encore ouverte
        
        API->>DB: COMMIT
        API-->>-Web: 201 Created<br/>{payment_id: 1234,<br/> remaining: 3/4}
        
        Web->>Caissier: ✅ Personne 1 payée<br/>Reste 3/4 (76.50€)
    end

    %% Phase 4: Paiements suivants (Personnes 2, 3, 4)
    Note over Caissier,TPE: Répétition pour couverts 2, 3, 4...

    %% Dernier paiement (Personne 4)
    Caissier->>Web: Clique "Payer Personne 4"
    Web->>+API: POST /api/payments/split<br/>(cover_number: 4)
    
    API->>+DB: BEGIN TRANSACTION
    API->>DB: INSERT payment (cover 4)
    API->>+TPE: Transaction 25.50€
    TPE-->>-API: Approved
    API->>DB: UPDATE payment status='paid'
    
    %% Vérification addition complète
    API->>DB: SELECT COUNT(*)<br/>WHERE order_id=789<br/>AND status='paid'
    DB-->>API: count = 4 (4/4 ✅)
    
    Note over API: Logique: Addition complète<br/>→ Clôture commande
    
    API->>DB: UPDATE orders<br/>SET status='paid_complete',<br/>    paid_at=NOW()
    
    API->>DB: COMMIT
    API-->>-Web: 201 Created<br/>{status: "order_complete"}
    
    Web->>Caissier: ✅ Addition Table 12 PAYÉE<br/>4/4 paiements validés

    Note over Caissier,TPE: Temps total: ~8s (4 paiements TPE)<br/>ACID garantit 0% double paiement
```

**Points Clés** :
- ✅ **ACID Serializable** : `FOR UPDATE` + transaction isolée → Impossible de payer 2 fois
- ✅ **Row-level locking PostgreSQL** : Concurrence safe (2 caissiers simultanés OK)
- ✅ **VLAN isolé TPE** : Conformité PCI DSS (données carte ≠ app)
- ✅ **Idempotence** : Vérification `SELECT cover_number` avant `INSERT`
- ✅ **Logique métier** : Auto-détection addition complète (COUNT=covers)

---

### 1.3. Scénario 3 : Mode Offline + Synchronisation (IT3 - Résilience Critique)

**Contexte** : Perte WiFi pendant prise de commande, puis reconnexion → Synchronisation automatique.

**Acteurs** :
- **Serveur** (app mobile)
- **SQLite local**
- **Backend API** (accessible après reconnexion)

**Flux nominal** :

```mermaid
sequenceDiagram
    actor Serveur
    participant Mobile as Mobile App<br/>(React Native)
    participant NetInfo as NetInfo<br/>(Détecteur réseau)
    participant SQLite as SQLite Local<br/>(Offline Storage)
    participant SyncQueue as SyncQueue<br/>(File d'attente)
    participant API as Backend API<br/>(Node.js)
    participant DB as PostgreSQL

    Note over Serveur,DB: UC7.1 : Mode Offline-First

    %% Phase 1: Détection perte réseau
    Serveur->>+Mobile: Prend commande Table 8
    Mobile->>+NetInfo: Check connectivity
    NetInfo-->>-Mobile: ❌ WiFi: DISCONNECTED

    Note over Mobile: Mode Offline AUTO-ACTIVÉ<br/>Badge UI rouge "Hors ligne"

    Mobile->>Serveur: 🔴 Badge "Mode Hors Ligne"

    %% Phase 2: Sauvegarde locale
    Serveur->>Mobile: Ajoute plats au panier
    Serveur->>Mobile: Valide commande Table 8
    
    Mobile->>Mobile: Génère UUID local:<br/>order_uuid="offline-123-abc"
    
    Mobile->>+SQLite: INSERT INTO orders_offline<br/>(uuid, table_id, items,<br/> status, created_at)<br/>VALUES ('offline-123-abc',<br/> 8, '[...]', 'pending_sync',<br/> NOW())
    SQLite-->>-Mobile: Commande sauvegardée localement
    
    Mobile->>+SyncQueue: Enqueue(order_uuid)<br/>(Priorité: HIGH)
    SyncQueue-->>-Mobile: Ajouté à la file (position 1)
    
    Mobile->>Serveur: ✅ Commande enregistrée<br/>⚠️ Sera envoyée à la reconnexion

    %% Phase 3: Tentatives sync pendant offline
    loop Toutes les 30s
        SyncQueue->>+NetInfo: Check connectivity
        NetInfo-->>-SyncQueue: ❌ Toujours offline
        Note over SyncQueue: Exponential backoff:<br/>30s → 60s → 120s...
    end

    %% Phase 4: Reconnexion
    Note over Serveur,DB: 5 minutes plus tard...

    NetInfo->>Mobile: 🟢 Event: WiFi CONNECTED
    Mobile->>Serveur: 🟢 Badge "En ligne"
    
    Mobile->>SyncQueue: Trigger sync NOW
    
    %% Phase 5: Synchronisation
    SyncQueue->>+SQLite: SELECT * FROM orders_offline<br/>WHERE status='pending_sync'<br/>ORDER BY created_at ASC
    SQLite-->>-SyncQueue: 3 commandes en attente:<br/>['offline-123-abc',<br/> 'offline-124-def',<br/> 'offline-125-ghi']
    
    loop Pour chaque commande en file
        SyncQueue->>+API: POST /api/orders/sync<br/>Body: {uuid: 'offline-123-abc',<br/>   table_id: 8,<br/>   items: [...],<br/>   offline_timestamp: T0}
        
        alt API Success
            API->>+DB: INSERT orders<br/>(sync correlation_id)
            DB-->>-API: order_id = 999
            API-->>SyncQueue: 201 Created<br/>{order_id: 999,<br/> server_timestamp: T1}
            
            SyncQueue->>SQLite: UPDATE orders_offline<br/>SET status='synced',<br/>    server_order_id=999
            
            Note over SyncQueue: Latence sync = T1 - T0<br/>(ex: 2.3s)
        else API Fail (conflit, erreur)
            API--xSyncQueue: 409 Conflict<br/>"Table déjà occupée"
            SyncQueue->>SQLite: UPDATE status='sync_failed',<br/>    error='Conflict Table 8'
            SyncQueue->>Mobile: Notification serveur:<br/>"⚠️ Conflit commande Table 8"
        end
    end
    
    SyncQueue->>Mobile: ✅ Sync complète:<br/>3/3 commandes envoyées
    Mobile->>Serveur: Notification:<br/>"Synchronisation terminée"

    Note over Serveur,DB: Total sync time: 2.3s (3 commandes)
```

**Points Clés** :
- ✅ **Offline-First** : SQLite = source de vérité locale (mode online OU offline)
- ✅ **Auto-détection** : NetInfo listener → Pas de bouton manuel
- ✅ **Exponential backoff** : 30s → 60s → 120s (économie batterie)
- ✅ **Gestion conflits** : API détecte `table_id` déjà occupé → Retour erreur explicite
- ✅ **UUID corrélation** : `offline-123-abc` permet traçabilité sync
- ✅ **Performance** : Sync moyenne **2.3s** pour 3 commandes (objectif IT3 <5s ✅)

---

### 1.4. Scénario 4 : Clôture Journalière NF525 (IT2 - Obligation Légale)

**Contexte** : En fin de journée, le caissier lance la clôture Z (obligation légale NF525).

**Acteurs** :
- **Caissier**
- **Backend API**
- **PostgreSQL** (table immuable `audit_logs`)
- **Module NF525** (génération hash + signature RSA)

**Flux nominal** :

```mermaid
sequenceDiagram
    actor Caissier
    participant Web as Interface Caisse<br/>(React.js)
    participant API as Backend API<br/>(Node.js)
    participant NF525 as NF525Service<br/>(Module Certifié)
    participant DB as PostgreSQL<br/>(audit_logs immuable)
    participant Crypto as Node Crypto<br/>(SHA-256 + RSA)

    Note over Caissier,Crypto: UC5.2 : Clôture Z NF525

    %% Phase 1: Déclenchement
    Caissier->>+Web: Clique "Clôture Journée"
    Web->>Caissier: Confirmation:<br/>"Clôturer service du 02/02/2026 ?"
    Caissier->>Web: Confirme
    
    Web->>+API: POST /api/nf525/close<br/>Body: {date: "2026-02-02"}
    
    API->>API: JWT + RBAC (role=caissier/admin)
    
    %% Phase 2: Récupération transactions jour
    API->>+DB: SELECT * FROM payments<br/>WHERE DATE(paid_at)='2026-02-02'<br/>AND status='paid'<br/>ORDER BY paid_at ASC
    
    DB-->>-API: 87 transactions:<br/>[{payment_id: 1001, amount: 25.50},<br/> {payment_id: 1002, amount: 42.00},<br/> ...]
    
    %% Phase 3: Calcul totaux
    API->>+NF525: calculateDailySummary(transactions)
    NF525->>NF525: Groupement par méthode:<br/>CB: 2 145.30€<br/>Espèces: 387.50€<br/>TR: 124.00€
    NF525-->>-API: Summary:<br/>{total_ttc: 2656.80€,<br/> total_tva: 265.68€,<br/> count: 87}
    
    %% Phase 4: Génération Hash Chaîné
    API->>+DB: SELECT hash_current<br/>FROM audit_logs<br/>ORDER BY id DESC LIMIT 1
    DB-->>-API: hash_previous="5f3a..."<br/>(dernier hash de la veille)
    
    API->>+NF525: generateHash(summary, hash_previous)
    NF525->>+Crypto: SHA-256(<br/>  date="2026-02-02" +<br/>  total_ttc=2656.80 +<br/>  hash_previous="5f3a..."<br/>)
    Crypto-->>-NF525: hash_current="a7b2..."
    NF525-->>-API: hash_current="a7b2..."
    
    %% Phase 5: Signature RSA
    API->>+NF525: signRSA(hash_current, private_key)
    NF525->>+Crypto: RSA-SHA256 Sign<br/>Key: 2048 bits (certificat NF525)
    Crypto-->>-NF525: signature="MIIBIj..."<br/>(Base64, 344 chars)
    NF525-->>-API: signature="MIIBIj..."
    
    %% Phase 6: Insertion Immuable
    API->>+DB: BEGIN TRANSACTION
    API->>DB: INSERT INTO audit_logs<br/>(transaction_type,<br/> amount, date,<br/> hash_current,<br/> hash_previous,<br/> signature,<br/> metadata)<br/>VALUES<br/>('CLOSURE_Z',<br/> 2656.80, '2026-02-02',<br/> 'a7b2...', '5f3a...',<br/> 'MIIBIj...',<br/> '{count: 87, ...}')
    
    DB-->>API: audit_log_id = 5678<br/>✅ Insertion OK
    
    Note over DB: Trigger immuable ACTIF:<br/>BEFORE UPDATE/DELETE<br/>→ RAISE EXCEPTION
    
    API->>DB: UPDATE payments<br/>SET audit_log_id=5678<br/>WHERE DATE(paid_at)='2026-02-02'
    DB-->>API: 87 rows updated
    
    API->>DB: COMMIT
    
    %% Phase 7: Génération Ticket Z
    API->>+NF525: generateTicketZ(summary, signature)
    NF525->>NF525: Template ASCII:<br/>┌─────────────────────┐<br/>│ TICKET Z 02/02/2026│<br/>│ Total TTC: 2656.80€│<br/>│ Hash: a7b2...      │<br/>│ Sign: MIIBIj...    │<br/>└─────────────────────┘
    NF525-->>-API: ticket_text (ASCII art)
    
    API-->>-Web: 200 OK<br/>{audit_log_id: 5678,<br/> ticket: "...",<br/> hash: "a7b2...",<br/> signature: "MIIBIj..."}
    
    Web->>Caissier: ✅ Clôture Z validée<br/>Affichage ticket imprimable
    
    %% Phase 8: Impression (optionnel)
    Caissier->>Web: Clique "Imprimer Ticket Z"
    Web->>Web: window.print() → Imprimante thermique
    
    Note over Caissier,Crypto: Obligations NF525 VALIDÉES:<br/>✅ Hash chaîné SHA-256<br/>✅ Signature RSA certificat<br/>✅ Immuabilité audit_logs<br/>✅ Ticket Z imprimable

    %% Phase 9: Tentative modification (TEST)
    Note over API,DB: TEST: Modification interdite ?
    
    API->>DB: UPDATE audit_logs<br/>SET amount=9999<br/>WHERE id=5678
    DB-->>API: ❌ ERROR: "Modification<br/>audit_logs interdite (NF525)"<br/>(Trigger BEFORE UPDATE)
    
    Note over DB: ✅ Immuabilité garantie<br/>par PostgreSQL trigger
```

**Points Clés** :
- ✅ **Hash chaîné SHA-256** : Chaque clôture référence hash précédent → Détection altération
- ✅ **Signature RSA 2048 bits** : Certificat NF525 → Preuve légale
- ✅ **Trigger PostgreSQL immuable** : `BEFORE UPDATE/DELETE → RAISE EXCEPTION`
- ✅ **Ticket Z imprimable** : Obligation légale (conservation 6 ans)
- ✅ **Metadata JSON** : Détails transactions (CB/Espèces/TR) pour audit fiscal

---

### 1.5. Scénario 5 : Notification Temps Réel "Plat Prêt" (IT2 - WebSocket)

**Contexte** : Cuisine termine un plat, notification push instantanée au serveur assigné.

**Acteurs** :
- **ERP Cuisine** (QuiCuisineIci)
- **Backend API** (Socket.io WebSocket)
- **Mobile serveur** (React Native)

**Flux nominal** :

```mermaid
sequenceDiagram
    participant ERP as ERP Cuisine<br/>(QuiCuisineIci)
    participant API as Backend API<br/>(Node.js + Socket.io)
    participant Redis as Redis<br/>(Pub/Sub)
    participant Mobile1 as Mobile Serveur 1<br/>(Table 5)
    participant Mobile2 as Mobile Serveur 2<br/>(Table 8)
    participant Mobile3 as Mobile Serveur 3<br/>(Table 12)

    Note over ERP,Mobile3: UC8.1 : Notification Plat Prêt

    %% Phase 1: Connexion WebSocket initiale
    Note over Mobile1,Mobile3: Démarrage app (matin)
    
    Mobile1->>+API: WebSocket CONNECT<br/>Auth: JWT token
    API->>API: Extract user_id from JWT<br/>user_id=1 (Serveur Alice)
    API->>API: socket.join('user:1')
    API-->>Mobile1: ✅ Connected (session_id: abc123)
    
    Mobile2->>+API: WebSocket CONNECT (JWT)
    API->>API: user_id=2 (Serveur Bob)
    API->>API: socket.join('user:2')
    API-->>Mobile2: ✅ Connected (session_id: def456)
    
    Mobile3->>+API: WebSocket CONNECT (JWT)
    API->>API: user_id=3 (Serveur Charlie)
    API->>API: socket.join('user:3')
    API-->>Mobile3: ✅ Connected (session_id: ghi789)
    
    Note over API: State actuel:<br/>Room 'user:1' → Mobile1<br/>Room 'user:2' → Mobile2<br/>Room 'user:3' → Mobile3

    %% Phase 2: Callback ERP (Plat prêt)
    Note over ERP,Mobile3: 15h23 : Plat terminé en cuisine
    
    ERP->>+API: POST /api/webhooks/erp/dish-ready<br/>Body: {<br/>  erp_order_id: "K789",<br/>  dish_name: "Magret canard",<br/>  table_id: 5<br/>}
    
    API->>API: JWT validation (ERP secret token)
    
    %% Récupération infos commande
    API->>API: DB Query:<br/>SELECT * FROM orders<br/>WHERE erp_id='K789'
    API->>API: Result:<br/>order_id=456,<br/>table_id=5,<br/>server_user_id=1 (Alice)
    
    %% Phase 3: Broadcasting WebSocket
    Note over API: Logique routing:<br/>Plat Table 5 → Serveur Alice (user:1)
    
    API->>Redis: PUBLISH channel:dish_ready<br/>{order_id: 456, table: 5,<br/> dish: "Magret", server: 1}
    
    Redis->>API: Pub/Sub propagation
    
    API->>API: socket.to('user:1').emit(<br/>  'dish_ready',<br/>  {table: 5,<br/>   dish: "Magret canard",<br/>   timestamp: "15h23"})<br/>)
    
    API-->>Mobile1: 🔔 WebSocket Event:<br/>'dish_ready'<br/>{table: 5, dish: "Magret"}
    
    Note over Mobile2,Mobile3: Pas de notification<br/>(pas leur table)
    
    API-->>-ERP: 200 OK<br/>{status: "notified"}
    
    %% Phase 4: Affichage Mobile
    Mobile1->>Mobile1: React Native:<br/>Trigger local notification
    
    Mobile1->>Mobile1: Display Push:<br/>🔔 "Plat prêt Table 5"<br/>"Magret de canard"<br/>(Vibration + Son)
    
    Note over Mobile1: Serveur Alice voit notification<br/>même si app en arrière-plan

    %% Phase 5: Confirmation serveur (optionnel)
    Mobile1->>+API: WebSocket EMIT<br/>'dish_acknowledged'<br/>{order_id: 456}
    
    API->>API: DB: UPDATE orders<br/>SET acknowledged_at=NOW()
    
    API-->>-Mobile1: ACK received
    
    Note over ERP,Mobile3: Temps total notification:<br/>ERP → Mobile = ~80ms<br/>(WebSocket temps réel)

    %% Phase 6: Déconnexion (fin service)
    Note over Mobile1: 23h : Fin de service
    
    Mobile1->>API: WebSocket DISCONNECT
    API->>API: socket.leave('user:1')
    API->>API: Cleanup session abc123
    
    Note over API: Connexions actives:<br/>Mobile2 (user:2) ✅<br/>Mobile3 (user:3) ✅
```

**Points Clés** :
- ✅ **WebSocket bi-directionnel** : API ↔ Mobile (pas de polling HTTP)
- ✅ **Room isolation** : `socket.join('user:X')` → Notification ciblée serveur
- ✅ **Redis Pub/Sub** : Broadcast multi-instances Node.js (cluster PM2)
- ✅ **Push notification native** : React Native local notification (vibration + son)
- ✅ **Latence <100ms** : ERP callback → Mobile notification (~80ms)
- ✅ **Résilience** : Reconnexion auto WebSocket si perte réseau

---

## Conclusion

Les **5 diagrammes de séquence majeurs** de ce document couvrent les flux critiques :

1. ✅ **Commande + Vin** (IT1) : Cache multi-niveaux + résilience ERP
2. ✅ **Split Bill** (IT1) : ACID + row-locking → 0% double paiement
3. ✅ **Offline Sync** (IT3) : SQLite queue + exponential backoff
4. ✅ **Clôture NF525** (IT2) : Hash chaîné + signature RSA + trigger immuable
5. ✅ **Notification WebSocket** (IT2) : Temps réel <100ms + Redis Pub/Sub

Chaque diagramme inclut :
- **Phases détaillées** du flux (consultation, validation, synchronisation...)
- **Gestion des erreurs** (alt/else avec scénarios d'échec)
- **Métriques de performance** (P50, P95, latence)
- **Points clés** techniques (patterns, justifications)

---

## Documents Complémentaires C7

Ce document fait partie d'une série de 3 documents C7 :

1. ✅ **C7-Diagrammes-Sequences.md** (ce document) : Diagrammes de séquence des scénarios majeurs
2. ✅ **C7-MCD.md** : Modèle Conceptuel de Données (8 entités + contraintes intégrité)
3. ⏳ **C7-Interactions-Environnement.md** : Diagramme interactions avec systèmes externes

---

## Prochaines Étapes

1. ⏳ **C7-Interactions-Environnement.md** : Diagramme de contexte C4 + interactions systèmes externes
2. ⏳ **C8** : Plan de tests TDD (unitaires, intégration, E2E)
3. ⏳ **C9** : Cahier des charges final (intégration C1-C8)
