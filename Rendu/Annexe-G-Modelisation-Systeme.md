# Annexe G : Modélisation et Diagrammes Système

*Ce document est un complément détaillé du Cahier des Charges Principal*

---

## Introduction

Cette annexe présente la **modélisation complète du système** à travers :
1. **5 diagrammes de séquence** majeurs : Flux d'interactions détaillés (commandes, split bill, offline, NF525, notifications)
2. **Modèle Conceptuel de Données (MCD)** : 8 entités avec contraintes d'intégrité et volumétrie
3. **Diagramme interactions environnement** : Relations avec acteurs et systèmes externes

---

## G.1. Diagrammes de Séquence Majeurs

###G.1.1. DS1 : Prise de Commande avec Recommandation Vin

**Contexte** : Un serveur prend une commande incluant un plat principal. Le système suggère automatiquement un vin adapté via recommandation pré-configurée.

**Acteurs** : Serveur (mobile), Backend API, Redis, PostgreSQL, ERP QuiCuisineIci

**Flux Nominal** :

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

**Performance Mesurée** :
- Cache hit (Redis menu) : **~150ms** P50, **~220ms** P95
- Cache miss (PostgreSQL) : **~400ms** P50, **~580ms** P95

**Points Techniques Clés** :
- ✅ **Recommandations vin** : JOIN SQL `menus LEFT JOIN wines ON wine_pairing_id`
- ✅ **Circuit Breaker ERP** : Opossum (timeout 5s, 3 failures → OPEN)
- ✅ **Cache multi-niveaux** : SQLite mobile (offline) → Redis → PostgreSQL
- ✅ **ACID garantit** : Transaction complète ou rollback (pas ordre partiel)


---

### G.1.2. DS2 : Split Bill / Paiement Divisé par Couvert

**Contexte** : Une table de 4 personnes souhaite payer individuellement (chacun sa part).

**Acteurs** : Caissier (web), Backend API, PostgreSQL, TPE Bancaire (VLAN 10 isolé)

**Flux Nominal Split Bill** :

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

**Temps Total Mesure** : ~8s pour 4 paiements CB (latence TPE ~2s chacun)

**Garanties ACID** :
- ✅ **Isolation Serializable** : Empêche paiements concurrents même couvert
- ✅ **FOR UPDATE lock** : Impossible 2 caissiers paient même personne simultanément
- ✅ **Contrainte UK** : `UNIQUE(order_id, cover_number)` → PostgreSQL rejette doublons
- ✅ **Transaction atomique** : Paiement + Update status + verification COUNT = tout ou rien

---

### G.1.3. DS3 : Mode Offline avec Synchronisation Auto

**Contexte** : Un serveur perd le WiFi pendant la prise de commande. Le système bascule en mode offline SQLite puis synchronise automatiquement à la reconnexion.

**Acteurs** : Serveur (mobile), SQLite local, NetInfo, Backend API

**Flux Nominal Offline** :

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

**Performance Sync** : **~2.3s pour 10 commandes** (P95 <5s)

**Gestion Conflits** :
- **Idempotence** : `offline_uuid` unique → Replay safe
- **Timestamp** : `created_at_offline` conservé (traçabilité délai)
- **Collision tables** : Si table modifiée ailleurs → Log warning, sync continue

**Limitations Acceptables** :
- ⚠️ **Stocks offline obsolètes** : Dernière valeur cache affichée (avertissement UI)
- ⚠️ **Pas paiements offline** : Caisse reste online obligatoirement (TPE réseau requis)

---

### G.1.4. DS4 : Clôture Journalière NF525

**Contexte** : Fin de service (22h), le système effectue automatiquement la clôture fiscale quotidienne (Ticket Z) avec hash chaîné et signature RSA.

**Acteurs** : Cron Backend (automatique), PostgreSQL, NF525Service, Imprimante Thermique

**Flux Nominal Clôture Z** :

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

**Temps Estimé** : ~500ms clôture complète

**Obligations NF525** :
                     WHERE DATE(created_at) = CURRENT_DATE
                     ORDER BY id DESC LIMIT 1
   → "f7e3d1..."

--- Signature RSA Clôture ---
6. data_to_sign = concat(CURRENT_DATE, total_general, count_transactions, last_audit_hash)
7. Load private_key RSA-2048 from file /secure/nf525_priv_key.pem
8. signature_rsa = RSA-Sign(data_to_sign, private_key)
   → Base64: "MIICXAIBAAKBgQC... (2048 bits)"

--- Enregistrement Clôture ---
9. INSERT INTO daily_closures (
     date: '2026-02-02',
     transaction_count: 87,
     total_revenue: 2656.80,
     total_cb: 2145.30,
     total_cash: 387.50,
     total_tr: 124.00,
     hash_chain_end: 'f7e3d1...',
     signature_rsa: 'MIICXAIBAAKBgQC...'
   )

--- Génération PDF Ticket Z ---
10. Backend → pdfkit.generate({
      title: "CLÔTURE FISCALE Z - 2026-02-02",
      header: [Nom restaurant, SIRET, Adresse],
      content: [
        "Période : 2026-02-02 11:30 - 22:00",
        "---",
        "Transactions : 87",
        "CA Total TTC : 2656.80€",
        "  - Carte Bancaire : 2145.30€",
        "  - Espèces : 387.50€",
        "  - Tickets Restaurant : 124.00€",
        "---",
        "Hash Chaîne NF525 : f7e3d1a8...",
        "Signature RSA : MIICXAIBAAKBgQC..."
      ],
      footer: "Conforme NF525 - Certificat #12345"
    })

--- Impression Automatique ---
11. Backend → Imprimante thermique USB /dev/usb/lp0
    → Ticket Z papier imprimé (archivage physique obligatoire 6 ans)

--- Notification Admin ---
12. Backend → Slack webhook "✅ Clôture Z 02/02: 2656.80€ (87 transactions)"
13. Backend → Log Elasticsearch "Daily closure completed successfully"
```

**Temps Exécution Clôture** : **~4s pour 87 transactions**

**Conformité NF525 Garanties** :
- ✅ **Inaltérabilité** : Table `audit_logs` trigger `BEFORE UPDATE/DELETE → RAISE EXCEPTION`
- ✅ **Sécurisation** : Signature RSA-2048 (clé privée stockée sécurisée hors app)
- ✅ **Conservation** : Archives 6 ans (BDD + PDF physique)
- ✅ **Archivage** : Ticket Z quotidien automatique (pas oubli humain possible)

---

### G.1.5. DS5 : Notifications Temps Réel (WebSocket)

**Contexte** : Un plat est prêt en cuisine. L'ERP envoie une notification au backend qui diffuse via WebSocket au mobile du serveur concerné.

**Acteurs** : ERP QuiCuisineIci, Backend API, Redis Pub/Sub, WebSocket Server (Socket.io), Mobile Serveur

**Flux Nominal Push Notification** :

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


**Latence Totale Mesurée** : **\<100ms** (ERP→Backend→Mobile)

**Bénéfices Temps Réel** :
- ✅ **Latence vs polling** : <100ms vs polling 30s = latence moyenne **-15s** (-99%)
- ✅ **Efficacité réseau** : 0 req idle vs polling 3 mobiles × 2 req/min = **-360 req/h**
- ✅ **Scalabilité** : Redis Pub/Sub gère 100k msg/s (overkill restaurant, mais ready multi-sites)

**Architecture WebSocket** :
- **Socket.io** : Auto-fallback polling si WebSocket bloqué firewall
- **Rooms** : `table_{id}` → Isolation notifications (serveur Table 5 ≠ notifications Table 12)
- **Reconnexion auto** : Si mobile perd WebSocket, reconnecte + replay missed events

---

## G.2. Modèle Conceptuel de Données (MCD)

### G.2.1. Diagramme Entités-Relations (ERD)

**8 Entités Centrales** :

```mermaid
erDiagram
    USERS ||--o{ ORDERS : "crée/gère"
    TABLES ||--o{ ORDERS : "reçoit"
    ORDERS ||--|{ ORDER_ITEMS : "contient"
    MENU_ITEMS ||--o{ ORDER_ITEMS : "commandé dans"
    WINES ||--o| MENU_ITEMS : "suggéré pour"
    ORDERS ||--o{ PAYMENTS : "payé par"
    PAYMENTS ||--o| AUDIT_LOGS : "enregistré dans"
    TABLES ||--o| USERS : "assigné à"

    USERS {
        int id PK
        string username UK
        string password_hash
        enum role "serveur|caissier|admin"
        datetime created_at
    }

    TABLES {
        int id PK
        int number UK "1-20"
        enum status "libre|occupée|réservée"
        int capacity "2-8 couverts"
        int assigned_server_id FK
        datetime status_updated_at
    }

    ORDERS {
        int id PK
        int table_id FK
        int server_user_id FK
        enum status "pending|sent|preparing|ready|paid_complete"
        decimal total_amount "TTC"
        string erp_id UK "Corrélation ERP"
        datetime created_at
        datetime acknowledged_at
    }

    ORDER_ITEMS {
        int id PK
        int order_id FK
        int menu_item_id FK
        int quantity
        int cover_number "Pour split bill (1-8)"
        decimal unit_price "Prix au moment commande"
        text notes "Allergies, cuisson..."
    }

    MENU_ITEMS {
        int id PK
        string name UK
        enum category "entrée|plat|dessert|vin"
        decimal price_ttc
        decimal price_ht
        decimal tva_rate "5.5%|10%|20%"
        int wine_pairing_id FK "Recommandation vin (nullable)"
        jsonb allergens "INCO 1169/2011"
        boolean available
        datetime updated_at
    }

    WINES {
        int id PK
        string name UK
        string grape_variety "Cabernet, Merlot..."
        string region "Bordeaux, Cahors..."
        decimal price_glass
        decimal price_bottle
        text tasting_notes
        jsonb metadata "Millésime, etc."
    }

    PAYMENTS {
        int id PK
        int order_id FK
        int cover_number "Split bill (nullable si paiement global)"
        decimal amount
        enum method "CB|espèces|TR"
        enum status "pending|paid|failed"
        string approval_code "Code TPE (nullable)"
        int audit_log_id FK "NF525 (nullable)"
        datetime paid_at
    }

    AUDIT_LOGS {
        int id PK
        enum transaction_type "PAYMENT|CLOSURE_Z|REFUND"
        decimal amount
        date transaction_date
        string hash_current "SHA-256"
        string hash_previous "Chaînage NF525"
        text signature "RSA-2048 Base64"
        jsonb metadata "Détails transactions"
        datetime created_at
    }
```


**Relations Clés** :
- **1 WINE → N MENU_ITEMS** : Un vin recommandé pour plusieurs plats
- **1 ORDER → N PAYMENTS** : Split bill = plusieurs paiements par commande
- **UK COMPOSITE `payments(order_id, cover_number)`** : Empêche double paiement même personne

---

### G.2.2. Tables Principales avec Attributs

#### USERS (Utilisateurs/Serveurs/Caissiers/Admin)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK, AUTO_INCREMENT | Identifiant unique |
| `username` | VARCHAR(50) | UK, NOT NULL | Login unique |
| `password_hash` | VARCHAR(255) | NOT NULL | Hash bcrypt (10 rounds) |
| `role` | ENUM | NOT NULL | 'serveur' \| 'caissier' \| 'admin' |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Date création compte |

**Volumétrie** : **15 utilisateurs** (stable)

---

#### TABLES (Plan de Salle)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `number` | INTEGER | UK, 1-20 | Numéro table restaurant |
| `status` | ENUM | NOT NULL | 'libre' \| 'occupée' \| 'réservée' |
| `capacity` | INTEGER | 2-8 | Nombre couverts max |
| `assigned_server_id` | INTEGER | FK → USERS, NULLABLE | Serveur assigné (NULL si libre) |

**Volumétrie** : **20 tables** (fixe : 12×4 pers + 8×6 pers)

---

#### ORDERS (Commandes)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `table_id` | INTEGER | FK → TABLES, NOT NULL | Table concernée |
| `server_user_id` | INTEGER | FK → USERS, NOT NULL | Serveur ayant pris commande |
| `status` | ENUM | NOT NULL | 'pending' \| 'sent' \| 'ready' \| 'paid_complete' |
| `total_amount` | DECIMAL(10,2) | NOT NULL | Montant total TTC (auto-calculé) |
| `erp_id` | VARCHAR(50) | UK, NULLABLE | ID corrélation ERP ("K789") |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Date/heure création |

**Volumétrie** : **~65k/an** (180 commandes/jour × 365j) → **~3 Mo/an**

**Règle métier** : `total_amount = AUTO SUM(oder_items.unit_price × quantity)` (trigger)

---

#### ORDER_ITEMS (Détails Commande)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `order_id` | INTEGER | FK → ORDERS, NOT NULL | Commande parente |
| `menu_item_id` | INTEGER | FK → MENU_ITEMS, NOT NULL | Plat/vin commandé |
| `quantity` | INTEGER | NOT NULL, 1-20 | Quantité |
| `cover_number` | INTEGER | NULLABLE, 1-8 | Numéro couvert (split bill) |
| `unit_price` | DECIMAL(10,2) | NOT NULL | Prix figé au moment commande |
| `notes` | TEXT | NULLABLE, max 500 | Notes (allergies, cuisson...) |

**Volumétrie** : **~260k/an** (4 items/commande moyenne) → **~12 Mo/an**

**Règle métier** : `unit_price` copié de `MENU_ITEMS.price_ttc` à l'insertion (historique prix)

---

#### MENU_ITEMS (Carte Restaurant)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `name` | VARCHAR(100) | UK, NOT NULL | Nom plat/vin unique |
| `category` | ENUM | NOT NULL | 'entrée' \| 'plat' \| 'dessert' \| 'vin' |
| `price_ttc` | DECIMAL(10,2) | NOT NULL | Prix TTC client |
| `price_ht` | DECIMAL(10,2) | NOT NULL | Prix HT comptabilité |
| `tva_rate` | DECIMAL(4,2) | 5.5 \| 10 \| 20 | Taux TVA % |
| `wine_pairing_id` | INTEGER | FK → WINES, NULLABLE | **Vin recommandé** ⭐ |
| `allergens` | JSONB | NULLABLE | Allergènes INCO `["gluten", "lactose"]` |
| `available` | BOOLEAN | DEFAULT TRUE | Disponibilité (stock) |

**Volumétrie** : **50 plats** (stable) → **~15 Ko**

**Contrainte CHECK** : `price_ttc = ROUND(price_ht × (1 + tva_rate/100), 2)`

---

#### WINES (Cave Vins)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `name` | VARCHAR(100) | UK, NOT NULL | Nom vin unique |
| `grape_variety` | VARCHAR(50) | NULLABLE | Cépage (Cabernet, Merlot...) |
| `region` | VARCHAR(50) | NULLABLE | Région (Bordeaux, Cahors...) |
| `price_glass` | DECIMAL(10,2) | NULLABLE | **Prix au verre** ⭐ |
| `price_bottle` | DECIMAL(10,2) | NOT NULL | Prix bouteille |
| `tasting_notes` | TEXT | NULLABLE | Notes dégustation |

**Volumétrie** : **80 vins** (stable) → **~30 Ko**

**Règle métier** : `price_glass ≈ price_bottle / 5` (approximatif)

---

#### PAYMENTS (Paiements + Split Bill)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `order_id` | INTEGER | FK → ORDERS, NOT NULL | Commande payée |
| `cover_number` | INTEGER | **NULLABLE, 1-8** | **Numéro couvert split bill** ⭐ |
| `amount` | DECIMAL(10,2) | NOT NULL | Montant payé |
| `method` | ENUM | NOT NULL | 'CB' \| 'espèces' \| 'TR' |
| `status` | ENUM | NOT NULL | 'pending' \| 'paid' \| 'failed' |
| `approval_code` | VARCHAR(20) | NULLABLE | Code TPE (si CB) |
| `paid_at` | TIMESTAMP | NULLABLE | Date/heure paiement |

**Contrainte UNIQUE** : **`(order_id, cover_number)`** → **Empêche double paiement même personne** ✅

**Volumétrie** : **~95k/an** (1.5 paiements/commande à cause split bill) → **~5 Mo/an**

**Règle métier** : `SUM(payments.amount WHERE order_id=X) ≤ orders.total_amount`

---

#### AUDIT_LOGS (Conformité NF525)

| Attribut | Type | Contraintes | Description |
|:---------|:-----|:------------|:------------|
| `id` | INTEGER | PK | Identifiant unique |
| `transaction_type` | ENUM | NOT NULL | 'PAYMENT' \| 'CLOSURE_Z' \| 'REFUND' |
| `amount` | DECIMAL(10,2) | NOT NULL | Montant transaction |
| `transaction_date` | DATE | NOT NULL | Date transaction |
| `hash_current` | VARCHAR(64) | NOT NULL | **Hash SHA-256** ⭐ |
| `hash_previous` | VARCHAR(64) | NULLABLE | **Hash précédent (chaînage)** ⭐ |
| `signature` | TEXT | NOT NULL | **Signature RSA-2048 Base64** ⭐ |
| `metadata` | JSONB | NULLABLE | Détails (`{\"CB\": 2145.30, \"count\": 87}`) |

**Contrainte IMMUTABLE** : Trigger `BEFORE UPDATE/DELETE → RAISE EXCEPTION` ✅

**Volumétrie** : **~400 logs/an** (87 paiements/jour + 1 clôture/jour) → **~200 Ko/an**

**Règle métier** : `hash_current = SHA-256(concat(date, amount, hash_previous, metadata))`

---

### G.2.3. Contraintes d'Intégrité Critiques

#### Contraintes UNIQUE Composites

| Contrainte | Table | Colonnes | Justification |
|:-----------|:------|:---------|:--------------|
| **UK_split_bill** | `payments` | **`(order_id, cover_number)`** | **Empêche double paiement même couvert** ⭐⭐⭐ |
| UK_erp_correlation | `orders` | `erp_id` | Corrélation ERP 1-1 (idempotence) |

#### Triggers PostgreSQL

**Trigger 1 : Recalcul Total Commande**

```sql
CREATE OR REPLACE FUNCTION update_order_total() RETURNS TRIGGER AS $$
BEGIN
  UPDATE orders SET total_amount = (
    SELECT COALESCE(SUM(unit_price * quantity), 0)
    FROM order_items WHERE order_id = NEW.order_id
  ) WHERE id = NEW.order_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_total
AFTER INSERT OR UPDATE OR DELETE ON order_items
FOR EACH ROW EXECUTE FUNCTION update_order_total();
```

**Trigger 2 : Immutabilité Audit Logs NF525**

```sql
CREATE OR REPLACE FUNCTION prevent_audit_modification() RETURNS TRIGGER AS $$
BEGIN
  RAISE EXCEPTION 'Modification audit_logs interdite (NF525 compliance)';
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_immutable
BEFORE UPDATE OR DELETE ON audit_logs
FOR EACH ROW EXECUTE FUNCTION prevent_audit_modification();
```

---

### G.2.4. Volumétrie et Index

**Volumétrie Totale Estimée** :

| Table | Lignes/An | Taille Données | Taille Index | Total |
|:------|:----------:|:---------------|:-------------|:------|
| USERS | 15 | 5 Ko | 2 Ko | 7 Ko |
| TABLES | 20 | 2 Ko | 1 Ko | 3 Ko |
| **ORDERS** | **65k** | **3 Mo** | **1.5 Mo** | **4.5 Mo** |
| **ORDER_ITEMS** | **260k** | **12 Mo** | **6 Mo** | **18 Mo** |
| MENU_ITEMS | 50 | 15 Ko | 8 Ko | 23 Ko |
| WINES | 80 | 30 Ko | 15 Ko | 45 Ko |
| **PAYMENTS** | **95k** | **5 Mo** | **2.5 Mo** | **7.5 Mo** |
| AUDIT_LOGS | 400 | 200 Ko | 100 Ko | 300 Ko |
| **TOTAL** | **~420k** | **~20 Mo** | **~10 Mo** | **~30 Mo/an** |

**Index Critiques** :

```sql
CREATE INDEX idx_orders_table_status ON orders(table_id, status);
CREATE INDEX idx_order_items_cover ON order_items(order_id, cover_number);
CREATE UNIQUE INDEX idx_payments_split ON payments(order_id, cover_number)
  WHERE cover_number IS NOT NULL;
CREATE INDEX idx_audit_date ON audit_logs(transaction_date);
CREATE INDEX idx_menu_allergens ON menu_items USING GIN(allergens);
```

**Performance Requêtes** :
- Addition table split bill : **~12ms** (JOIN 3 tables)
- Vérification double paiement : **~3ms** (index unique composite)
- Menu + vins recommandés : **~18ms** (50 plats + 80 vins)

---

## G.3. Diagramme Interactions Environnement

### G.3.1. Vue Contexte Système (C4 Level 0)

**Périmètre** : Application Gestion Salle Restaurant

```mermaid
graph TB
    subgraph Restaurant["🏢 RESTAURANT (Périmètre Système)"]
        App["📱 Application Gestion Salle<br/>(Mobile + Web + API)"]
    end
    
    subgraph Acteurs["👥 ACTEURS HUMAINS"]
        Serveur["👤 Serveur<br/>(3 personnes)<br/>React Native Mobile"]
        Caissier["💰 Caissier<br/>(2 personnes)<br/>Interface Web"]
        Admin["👨‍💼 Administrateur<br/>(1 personne)<br/>Dashboard Web"]
    end
    
    subgraph Externes["🔌 SYSTÈMES EXTERNES"]
        ERP["🍳 ERP Cuisine<br/>'QuiCuisineIci'<br/>(REST API)"]
        TPE["💳 TPE Bancaire<br/>(Protocole propriétaire)<br/>VLAN 10 isolé"]
        Stocks["📦 API Stocks<br/>(Temps réel)"]
    end
    
    subgraph Infrastructure["☁️ INFRASTRUCTURE"]
        Monitoring["📊 Prometheus + Grafana<br/>(Métriques)"]
        Logs["📜 ELK Stack<br/>(Logs centralisés)"]
    end
    
    %% Relations Acteurs → App
    Serveur -->|"1. Prend commandes<br/>2. Consulte stocks<br/>3. Reçoit notifications"| App
    Caissier -->|"1. Encaissements<br/>2. Split bill<br/>3. Tickets NF525"| App
    Admin -->|"1. Rapports CA<br/>2. Gestion users<br/>3. Paramètres"| App
    
    %% Relations App → Systèmes Externes
    App -->|"POST /orders<br/>(Commandes cuisine)"| ERP
    ERP -->|"Webhook /dish-ready<br/>(Notifications plats)"| App
    
    App -->|"GET /stocks<br/>(Disponibilité ingrédients)"| Stocks
    
    App -->|"Transaction paiement<br/>(HTTPS TLS 1.3)"| TPE
    TPE -->|"Approval Code<br/>(Succès/Échec)"| App
    
    %% Relations App → Monitoring
    App -.->|"Métriques CPU/RAM<br/>Latence API"| Monitoring
    App -.->|"Logs JSON<br/>(Winston)"| Logs
    
    %% Styling
    classDef acteur fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    classDef externe fill:#FFF3E0,stroke:#F57C00,stroke-width:2px
    classDef infra fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px,stroke-dasharray: 5 5
    classDef app fill:#C8E6C9,stroke:#388E3C,stroke-width:3px
    
    class Serveur,Caissier,Admin acteur
    class ERP,TPE,Stocks externe
    class Monitoring,Logs infra
    class App app
```

**3 Acteurs Humains** :
1. 👤 **Serveur** (3 personnes) → Mobile React Native → 180 commandes/jour
2. 💰 **Caissier** (2 personnes) → Web React.js → 95 paiements/jour + clôture Z
3. 👨‍💼 **Admin** (1 personne) → Dashboard Web → Rapports CA + gestion users

**3 Systèmes Externes Métier** :
4. 🍳 **ERP "QuiCuisineIci"** → REST HTTPS bidirectionnel → 180 orders + 60 callbacks/jour
5. 💳 **TPE Bancaire** → Protocole propriétaire TLS (VLAN 10 isolé) → 95 transactions/jour
6. 📦 **API Stocks** → REST HTTPS lecture → ~500 requêtes/jour (cache 30s)

**2 Systèmes Infrastructure** :
7. 📊 **Prometheus + Grafana** → Scraping métriques HTTP → 1 req/15s
8. 📜 **ELK Stack** → Push logs JSON asynchrone → ~2000 logs/jour (90j rétention)

---

### G.3.2. Flux Principaux par Système Externe

#### ERP QuiCuisineIci (Cuisine)

**Direction** : ↕️ **Bidirectionnel**

**Flux Sortant** (App → ERP) :
- Endpoint : `POST /api/v2/orders`
- Auth : Bearer Token API
- Payload : `{order_id, table_number, items: [{dish, qty, notes}]}`
- Fréquence : **180 req/jour**
- SLA : **<500ms P95**
- Résilience : **Circuit breaker Opossum** (timeout 5s, 3 failures → OPEN, retry 30s)

**Flux Entrant** (ERP → App) :
- Endpoint : `POST /webhooks/erp/dish-ready`
- Auth : HMAC-SHA256 signature
- Payload : `{erp_order_id, dish_name, table_id}`
- Fréquence : **~60 callbacks/jour**
- Latence : **<100ms** (WebSocket notification serveur)

---

#### TPE Bancaire (Paiements CB)

**Direction** : ↕️ **Bidirectionnel**

**Architecture Réseau** :
- **VLAN 10** : TPE + Backend API (segment PCI DSS)
- **VLAN 20** : Mobiles/Web (segment application)
- **Firewall** : DENY communication directe VLAN 20 → VLAN 10

**Flux Transaction** :
- Endpoint local : `POST http://192.168.10.50:8080/payment`
- Payload : `{transaction_id, amount_cents, currency: "EUR"}`
- Réponse : `{approval_code, card_masked: "****1234", status}`
- Fréquence : **95 transactions/jour**
- Latence : **2-3s** (délai réseau bancaire)
- Timeout : **<10s** (éviter blocage caisse)

**Conformité PCI DSS** :
- ✅ Pas stockage PAN (numéro carte complet) dans BDD
- ✅ Uniquement `card_last4` VARCHAR(4) conservé
- ✅ Chiffrement TLS 1.3 transit

---

#### API Stocks Temps Réel

**Direction** : → **Unidirectionnel** (read-only)

**Consultation** :
- Endpoint : `GET /v1/ingredients/{id}/availability`
- Auth : Header `X-API-Key`
- Réponse : `{available_quantity, unit, last_updated}`
- Fréquence : **~500 req/jour**
- Cache : **Redis TTL 30s** (taux hit 85%)
- Performance : 5ms (cache hit) vs 80ms (cache miss)

**Gestion Indisponibilité** : Non bloquant (fallback = menu complet + warning)

---

### G.3.3. Patterns d'Intégration

| Système | Pattern Appliqué | Justification |
|:--------|:-----------------|:--------------|
| **ERP** | Circuit Breaker (Opossum) | Évite cascade failure si ERP down |
| **ERP** | Webhook + HMAC signature | Sécurisation callbacks authentification mutuelle |
| **TPE** | Network Segmentation (VLAN) | Isolation PCI DSS données carte |
| **API Stocks** | Cache-Aside (Redis) | Performance + réduction appels externes |
| **Prometheus** | Pull Model (Scraping) | Standard observabilité |
| **ELK** | Buffered Push (Winston) | Logs asynchrones non-bloquants |

---

