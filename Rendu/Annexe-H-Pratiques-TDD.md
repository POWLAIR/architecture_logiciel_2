# Annexe H : Pratiques Test Driven Development (TDD)

*Ce document est un complément détaillé du Cahier des Charges Principal*

---

## Introduction

Cette annexe présente la **démarche TDD** déployée pour garantir la qualité du système. Elle démontre comment cette approche s'aligne parfaitement avec l'architecture modulaire 3-tiers retenue et renforce la maintenabilité, la conformité, et la confiance dans le code produit.

---

## H.1. Rappel Architecture Modulaire

### H.1.1. Architecture 3-Tiers Testable

**Organisation adoptée** :

```
┌─────────────────────────────────────────────┐
│  COUCHE PRÉSENTATION                        │
│  - Mobile React Native (serveurs)           │
│  - Web React.js (caissiers/admin)           │
│  - Composants UI modulaires réutilisables   │
└─────────────────────────────────────────────┘
                    ↓ API REST/WebSocket
┌─────────────────────────────────────────────┐
│  COUCHE MÉTIER (Backend Node.js)            │
│  - Controllers (routes API)                 │
│  - Services (logique métier)                │
│  - Repositories (accès données)             │
│  - Middlewares (auth, validation, logs)     │
└─────────────────────────────────────────────┘
                    ↓ SQL/Cache
┌─────────────────────────────────────────────┐
│  COUCHE DONNÉES                             │
│  - PostgreSQL (SGBD relationnel)            │
│  - Redis (cache + Pub/Sub)                  │
│  - SQLite (offline mobile)                  │
└─────────────────────────────────────────────┘
```

### H.1.2. Modularité par Domaines Métier

**Organisation Backend** :

```
backend/src/modules/
├── orders/           # Gestion commandes
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   ├── orders.repository.ts
│   └── __tests__/
├── payments/         # Gestion paiements + split bill
│   ├── payments.controller.ts
│   ├── split-bill.service.ts
│   ├── payments.repository.ts
│   └── __tests__/
├── nf525/           # Conformité fiscale
│   ├── nf525.service.ts
│   ├── nf525.crypto.ts
│   └── __tests__/
├── menu/            # Carte + vins
├── tables/          # Plan de salle
├── users/           # Authentification
└── websocket/       # Notifications temps réel
```

**Avantages pour TDD** :
- ✅ **Isolation** : Modules testables indépendamment
- ✅ **Responsabilité unique** : Scope tests bien défini
- ✅ **Mocking facilité** : Dépendances clairement identifiées
- ✅ **Parallélisation** : Tests modules exécutés concurremment (CI/CD rapide)

---

## H.2. Cycle TDD Red-Green-Refactor

### H.2.1. Méthodologie Classique

**Cycle itératif appliqué** :

```
🔴 RED : Écrire test qui échoue
   ↓
🟢 GREEN : Code minimal pour passer test
   ↓
🔵 REFACTOR : Améliorer sans casser tests
   ↓ (retour)
🔴 RED : Test suivant...
```

**Application Concrète** :

**1. 🔴 Phase RED** : Écrire test fonctionnel AVANT implémentation
- Exemple : `test('should recommend wine for main dish')`
- Test échoue car fonction n'existe pas encore
- Focus sur **comportement attendu** (spec métier)

**2. 🟢 Phase GREEN** : Implémenter code minimal pour passer
- Écrire juste assez de code (pas de sur-engineering)
- Test passe → Baseline validée
- Performance non optimisée (acceptable)

**3. 🔵 Phase REFACTOR** : Améliorer qualité code
- Éliminer duplication (principe DRY)
- Respecter SOLID principles
- Tests toujours verts → Confiance refactoring

### H.2.2. Adaptation à l'Architecture Modulaire

#### TDD par Couche

| Couche | Approche TDD | Outils | Isolation |
|:-------|:-------------|:-------|:----------|
| **Présentation** | Component Testing | Jest + React Testing Library | Mock API calls |
| **Métier** | Unit + Integration Tests | Jest + Supertest | Mock repositories |
| **Données** | Integration Tests DB | Jest + TestContainers | Transactions rollback |

#### Exemple Module `orders`

```typescript
// 1. Test service métier (logique pure)
describe('OrdersService', () => {
  it('should calculate total amount with wine pairing', () => {
    const mockRepo = {findById: jest.fn()};
    const service = new OrdersService(mockRepo);
    
    const total = service.calculateTotal({
      items: [{price: 25.50, qty: 2}, {price: 8.00, qty: 1}]
    });
    
    expect(total).toBe(59.00); // 2×25.50 + 8.00
  });
});

// 2. Test repository (accès données)
describe('OrdersRepository', () => {
  it('should insert order with ACID transaction', async () => {
    // TestContainer PostgreSQL réel
    await repo.create({table_id: 5, items: [...]});
    
    const order = await repo.findById(1);
    expect(order.status).toBe('pending');
  });
});

// 3. Test controller (API REST)
describe('POST /api/orders', () => {
  it('should return 201 with valid JWT', async () => {
    const response = await supertest(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${validToken}`)
      .send({table_id: 5, items: [...]});
    
    expect(response.status).toBe(201);
  });
});
```

**Bénéfices** :
- ✅ Tests `OrdersService` isolés (pas de DB, pas HTTP)
- ✅ Tests `OrdersRepository` vérifient SQL réel (contraintes UK, ACID)
- ✅ Tests `OrdersController` valident stack complète (JWT + RBAC + Joi)

---

## H.3. Pyramide de Tests

### H.3.1. Répartition Stratégique

**Distribution 70/25/5** :

```
        ╱╲
       ╱  ╲     E2E (5%) 🎭
      ╱    ╲    150 tests scénarios complets
     ╱──────╲   
    ╱        ╲  Intégration (25%) 🔗
   ╱          ╲ 700 tests modules + DB + APIs
  ╱────────────╲
 ╱              ╲ Unitaires (70%) ⚙️
╱────────────────╲ 2000 tests fonctions pures
```

**Total : 2850 tests** garantissant **92% confiance globale**

### H.3.2. Tests Unitaires (70% - Base Pyramide)

**Objectif** : Valider logique métier **isolée** et **rapide**

**Scope Architectural** :
- `*.service.ts` : Logique métier pure (calculs, validations, transformations)
- `*.utils.ts` : Fonctions utilitaires (formatage, parsing)
- `*.crypto.ts` : Algorithmes cryptographiques (hashing NF525, signatures RSA)

**Caractéristiques** :
- ✅ **Aucune dépendance externe** (pas DB, pas HTTP, pas filesystem)
- ✅ **Exécution ultra-rapide** (<1ms/test)
- ✅ **Mocking total** : Services dépendants remplacés par mocks Jest
- ✅ **Parallélisation maximale** : Tests complètement indépendants

**Exemple Module Split Bill** :

```typescript
describe('SplitBillService', () => {
  describe('calculateEqualSplit', () => {
    it('should split 102€ among 4 covers = 25.50€ each', () => {
      const service = new SplitBillService();
      const amounts = service.calculateEqualSplit(102.00, 4);
      
      expect(amounts).toEqual([25.50, 25.50, 25.50, 25.50]);
    });
    
    it('should handle decimal rounding (100€ / 3)', () => {
      const amounts = service.calculateEqualSplit(100.00, 3);
      
      // 33.33 + 33.33 + 33.34 = 100.00 (dernier ajusté)
      expect(amounts).toEqual([33.33, 33.33, 33.34]);
      expect(amounts.reduce((a,b) => a+b, 0)).toBe(100.00);
    });
    
    it('should throw error for covers = 0', () => {
      expect(() => {
        service.calculateEqualSplit(100, 0);
      }).toThrow('Covers must be > 0');
    });
    
    it('should throw error for negative amount', () => {
      expect(() => {
        service.calculateEqualSplit(-50, 4);
      }).toThrow('Amount must be >= 0');
    });
  });
});
```

**Métriques Cibles** :
- **Quantité** : ~2000 tests unitaires (70% total)
- **Couverture** : ≥90% lignes code logique métier
- **Performance** : Suite complète <2s

---

### H.3.3. Tests d'Intégration (25% - Milieu Pyramide)

**Objectif** : Valider **interactions réelles** entre modules et systèmes

**Scope Architectural** :
- `*.repository.ts` : Accès données PostgreSQL (CRUD, transactions ACID, contraintes)
- `*.controller.ts` : Endpoints API REST (middlewares, validation, réponses)
- Intégrations externes : ERP, TPE (via mocks MSW réseau)

**Caractéristiques** :
- ✅ **Base données réelle** : PostgreSQL via TestContainers (pas mock ORM)
- ✅ **Transactions rollback** : Isolation complète entre tests
- ✅ **Contraintes DB testées** : UNIQUE, FK, triggers, CHECK
- ✅ **Latence acceptable** : 50-200ms/test (I/O disque réel)

**Infrastructure TestContainers** :

```typescript
// Setup global suite intégration
beforeAll(async () => {
  // Lance container PostgreSQL éphémère
  container = await new PostgreSqlContainer('postgres:15')
    .withDatabase('test_restaurant')
    .withUsername('test')
    .withPassword('test')
    .start();
  
  // Applique migrations schema
  await runMigrations(container.getConnectionString());
});

afterAll(async () => {
  await container.stop(); // Cleanup auto
});

beforeEach(async () => {
  await db.query('BEGIN'); // Transaction isolée
});

afterEach(async () => {
  await db.query('ROLLBACK'); // Annule modifications
});
```

**Exemple Module Payments Repository** :

```typescript
describe('PaymentsRepository Integration', () => {
  it('should reject duplicate payment (order_id, cover_number) UK', async () => {
    // Premier paiement : OK
    await repo.create({
      order_id: 1,
      cover_number: 1,
      amount: 25.50,
      method: 'CB'
    });
    
    // Tentative doublon : ❌ Exception PostgreSQL
    await expect(
      repo.create({
        order_id: 1,
        cover_number: 1, // MÊME couvert
        amount: 30.00,
        method: 'CB'
      })
    ).rejects.toThrow('duplicate key value violates unique constraint');
  });
  
  it('should use row-level lock FOR UPDATE (concurrence)', async () => {
    // Simulation 2 caissiers simultanés
    const promise1 = repo.findByIdForUpdate(1); // Acquiert lock
    const promise2 = repo.findByIdForUpdate(1); // Attend lock
    
    const [result1] = await Promise.all([promise1, promise2]);
    
    expect(result1.id).toBe(1);
    // promise2 attend que promise1 COMMIT/ROLLBACK
  });
  
  it('should rollback transaction if TPE fails', async () => {
    await db.query('BEGIN');
    
    await repo.create({order_id: 2, amount: 50.00});
    
    // Simulation échec TPE
    throw new Error('TPE timeout');
    
    await db.query('ROLLBACK');
    
    // Vérification : paiement pas persisté
    const count = await repo.countByOrderId(2);
    expect(count).toBe(0);
  });
});
```

**Métriques Cibles** :
- **Quantité** : ~700 tests intégration (25% total)
- **Couverture** : ≥80% chemins critiques DB
- **Performance** : Suite complète <140s

---

### H.3.4. Tests End-to-End (5% - Sommet Pyramide)

**Objectif** : Valider **parcours utilisateur complets** de bout en bout

**Scope Architectural** :
- Parcours critiques IT1-IT4 (scénarios UC majeurs)
- Stack complète : React UI → Express API → PostgreSQL
- Simulation utilisateurs réels

**Caractéristiques** :
- ✅ **Pas de mocks** : Système réel complet (sauf ERP/TPE mockés MSW)
- ✅ **Scénarios métier** : Alignés cas d'usage Annexe A
- ✅ **Validation bout en bout** : UX + API + persistance + conformité
- ✅ **Latence réaliste** : <5s/scénario (acceptable production)

**Exemple UC4.3 Split Bill Complet** :

```typescript
describe('E2E: Split Bill 4 Personnes', () => {
  it('should allow 4 individual CB payments for same order', async () => {
    // 1. Authentification caissier
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({username: 'caissier1', password: 'test123'});
    
    const token = loginRes.body.token;
    
    // 2. Création commande table 12 (4 couverts)
    const orderRes = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${token}`)
      .send({
        table_id: 12,
        items: [
          {menu_item_id: 5, quantity: 4, cover_number: null} // Plat partagé
        ]
      });
    
    const orderId = orderRes.body.order_id;
    expect(orderRes.status).toBe(201);
    
    // 3. Paiement individuel personne 1
    const payment1 = await request(app)
      .post('/api/payments/split')
      .set('Authorization', `Bearer ${token}`)
      .send({
        order_id: orderId,
        cover_number: 1,
        amount: 25.50,
        method: 'CB'
      });
    
    expect(payment1.status).toBe(201);
    expect(payment1.body.remaining).toBe('3/4'); // 3 personnes restantes
    
    // 4. Paiements 2, 3, 4 (boucle)
    for (let cover = 2; cover <= 4; cover++) {
      const paymentN = await request(app)
        .post('/api/payments/split')
        .set('Authorization', `Bearer ${token}`)
        .send({
          order_id: orderId,
          cover_number: cover,
          amount: 25.50,
          method: 'CB'
        });
      
      expect(paymentN.status).toBe(201);
    }
    
    // 5. Vérification addition complète
    const orderCheck = await request(app)
      .get(`/api/orders/${orderId}`)
      .set('Authorization', `Bearer ${token}`);
    
    expect(orderCheck.body.status).toBe('paid_complete');
    
    // 6. Tentative re-paiement personne 1 → ❌ 409 Conflict
    const duplicatePayment = await request(app)
      .post('/api/payments/split')
      .set('Authorization', `Bearer ${token}`)
      .send({
        order_id: orderId,
        cover_number: 1, // Déjà payé
        amount: 25.50,
        method: 'CB'
      });
    
    expect(duplicatePayment.status).toBe(409);
    expect(duplicatePayment.body.error).toContain('already paid');
  });
});
```

**Scénarios E2E Critiques** :

| Scénario UC | Description | Tests E2E | Confiance |
|:------------|:------------|:---------:|:---------:|
| **UC4.3 Split Bill** | 4 paiements individuels + protection doublon | 2 | 100% |
| **UC5.2 Clôture NF525** | Hash chaîné + signature RSA + ticket Z | 1 | 100% |
| **UC7.1 Offline Sync** | 3 commandes offline → reconnexion → sync auto | 1 | 95% |
| **UC2.1 Vin recommandé** | Commande plat → suggestion vin automatique | 1 | 90% |
| **UC8.2 WebSocket notif** | ERP plat prêt → push serveur mobile | 1 | 90% |

**Métriques Cibles** :
- **Quantité** : ~150 tests E2E (5% total)
- **Couverture** : 100% UC critiques IT1-IT4
- **Performance** : Suite complète <12min

---

## H.4. Stratégie TDD par Scénario Critique

### H.4.1. Scénario NF525 - Clôture Journalière (IT2)

**Exigences Critiques** :
- ✅ Hash chaîné SHA-256 (chaque clôture référence précédente)
- ✅ Signature RSA-2048 (certificat NF525 légal)
- ✅ Trigger immuable PostgreSQL (interdiction UPDATE/DELETE audit_logs)
- ✅ Ticket Z imprimable PDF (obligation conservation 6 ans)

**Décomposition TDD** :

**Tests Unitaires (50 tests)** :
```typescript
describe('NF525CryptoService', () => {
  it('should generate deterministic SHA-256 hash', () => {
    const data = "2026-02-02|87|2656.80|a3f2d1...";
    
    const hash1 = service.generateHash(data);
    const hash2 = service.generateHash(data);
    
    expect(hash1).toBe(hash2); // Déterministe
    expect(hash1).toMatch(/^[a-f0-9]{64}$/); // 64 hex chars
  });
  
  it('should produce different hash if input changes', () => {
    const hash1 = service.generateHash("data1");
    const hash2 = service.generateHash("data2");
    
    expect(hash1).not.toBe(hash2);
  });
  
  it('should sign hash with RSA-2048 private key', () => {
    const hash = "d8b4c2a1...";
    const privateKey = loadKey('/secure/nf525_priv.pem');
    
    const signature = service.signRSA(hash, privateKey);
    
    expect(signature).toMatch(/^[A-Za-z0-9+/=]+$/); // Base64
  });
  
  it('should verify signature with public key', () => {
    const hash = "d8b4c2a1...";
    const signature = "MIICXAIBAAKBgQC...";
    const publicKey = loadKey('/secure/nf525_pub.pem');
    
    const isValid = service.verifyRSA(hash, signature, publicKey);
    
    expect(isValid).toBe(true);
  });
});
```

**Tests Intégration (15 tests)** :
```typescript
describe('AuditLogsRepository Integration', () => {
  it('should prevent UPDATE via trigger', async () => {
    const log = await repo.create({
      transaction_type: 'PAYMENT',
      amount: 25.50,
      hash_current: 'd8b4...',
      hash_previous: 'a3f2...',
      signature: 'MIIC...'
    });
    
    // Tentative modification ❌
    await expect(
      db.query('UPDATE audit_logs SET amount = 30.00 WHERE id = $1', [log.id])
    ).rejects.toThrow('Modification audit_logs interdite (NF525 compliance)');
  });
  
  it('should prevent DELETE via trigger', async () => {
    const log = await repo.create({...});
    
    await expect(
      db.query('DELETE FROM audit_logs WHERE id = $1', [log.id])
    ).rejects.toThrow('Modification audit_logs interdite');
  });
  
  it('should chain hash_previous correctly', async () => {
    const log1 = await repo.create({hash_current: 'aaa...', hash_previous: null});
    const log2 = await repo.create({hash_current: 'bbb...', hash_previous: 'aaa...'});
    
    const retrieved = await repo.findById(log2.id);
    expect(retrieved.hash_previous).toBe(log1.hash_current);
  });
});
```

**Tests E2E (3 tests)** :
```typescript
it('should execute complete daily closure Z', async () => {
  // 1. Créer 87 paiements journée
  for (let i = 0; i < 87; i++) {
    await createPayment({...});
  }
  
  // 2. Déclencher clôture
  const closureRes = await request(app)
    .post('/api/nf525/close-day')
    .set('Authorization', adminToken);
  
  expect(closureRes.status).toBe(200);
  
  // 3. Vérifier hash chaîné
  const logs = await db.query(
    'SELECT hash_current, hash_previous FROM audit_logs ORDER BY id'
  );
  
  for (let i = 1; i < logs.rows.length; i++) {
    expect(logs.rows[i].hash_previous).toBe(logs.rows[i-1].hash_current);
  }
  
  // 4. Vérifier signature RSA valide
  const lastLog = logs.rows[logs.rows.length - 1];
  const isValid = verifyRSA(lastLog.hash_current, lastLog.signature, publicKey);
  
  expect(isValid).toBe(true);
  
  // 5. Vérifier ticket Z PDF généré
  expect(closureRes.body.ticket_pdf_url).toBeDefined();
});
```

**Total NF525** : **68 tests** → **100% confiance conformité légale**

---

### H.4.2. Scénario Offline Sync (IT3)

**Exigences Critiques** :
- ✅ SQLite local (source vérité offline)
- ✅ Exponential backoff (retry 30s → 60s → 120s → 240s...)
- ✅ Détection conflits (table déjà occupée ailleurs)
- ✅ UUID corrélation sync (idempotence)

**Décomposition TDD** :

**Tests Unitaires (35 tests)** :
```typescript
describe('SyncQueueService', () => {
  it('should calculate exponential backoff correctly', () => {
    const delays = service.calculateBackoff(retryCount);
    
    expect(delays[0]).toBe(30000);   // 30s
    expect(delays[1]).toBe(60000);   // 60s
    expect(delays[2]).toBe(120000);  // 120s
    expect(delays[3]).toBe(240000);  // 240s
    expect(delays[4]).toBe(240000);  // Cap max
  });
  
  it('should mark conflict status on 409 response', () => {
    const order = {uuid: 'offline-123', status: 'pending_sync'};
    
    service.handleSyncResponse(order, {status: 409});
    
    expect(order.status).toBe('conflict');
    expect(order.conflict_reason).toBeDefined();
  });
  
  it('should prioritize queue by timestamp FIFO', () => {
    const orders = [
      {uuid: 'a', created_at: '2026-02-02T12:30:00Z'},
      {uuid: 'b', created_at: '2026-02-02T12:25:00Z'},
      {uuid: 'c', created_at: '2026-02-02T12:35:00Z'}
    ];
    
    const sorted = service.sortQueue(orders);
    
    expect(sorted.map(o => o.uuid)).toEqual(['b', 'a', 'c']);
  });
});
```

**Tests Intégration (20 tests)** :
```typescript
describe('OfflineStorage (React Native SQLite)', () => {
  it('should insert order locally in offline mode', async () => {
    const order = {
      uuid: 'offline-abc-123',
      table_id: 8,
      items: [{dish_id: 12, qty: 1}],
      status: 'pending_sync'
    };
    
    await offlineDB.insertOrder(order);
    
    const retrieved = await offlineDB.getOrderByUuid('offline-abc-123');
    expect(retrieved.table_id).toBe(8);
  });
  
  it('should update status after successful sync', async () => {
    await offlineDB.insertOrder({uuid: 'offline-xyz', status: 'pending_sync'});
    
    await offlineDB.updateSyncStatus('offline-xyz', 'synced', Date.now());
    
    const order = await offlineDB.getOrderByUuid('offline-xyz');
    expect(order.status).toBe('synced');
    expect(order.synced_at).toBeDefined();
  });
});
```

**Tests E2E (5 tests)** :
```typescript
it('should sync 3 offline orders after reconnection', async () => {
  // 1. Mode offline : créer 3 commandes locales
  await mobileApp.createOrderOffline({table_id: 3, items: [...]});
  await mobileApp.createOrderOffline({table_id: 5, items: [...]});
  await mobileApp.createOrderOffline({table_id: 7, items: [...]});
  
  // 2. Simuler reconnexion réseau
  await mobileApp.simulateReconnection();
  
  // 3. Attendre synchronisation auto
  await waitFor(() => {
    expect(mobileApp.getSyncStatus()).toBe('completed');
  }, {timeout: 5000});
  
  // 4. Vérifier backend a reçu 3 commandes
  const backendOrders = await db.query(
    'SELECT * FROM orders WHERE offline_uuid LIKE $1',
    ['offline-%']
  );
  
  expect(backendOrders.rows.length).toBe(3);
  
  // 5. Vérifier latence sync < 5s (objectif IT3)
  const syncDuration = mobileApp.getSyncDuration();
  expect(syncDuration).toBeLessThan(5000);
});
```

**Total Offline Sync** : **60 tests** → **95% confiance résilience**

---

## H.5. Organisation et Outillage TDD

### H.5.1. Stack Technique Tests

| Outil | Usage | Justification |
|:------|:------|:--------------|
| **Jest** | Test runner principal | Standard JavaScript, mocking intégré, snapshot testing, parallélisation native |
| **Supertest** | Tests API HTTP | Simulation requêtes Express sans serveur HTTP réel, idéal contrôleurs |
| **React Testing Library** | Tests composants React | Approche centrée utilisateur (teste comportement visible, pas implémentation interne) |
| **TestContainers** | DB éphémères Docker | PostgreSQL/Redis **réels** (pas mock ORM), isolation complète, compatible CI/CD |
| **MSW (Mock Service Worker)** | Mock APIs externes | Intercepte requêtes ERP/TPE au niveau réseau (transparent pour code) |
| **Faker.js** | Génération données test | Données réalistes reproductibles (noms, montants, dates) |

### H.5.2. Configuration Jest

**Fichier `jest.config.js`** :

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  
  // Seuils couverture (gate qualité CI/CD)
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 85,
      lines: 90,
      statements: 90
    },
    // Modules critiques : seuil plus élevé
    './src/modules/nf525/**/*.ts': {
      branches: 90,
      functions: 95,
      lines: 95,
      statements: 95
    },
    './src/modules/payments/**/*.ts': {
      branches: 90,
      functions: 95,
      lines: 95,
      statements: 95
    }
  },
  
  // Parallélisation (50% CPUs disponibles)
  maxWorkers: '50%',
  
  // Patterns fichiers tests
  testMatch: [
    '**/__tests__/**/*.spec.ts',
    '**/__tests__/**/*.integration.spec.ts',
    '**/__tests__/**/*.e2e.spec.ts'
  ],
  
  // Coverage reporters
  coverageReporters: ['text', 'lcov', 'html']
};
```

**Scripts NPM** :

```json
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest --testPathPattern=spec.ts$",
    "test:integration": "jest --testPathPattern=integration.spec.ts$",
    "test:e2e": "jest --testPathPattern=e2e.spec.ts$",
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch",
    "test:ci": "jest --ci --coverage --maxWorkers=2"
  }
}
```

---

## H.6. Métriques et Objectifs Qualité

### H.6.1. Couverture par Module

| Module | Couverture Cible | Criticité | Justification |
|:-------|:----------------:|:---------:|:--------------|
| `nf525` | **≥95%** | 🔴 CRITIQUE | Obligation légale fiscale, audit DGFiP |
| `payments` | **≥95%** | 🔴 CRITIQUE | Transactions financières ACID, split bill |
| `orders` | **≥90%** | 🟠 HAUTE | Cœur métier commandes restaurant |
| `sync` | **≥90%** | 🟠 HAUTE | Résilience offline IT3 critique serveurs |
| `auth` | **≥85%** | 🟡 MOYENNE | Sécurité JWT + RBAC |
| `menu` | **≥80%** | 🟢 BASSE | Logique simple CRUD |
| `websocket` | **≥80%** | 🟢 BASSE | Notifications temps réel (non bloquant) |

**Global** : **≥90% lignes** + **≥85% branches** + **≥90% fonctions**

### H.6.2. Performance Tests

**Objectifs Vitesse Exécution** :

```
Tests Unitaires      : <1ms/test    (2000 tests en <2s)
Tests Intégration    : <200ms/test  (700 tests en <140s)
Tests E2E            : <5s/scénario (150 tests en <12min)

Total Suite Complète : <15min (bloquant CI/CD si >20min)
```

**Justification Temps** :
- Feedback rapide développeurs (suite unitaire <2s = iteration loop rapide)
- CI/CD viable (pipeline ~15min acceptable vs >30min frustrant)
- ROI productivité (évite attentes longues développeurs)

### H.6.3. Répartition Tests par Scénario IT

| Scénario IT | Tests Unit | Tests Integ | Tests E2E | Total | Confiance |
|:------------|:----------:|:-----------:|:---------:|:-----:|:---------:|
| **IT1 - Commandes/Vins** | 180 | 85 | 12 | **277** | 95% |
| **IT2 - NF525 Clôture** | 50 | 15 | 3 | **68** | 100% |
| **IT2 - Paiements Split** | 45 | 28 | 8 | **81** | 100% |
| **IT3 - Offline Sync** | 35 | 20 | 5 | **60** | 95% |
| **IT3 - Performance Cache** | 40 | 18 | 3 | **61** | 90% |
| **IT4 - Notifications WS** | 25 | 10 | 3 | **38** | 90% |
| **IT4 - Observabilité** | 20 | 8 | 2 | **30** | 85% |
| Autres (auth, menu, tables...) | 1605 | 516 | 114 | **2235** | 85-90% |
| **TOTAL** | **2000** | **700** | **150** | **2850** | **92%** |

**Score Confiance Global** : **92%** (aligné score ISO 25010 architecture 92.5/100)

---

## H.7. Intégration CI/CD

### H.7.1. Pipeline GitLab CI (Conceptuel)

**Étapes Pipeline** :

```yaml
stages:
  - lint              # ESLint + Prettier
  - test:unit         # Tests unitaires (parallèles)
  - test:integration  # Tests intégration (DB TestContainers)
  - test:e2e          # Tests E2E (optionnel MR, obligatoire main)
  - quality-gate      # Gates qualité (bloque merge)
  - build             # Build Docker images
  - deploy            # Déploiement staging/prod
```

**Gates Qualité (Bloquants)** :
- ✅ Tests unitaires = **0 échecs** (bloquant)
- ✅ Couverture globale ≥ **90%** (bloquant)
- ✅ Couverture critiques ≥ **95%** (`nf525`, `payments`) (bloquant)
- ⚠️ Tests E2E = 0 échecs (warning MR, bloquant `main`)
- ✅ Pas de vulnérabilités CRITICAL/HIGH (npm audit)

### H.7.2. Pre-commit Hooks (Husky)

**Validation Pré-commit** :

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged && npm run test:unit -- --findRelatedTests"
    }
  },
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"]
  }
}
```

**Bénéfice** : Détection erreurs avant push (économie temps CI/CD)

---

## H.8. Synergies Architecture × TDD

### H.8.1. Alignement Naturel

**Pourquoi TDD est adapté à l'architecture 3-tiers modulaire :**

#### Isolation Naturelle Modules

✅ **Architecture** : Modules domaine indépendants (`orders/`, `payments/`, `nf525/`)

✅ **TDD** : Tests isolés par module sans couplage inter-modules

**Bénéfice** : Parallélisation CI/CD maximale (tests `orders` ≠ tests `payments`)

#### Responsabilité Unique (SOLID)

✅ **Architecture** : Séparation Controller → Service → Repository

✅ **TDD** : 1 scope test = 1 responsabilité
- `service.spec.ts` teste logique métier pure
- `repository.integration.spec.ts` teste persistence
- `controller.integration.spec.ts` teste API REST + middlewares

**Bénéfice** : Pas de tests "fourre-tout" (maintenance facile, debugging rapide)

#### Injection Dépendances

✅ **Architecture** : Services injectés (IoC pattern)

✅ **TDD** : Mocking facilité (swap implémentation réelle → mock)

**Exemple** :
```
OrdersService (production)
  ↓ dépend de
PaymentsRepository (mock en test unitaire)
```

**Bénéfice** : Tests rapides (pas appel DB réel tests service)

#### Refactoring Safe

✅ **Architecture** : Interfaces stables entre couches

✅ **TDD** : Tests verts = contrat respecté

**Bénéfice** : Refactoring interne service sans casser tests (tant que comportement ≈ spec)

### H.8.2. Garanties Fonctionnelles TDD

| Exigence Critique | Tests TDD Associés | Niveau Confiance |
|:------------------|:-------------------|:----------------:|
| **Split Bill ACID** | 25 tests intégration PostgreSQL (row-locking, UK composite) | ✅ 100% |
| **NF525 Immutabilité** | 15 tests trigger + 50 tests crypto SHA-256 + RSA | ✅ 100% |
| **Offline Sync** | 20 tests SQLite + 35 tests queue exponential backoff | ✅ 95% |
| **Cache Performance Redis** | 30 tests TTL + invalidation + hit ratio | ✅ 90% |
| **Circuit Breaker ERP** | 10 tests Opossum states (CLOSED/OPEN/HALF-OPEN) | ✅ 100% |
| **JWT Sécurité** | 18 tests expiration + RBAC + injection SQL | ✅ 95% |

### H.8.3. Garanties Non-Fonctionnelles

**Performance** :
- ✅ Tests latence API (<200ms P95) intégrés suite E2E
- ✅ Tests charge (Artillery 100 req/s) séparés TDD classique

**Sécurité** :
- ✅ Tests JWT expiration automatique
- ✅ Tests RBAC unauthorized (role serveur ≠ endpoint admin)
- ✅ Tests injection SQL (queries paramétrées Prisma)

**Conformité NF525** :
- ✅ Tests hash chaîné déterministe
- ✅ Tests signature RSA vérifiable
- ✅ Tests trigger immuable PostgreSQL

---

## H.9. Démarche Opérationnelle

### H.9.1. Workflow Développement TDD

**Processus Type Nouvelle Fonctionnalité** :

```
1. Spécification métier (UC détaillé Annexe A)
   ↓
2. Décomposition modules impactés
   ↓
3. 🔴 RED: Écrire tests unitaires (logique métier)
   ↓
4. 🟢 GREEN: Implémenter service minimal
   ↓
5. 🔵 REFACTOR: Optimiser code (DRY, SOLID)
   ↓
6. 🔴 RED: Écrire tests intégration (repository)
   ↓
7. 🟢 GREEN: Implémenter persistence SQL
   ↓
8. 🔵 REFACTOR: Optimiser requêtes (index, JOIN)
   ↓
9. 🔴 RED: Écrire test E2E (scénario complet)
   ↓
10. 🟢 GREEN: Intégrer UI → API → DB
    ↓
11. ✅ Validation: CI/CD pipeline vert
```

### H.9.2. Stratégie Pragmatique

**100% TDD Strict** :
- **Modules** : `nf525`, `payments`, `sync`
- **Justification** : Risque légal/financier élevé (conformité NF525, ACID financier, offline critique)

**80% TDD Souple** (tests après code acceptable) :
- **Modules** : `orders`, `menu`, `tables`, `websocket`
- **Justification** : Logique métier complexe mais risque moyen

**50% TDD Opportuniste** :
- **Modules** : `logger`, `config`, `validators` (utilitaires simples)
- **Justification** : Fonctions simples, ROI tests faible

**Tests Manuels Complémentaires** :
- UX mobile (gestes tactiles, ergonomie offline)
- Tests terrain restaurant pilote (validation réelle utilisateurs)
- Intégrations matérielles (TPE bancaire physique, imprimante thermique)

---

## H.10. Limitations et ROI TDD

### H.10.1. Ce que TDD NE Couvre PAS

❌ **UX/UI pixels-perfect** : Tests composants valident comportement (clic button), pas design visuel

❌ **Performance production réelle** : Tests charge locaux ≠ trafic utilisateurs réels (load tests séparés Artillery)

❌ **Bugs "unknown unknowns"** : TDD teste ce qu'on anticipe (pas bugs imprévus edge cases non identifiés)

❌ **Intégrations matérielles** : TPE bancaire physique, imprimante (tests device mocks, validation terrain manuelle)

### H.10.2. Coûts TDD

**Temps Développement Initial** : **+30-40%** (écriture tests + refactoring continu)

**Infrastructure CI/CD** : TestContainers + Docker (configuration complexe initiale)

**Courbe Apprentissage** : Formation équipe mocking, fixtures, TestContainers (2 jours workshop)

**ROI Estimé** :

| Période | Vélocité | Bugs Production | Refactoring | Bilan |
|:--------|:--------:|:---------------:|:-----------:|:------|
| **Court terme (0-6 mois)** | -30% | -40% | +50% confiance | ⚠️ Négatif court terme |
| **Moyen terme (6-18 mois)** | +50% | -60% | +100% vitesse | ✅ Positif moyen terme |
| **Long terme (18+ mois)** | +100% | -80% | +200% vitesse | ✅ Très positif long terme |

**Recommandation** : Investissement TDD rentable à partir de **6 mois** (maintenance V2, évolutions)

---
