# Annexe F : Validation et Justification Architecture

*Ce document est un complément détaillé du Cahier des Charges Principal*

---

## Introduction

Cette annexe présente la validation détaillée de l'architecture logicielle selon les critères **ISO 25010**, les **dilemmes architecturaux** rencontrés, et la justification du choix **3-tiers modulaire** versus alternatives.

---

## F.1. Validation Critères ISO 25010

### F.1.1. Functional Suitability (Adéquation Systèmes Fonctionnels)

| Sous-critère | Score | Validation | Preuve |
|:-------------|:-----:|:-----------|:-------|
| **Completeness** | 100% | ✅ Excellent | 166 fonctionnalités couvertes (Menu + Vin + Split Bill + Offline + NF525) |
| **Correctness** | 95% | ✅ Excellent | PostgreSQL ACID garantit intégrité (split bill, multi-paiements) |
| **Appropriateness** | 90% | ✅ Bon | Stack JavaScript adaptée métier restaurant (ref: Uber Eats, Toast POS) |

**Score Global** : **95/100** ✅

**Justification** :
- **Architecture 3-tiers** couvre 100% des fonctionnalités métier identifiées (C5, 166 UC)
- **PostgreSQL triggers immuables** = seule solution native conforme NF525
- **Mode offline SQLite** = fonctionnalité critique serveurs mobiles (UC7)

---

###F.1.2. Performance Efficiency (Efficacité Performance)

| Sous-critère | Score | Validation | Mesure Réalisée |
|:-------------|:-----:|:-----------|:----------------|
| **Time Behaviour** | 90% | ✅ Excellent | P95 API 87ms (objectif \<200ms ✅ -56%) |
| **Resource Utilization** | 85% | ✅ Bon | Node.js : 180MB RAM, 28% CPU (vs 512MB Java Spring Boot) |
| **Capacity** | 95% | ✅ Excellent | 300 req/h restaurant = 0.3% capacité max Node.js (100k req/h) |

**Score Global** : **90/100** ✅

**Métriques Détaillées IT3** :

| Métrique | Objectif | Réalisé | Écart | Statut |
|:---------|:---------|:--------|:-----:|:------:|
| **Latence P50** | \<100ms | 34ms | -66% | ✅ ⭐⭐⭐ |
| **Latence P95** | \<200ms | 87ms | -56% | ✅ ⭐⭐⭐ |
| **Cache hit ratio** | \>60% | 75% | +15% | ✅ ⭐⭐ |
| **Offline sync** | \<5s (10 cmd) | 2.3s | -54% | ✅ ⭐⭐⭐ |

**Optimisations Clés** :
- ✅ **Redis cache** → -60% charge BDD (TTL 5min menu, 30s stocks, 1h vins)
- ✅ **Connection pooling** → Max 50 connexions PostgreSQL simultanées
- ✅ **NGINX HTTP/2** → Multiplexing requêtes (-40% latence)
- ✅ **Compression Gzip niveau 6** → -70% bande passante (120 Ko → 18 Ko JSON)

**Point d'Attention** :
- ⚠️ **Node.js mono-thread** → Mitigé par clustering PM2 (3 workers, 1 par CPU)

---

### F.1.3. Security (Sécurité)

| Sous-critère | Score | Validation | Mesure |
|:-------------|:-----:|:-----------|:-------|
| **Confidentiality** | 95% | ✅ Excellent | HTTPS TLS 1.3 (Let's Encrypt) + JWT HS256 + bcrypt passwords (salt 10) |
| **Integrity** | 100% | ✅ Parfait | NF525 hash SHA-256 chaîné + signature RSA-2048 (immuabilité garantie) |
| **Non-repudiation** | 100% | ✅ Parfait | Audit logs horodatés + signature RSA (preuve légale fiscale) |
| **Accountability** | 90% | ✅ Excellent | Winston logs → Logstash → ELK Stack (traçabilité complète actions) |
| **Authenticity** | 95% | ✅ Excellent | JWT + RBAC 3 rôles (Serveur, Caissier, Admin) |

**Score Global** : **96/100** ✅

**Conformités Réglementaires** :

| Norme | Exigence | Solution Technique | Validation |
|:------|:---------|:-------------------|:-----------|
| **NF525** | Inaltérabilité transactions | PostgreSQL trigger `BEFORE UPDATE/DELETE → RAISE EXCEPTION` | ✅ 100% |
| **NF525** | Signature cryptographique | Node.js `crypto.createSign('RSA-SHA256')` + clé PEM 2048-bit | ✅ 100% |
| **RGPD** | Consentement explicite | Table `customer_consent` (date, IP, type) | ✅ 100% |
| **RGPD** | Droit à l'oubli | Anonymisation `name → 'Anonyme_<hash>'`, `email → NULL` | ✅ 100% |
| **PCI DSS** | Segmentation réseau | 3 VLAN isolés (10 Monétique, 20 Métier, 30 Management) | ✅ 100% |
| **PCI DSS** | Pas stockage PAN | Uniquement `card_last4` VARCHAR(4), jamais `card_number` | ✅ 100% |

**Architecture Sécuritaire Multi-Couches** :

```
┌─────────────────────────────────────────────┐
│  Couche Réseau                              │
│  - TLS 1.3 (Let's Encrypt auto-renew)      │
│  - VLAN 10 Monétique isolé (PCI DSS Level 3)│
│  - Rate limiting NGINX (300 req/min/IP)    │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│  Couche Application                         │
│  - JWT HS256 (expiration 8h)                │
│  - RBAC granulaire endpoint-level           │
│  - Input validation Joi (SQL injection ✅) │
│  - CORS strict (whitelist domaines)         │
│  - Helmet.js (headers sécurité 12 types)    │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│  Couche Données                             │
│  - bcrypt passwords (salt 10 rounds)        │
│  - PostgreSQL SSL connections               │
│  - Trigger immuable audit_logs (NF525)      │
│  - Masquage auto logs (card_number → ****)  │
└─────────────────────────────────────────────┘
```

---

### F.1.4. Reliability (Fiabilité)

| Sous-critère | Score | Validation | Mesure |
|:-------------|:-----:|:-----------|:-------|
| **Maturity** | 90% | ✅ Excellent | Technologies production-ready (Node.js LTS 20, React 18, PostgreSQL 15) |
| **Availability** | 95% | ✅ Excellent | SLA 99.5% validé (4h downtime/an acceptable restaurant) |
| **Fault Tolerance** | 85% | ✅ Bon | Circuit breaker ERP (Opossum) + retry exponentiel (backoff ×2, max 3×) |
| **Recoverability** | 90% | ✅ Excellent | PostgreSQL WAL replay + Redis RDB snapshot 5min + backup quotidien 3AM |

**Score Global** : **90/100** ✅

**Stratégies Résilience** :

| Type de Panne | Probabilité | Impact | Mitigation Technique | Temps Récupération |
|:--------------|:-----------:|:------:|:---------------------|:------------------:|
| **Perte WiFi mobile** | Haute (20%) | Critique | SQLite offline + sync auto reconnexion | \<3s |
| **ERP indisponible** | Moyenne (5%) | Élevé | Circuit breaker (fail-fast) + retry 3× | \<30s |
| **Crash Node.js** | Faible (2%) | Critique | PM2 auto-restart (watch mode) + health check `/health` | \<2s |
| **Corruption BDD** | Très faible (\<0.1%) | Critique | PostgreSQL WAL replay + backup restauration | \<15min |

**Mode Offline (Détail)** :

```
Détection perte réseau (NetInfo timeout >5s)
        ↓
Badge UI "📶 Mode Offline" (orange persistent)
        ↓
SQLite Local Mobile
  INSERT INTO offline_orders {...}
  INSERT INTO sync_queue {status: 'pending', retry_count: 0}
        ↓
[ATTENTE réseau rétabli]
        ↓
Health check API GET /health → 200 OK
        ↓
Sync batch POST /api/orders/sync (max 10 commandes)
        ↓
Backend validation (idempotency key order_id local)
        ↓
PostgreSQL INSERT + ERP send + NF525 hash
        ↓
UPDATE sync_queue SET status = 'synced', synced_at = NOW()
        ↓
Badge UI "✅ Synchronisé 7 commandes" (green 3s)
```

---

### F.1.5. Maintainability (Maintenabilité)

| Sous-critère | Score | Validation | Mesure |
|:-------------|:-----:|:-----------|:-------|
| **Modularity** | 95% | ✅ Excellent | Services indépendants (`OrderService`, `NF525Service`, `WinePairingService`) |
| **Reusability** | 90% | ✅ Excellent | Code partagé TypeScript Mobile/Web/Backend (40% réduction duplication) |
| **Analysability** | 85% | ✅ Bon | ELK Stack logs + Grafana 4 dashboards + Swagger API docs |
| **Modifiability** | 90% | ✅ Excellent | Architecture 3-tiers (changement couche isolée aucun impact autres) |
| **Testability** | 85% | ✅ Bon | Jest tests unitaires + mocks Redis/PostgreSQL + couverture 78% |

**Score Global** : **89/100** ✅

**Indicateurs Maintenabilité Code** :

| Métrique | Valeur Mesurée | Seuil Acceptable | Seuil Excellence | Statut |
|:---------|:--------------:|:----------------:|:----------------:|:------:|
| **Technical debt ratio** | 3.2% | \<5% | \<3% | ✅ Bon |
| **Code duplication** | 4.1% | \<5% | \<3% | ✅ Bon |
| **Cyclomatic complexity max** | 12 | \<15 | \<10 | ✅ Bon |
| **Test coverage global** | 78% | \>70% | \>85% | ✅ Acceptable |
| **Lines of code (LoC) total** | 18 400 | - | - | - |

**Architecture Modulaire Backend** :

```
src/
├── controllers/        # Couche HTTP (validation inputs, responses)
│   ├── OrderController.ts      (127 LoC)
│   ├── PaymentController.ts    (183 LoC)
│   └── UserController.ts       (94 LoC)
├── services/           # Logique métier pure (testable isolément)
│   ├── OrderService.ts         (254 LoC)
│   │   ├── createOrder(items, tableId)
│   │   ├── calculateTotal(order)
│   │   └── splitBill(order, covers)
│   ├── NF525Service.ts         (412 LoC) ← Critique
│   │   ├── generateHash(transaction)
│   │   ├── signRSA(hash, privateKey)
│   │   ├── chainHash(currentHash, previousHash)
│   │   └── closeDay()
│   ├── WinePairingService.ts   (87 LoC)
│   │   └── recommendWine(dishId) → Wine
│   └── ERPConnector.ts         (198 LoC)
│       └── sendOrder(order) // Opossum circuit breaker
├── middleware/         # Pipeline requêtes
│   ├── auth.middleware.ts      (45 LoC)
│   ├── rbac.middleware.ts      (73 LoC)
│   ├── validate.middleware.ts  (62 LoC)
│   └── errorHandler.ts         (118 LoC)
└── models/             # Prisma ORM schema
    └── schema.prisma           (347 LoC)
```

---

### F.1.6. Autres Critères ISO 25010

| Critère | Score | Validation | Justification Résumée |
|:--------|:-----:|:-----------|:----------------------|
| **Compatibility** | 97/100 | ✅ Excellent | VLAN PCI DSS isolé (100%) + API REST standard ERP (95%) |
| **Usability** | 82/100 | ⚠️ Bon | UI intuitive (85%) mais WCAG AA non conforme (60%) - V2 roadmap |
| **Portability** | 85/100 | ✅ Bon | Docker containers + React Native cross-platform iOS (95%) mais vendor lock-in PostgreSQL (70%) |

---

## F.2. Score Global ISO 25010

### Matrice Pondérée

| Critère ISO 25010 | Poids | Score /100 | Pondéré | Priorité Métier |
|:------------------|:-----:|:----------:|:-------:|:----------------|
| **Functional Suitability** | 25% | 95 | **23.8** | Critique (166 UC) |
| **Performance Efficiency** | 20% | 90 | **18.0** | Critique (P95 \<200ms) |
| **Security** | 20% | 96 | **19.2** | Critique (NF525 légal) |
| **Reliability** | 15% | 90 | **13.5** | Élevée (offline, ERP) |
| **Maintainability** | 10% | 89 | **8.9** | Élevée (évolution V2) |
| **Compatibility** | 5% | 97 | **4.9** | Moyenne (standards) |
| **Portability** | 3% | 85 | **2.6** | Faible (1 restaurant) |
| **Usability** | 2% | 82 | **1.6** | Faible (users formés) |
| **TOTAL** | **100%** | - | **92.5/100** | - |

**Score Global Pondéré** : **92.5/100** 🏆 **Grade A** (Excellence)

### Comparaison Seuils Industrie

| Critère | Notre Score | Seuil Acceptable | Seuil Excellence | Statut |
|:--------|:-----------:|:----------------:|:----------------:|:------:|
| Functional Suitability | 95 | 80 | 90 | ✅ **Excellence** |
| Performance | 90 | 70 | 85 | ✅ **Excellence** |
| Security | 96 | 85 | 92 | ✅ **Excellence** |
| Reliability | 90 | 75 | 88 | ✅ **Excellence** |
| Maintainability | 89 | 70 | 85 | ✅ **Excellence** |
| **GLOBAL** | **92.5** | **75** | **88** | ✅ **Excellence** |

**Validation** : Architecture **dépasse tous les seuils d'excellence** industrie logiciel.

---

## F.3. Dilemmes Architecturaux et Trade-offs

### F.3.1. Dilemme #1 : Accessibilité WCAG vs Time-to-Market

**Contexte** :
- Critère Usability Accessibility = **60/100** (faible, non conforme WCAG AA)
- Application non accessible lecteur écran, contraste insuffisant, navigation clavier limitée

**Alternatives Évaluées** :

| Option | Effort Dev | Délai IT1 | Impact MVP | Conformité |
|:-------|:----------:|:---------:|:-----------|:----------:|
| **A** : WCAG AA dès IT1 | Élevé (120h) | +3 semaines | ⚠️ Retard livraison -21% | ✅ 100% |
| **B** : Reporter V2 | Nul (0h) | 0 semaine | ✅ MVP rapide | ❌ 0% |
| **C** : WCAG partiel (contraste) | Moyen (40h) | +1 semaine | ⚠️ Compromis | ⚠️ 40% |

**Décision : Option B (Reporter V2)** 🏆

**Justification** :
1. **Public cible** : Serveurs/caissiers formés en interne (pas grand public)
2. **Contrainte temps** : MVP 21 semaines → WCAG AA = +14% délai (inacceptable sponsor)
3. **Obligation légale** : Pas de contrainte RGAA pour restaurant privé (hors ERP \>250 salariés)
4. **Roadmap V2** : Accessibilité planifiée itération 5 (4 semaines dédiées)

**Trade-off Accepté** :
- ❌ **Coût** : Exclusion utilisateurs en situation handicap (probabilité \<1% restaurant)
- ✅ **Bénéfice** : Time-to-Market MVP rapide (validation métier prioritaire)

**Critères Décision Formalisés** :
```
Score Business = Impact Métier × Urgence × (1 / Coût Dev)

Score(B) = 9 × 10 × (1/1) = 90
Score(A) = 5 × 10 × (1/3) = 16.7
Score(C) = 6 × 10 × (1/2) = 30

→ Option B score maximal = meilleur choix
```

---

### F.3.2. Dilemme #2 : MFA (Multi-Factor Auth) vs Expérience Utilisateur

**Contexte** :
- Authentification JWT simple `username + password` uniquement
- Pas de MFA (2FA SMS/TOTP) malgré bonnes pratiques sécurité modernes

**Alternatives Évaluées** :

| Option | Sécurité | UX Serveurs | Coût Implémentation | Impact Prise Commande |
|:-------|:--------:|:-----------:|:-------------------:|:---------------------:|
| **A** : MFA obligatoire tous | ✅ Excellent | ❌ Friction haute (30s login) | Moyen (2 sem) | ⚠️ -20% vitesse |
| **B** : MFA optionnel Admin | ✅ Bon | ✅ Fluide serveurs | Faible (1 sem) | ✅ Aucun |
| **C** : Pas de MFA | ⚠️ Moyen | ✅ Fluide (5s login) | Nul | ✅ Aucun |

**Décision : Option C (IT1) puis Option B (IT2)** 🏆

**Justification** :
1. **Contexte usage** : Application interne restaurant, WiFi privé (pas exposition Internet)
2. **Contrainte serveurs** : Prise commande rapide = friction MFA **inacceptable**
3. **Sécurité périmètre** : VLAN privé + rate limiting NGINX (300 req/min) = protection DDoS
4. **Évolution IT2** : MFA optionnel administrateurs uniquement (dashboard sensible)

**Trade-off Accepté** :
- ❌ **Risque** : Compte serveur compromis → Accès commandes (impact limité : pas finances)
- ✅ **Bénéfice** : Login rapide 5s serveurs (vs 30s avec 2FA) = productivité +500%

**Mitigation Risques Sécurité** :
- ✅ **JWT expiration courte** (8h) → Limite fenêtre attaque
- ✅ **Rate limiting login** (5 tentatives/min) → Protection brute-force
- ✅ **Winston logs connexions** → Détection IP suspectes (alertes Slack)
- ✅ **RBAC strict** : Serveurs ≠ Caissiers ≠ Admin (moindre privilège)

---

### F.3.3. Dilemme #3 : PostgreSQL Natif vs ORM Abstraction Complète

**Contexte** :
- Trigger PostgreSQL `BEFORE UPDATE → RAISE EXCEPTION` = SQL natif obligatoire (NF525)
- Risque vendor lock-in PostgreSQL (migration MySQL = effort élevé)

**Alternatives Évaluées** :

| Option | Portabilité | Performance | Maintenabilité | NF525 Compliance |
|:-------|:-----------:|:-----------:|:--------------:|:----------------:|
| **A** : ORM complet (Prisma pur) | ✅ Excellent | ⚠️ Moyen (-15% perf) | ✅ Excellent | ❌ Impossible trigger |
| **B** : Mix ORM + SQL natif | ⚠️ Moyen | ✅ Bon | ⚠️ Moyen (2 syntaxes) | ✅ Oui |
| **C** : SQL natif partout | ❌ Faible | ✅ Excellent | ⚠️ Faible (verbose) | ✅ Oui |

**Décision : Option B (Hybride ORM + SQL Natif)** 🏆

**Répartition Stratégique** :

| Use Case | Approche | Raison | LoC Économisées |
|:---------|:---------|:-------|:---------------:|
| **CRUD menu/orders/tables** | Prisma ORM | Productivité (boilerplate -60%) | -840 LoC |
| **NF525 trigger immuable** | SQL natif PostgreSQL | **Obligation légale** (non-négociable) | - |
| **Recommandations vin (JOIN 3 tables)** | SQL natif optimisé | Performance (-30% vs ORM) | - |
| **Split bill transactions ACID** | Prisma + raw SQL | Garanties transactions natives | - |

**Justification** :
1. **NF525 critique** : Trigger immuable = **obligation légale fiscale** → SQL natif non-négociable
2. **CRUD standard** : Prisma suffisant (menu, orders) → Productivité +60%
3. **Performance queries complexes** : Raw SQL optimisé (vin JOIN 3 tables) → -30% latence

**Trade-off Accepté** :
- ❌ **Coût** : Migration PostgreSQL → MySQL = effort moyen (réécriture triggers ~40h)
- ✅ **Bénéfice** : Performance NF525 optimale + Productivité CRUD

**Probabilité Migration SGBD** : **\<5%** (PostgreSQL = standard industrie Open Source)

---

### F.3.4. Dilemme #4 : Monolithe Modulaire vs Microservices

**Contexte** :
- Architecture 3-tiers monolithique adoptée (1 backend Node.js)
- Tendance industrie = microservices (Netflix, Uber, Airbnb)

**Alternatives Évaluées** :

| Architecture | Complexité | Scalabilité | Coût Infra/An | Time-to-Market | Équipe Requise |
|:-------------|:----------:|:-----------:|:-------------:|:--------------:|:--------------:|
| **A** : Microservices (5 services) | Élevée | ✅ Excellent (horizontal) | 15 000€ (Kubernetes) | Lent (+8 sem) | 8-10 devs |
| **B** : Monolithe modulaire | Faible | ⚠️ Bon (3 instances max) | 2 000€ (Docker Compose) | Rapide (21 sem) | 3-4 devs |

**Décision : Option B (Monolithe Modulaire)** 🏆

**Justification Contexte Métier** :
1. **Échelle** : 1 restaurant, **300 req/h** → 0.3% charge max Node.js (100k req/h possible)
2. **Équipe** : 3-4 développeurs → Pas ressources gérer 5 microservices + Kubernetes
3. **Complexité** : Service mesh (Istio), distributed tracing (Jaeger) = **overkill** total
4. **Coût** : Kubernetes EKS = **+650%** vs Docker Compose (inacceptable restaurant)

**Stratégie Scaling Progressive** :

```
Phase 1 (IT1-IT2) : 1 instance Node.js (suffisant \<500 req/h)
         ↓
Phase 2 (IT3-IT4) : 3 instances Node.js cluster PM2 (→ 1500 req/h)
         ↓
Phase 3 (Multi-restaurants \<10) : Horizontal scaling Docker Compose
  → 1 container groupe par restaurant (isolation données)
         ↓
Phase 4 (\>10 restaurants, \>5000 req/h) : Réévaluer microservices
  → Services indépendants : Réservations, Paiements, Stats
```

**Trade-off Accepté** :
- ❌ **Limite scalabilité** : ~50 restaurants max estimé (3000 req/h)
- ✅ **Bénéfices** : Simplicité (debugging aisé), Time-to-Market (-38%), Coût minimal (-87%)

**Triggers Futurs Migration Microservices** :
- ✅ Volume \>10 restaurants simultanés
- ✅ Équipe \>10 développeurs (organisation Conway's Law)
- ✅ Features indépendantes (ex: système réservations clients externe)

---

### F.3.5. Dilemme #5 : React Native vs Flutter (Mobile)

**Contexte** :
- React Native retenu (score 92/100)
- Flutter compétitif (score 88/100, meilleures performances natives)

**Comparaison Détaillée** :

| Critère | React Native | Flutter | Vainqueur |
|:--------|:------------:|:-------:|:---------:|
| **Cold start app** | 340ms | 280ms | Flutter ✅ (-18%) |
| **Code partagé Web** | ✅ JavaScript commun | ❌ Dart séparé | React Native ✅ |
| **Équipe unifiée** | ✅ JS Full-Stack | ❌ Dart + JS (2 stacks) | React Native ✅ |
| **Offline SQLite R/W** | 8ms | 12ms | React Native ✅ (+50% rapide) |
| **Communauté** | ★★★★★ Meta (2.7M users) | ★★★★☆ Google (1.8M) | React Native ✅ |
| **Courbe apprentissage** | Faible (70% devs JS) | Moyenne (12% devs Dart) | React Native ✅ |

**Décision : React Native** 🏆

**Justification Holistique** :
1. **Code partagé Mobile+Web** : 40% réduction code total (même TypeScript partout)
2. **Équipe unifiée** : 1 stack JavaScript vs 2 stacks indépendantes (Dart Mobile + JS Web)
3. **Performance acceptable** : Cold start 340ms \< 500ms seuil acceptable restaurant
4. **Recrutement facile** : Marché JavaScript ≫ marché Dart (ratio 5:1)

**Trade-off Accepté** :
- ❌ **Coût** : Cold start +60ms vs Flutter (+21% latence initiale)
- ✅ **Bénéfice** : Équipe unifiée (-50% coût formation), code partagé (-40% LoC), productivité

**Cas où Flutter Aurait Gagné** :
- ✅ Application grand public millions users → Performance critique business
- ✅ Pas de web app → Code partagé JS inutile
- ✅ Équipe mobile dédiée → Stack Dart acceptable

---

## F.4. Patterns Architecturaux Appliqués

### F.4.1. Pattern Repository (Data Access Layer)

**Problème** : Couplage fort Controllers ↔ PostgreSQL direct (difficulté tests, changement SGBD)

**Solution** : Couche `repositories/` abstraction BDD

```typescript
// repositories/OrderRepository.ts
export class OrderRepository {
  async findById(orderId: string): Promise<Order> {
    return prisma.order.findUnique({ where: { id: orderId } });
  }
  
  async create(orderData: CreateOrderDTO): Promise<Order> {
    return prisma.order.create({ data: orderData });
  }
  
  // Méthode métier spécifique (pas CRUD générique)
  async findOrdersWithSplitBill(tableId: number): Promise<Order[]> {
    return prisma.$queryRaw`
      SELECT o.*, COUNT(DISTINCT p.cover_number) as split_count
      FROM orders o
      JOIN payments p ON o.id = p.order_id
      WHERE o.table_id = ${tableId}
      GROUP BY o.id
      HAVING COUNT(DISTINCT p.cover_number) > 1
    `;
  }
}
```

**Bénéfices** :
- ✅ **Testabilité** : Mock `OrderRepository` dans tests unitaires
- ✅ **Portabilité** : Changement SGBD = 1 seule couche modifiée
- ✅ **Maintenabilité** : Queries SQL complexes isolées

---

### F.4.2. Pattern Circuit Breaker (Tolérance Pannes ERP)

**Problème** : ERP "QuiCuisineIci" indisponible → cascade failures (timeout 30s × 10 commandes = 300s blocage)

**Solution** : Circuit Breaker Opossum

```javascript
const CircuitBreaker = require('opossum');

const erpOptions = {
  timeout: 5000,                    // 5s max par appel
  errorThresholdPercentage: 50,    // OPEN si ≥50% erreurs
  resetTimeout: 60000,              // Retry après 60s
  volumeThreshold: 5                // Min 5 appels avant évaluation
};

const sendToERPRaw = async (order) => {
  const response = await axios.post('http://erp.local/orders', order, {
    timeout: 4500  // légèrement < circuit breaker timeout
  });
  return response.data;
};

const erpBreaker = new CircuitBreaker(sendToERPRaw, erpOptions);

// Fallback stratégie : enregistrer queue retry
erpBreaker.fallback((order) => {
  logger.warn(`ERP circuit OPEN, queueing order ${order.id}`);
  return queueForRetry(order);  // Table retry_queue PostgreSQL
});

// Statistiques monitoring
erpBreaker.on('open', () => {
  logger.error('ERP circuit OPEN - Fallback mode activé');
  alertSlack('🔴 ERP QuiCuisineIci indisponible - Mode dégradé');
});

erpBreaker.on('halfOpen', () => {
  logger.info('ERP circuit HALF_OPEN - Test reconnexion...');
});

erpBreaker.on('close', () => {
  logger.info('ERP circuit CLOSED - Service rétabli');
  alertSlack('✅ ERP QuiCuisineIci rétabli');
});

// Utilisation dans OrderService
async createOrder(orderData) {
  const order = await OrderRepository.create(orderData);
  
  try {
    const erpResponse = await erpBreaker.fire(order);
    order.erp_id = erpResponse.kitchen_order_id;
  } catch (err) {
    // Circuit OPEN ou erreur fallback
    order.erp_status = 'pending_retry';
  }
  
  return order;
}
```

**États Circuit Breaker** :

```
CLOSED (Normal) :
  5 succès consécutifs → Rest normal
  3 erreurs / 5 appels (60%) → OPEN
         ↓
OPEN (Blocage) :
  Tous appels → Fallback immédiat (pas d'appel ERP)
  Attente 60s timeout → HALF_OPEN
         ↓
HALF_OPEN (Test) :
  1 appel test ERP :
    - Succès → CLOSED (rétablissement)
    - Échec → OPEN (60s supplémentaires)
```

**Bénéfices** :
- ✅ **Isolation pannes** : ERP down ≠ API down (fail-fast, pas cascade)
- ✅ **Récupération auto** : Test reconnexion automatique (pas intervention manuelle)
- ✅ **Monitoring** : Events Slack temps réel état ERP

---

### F.4.3. Pattern Pub/Sub (Notifications Temps Réel)

**Problème** : Notifications "Plat Prêt" ERP → Serveurs (polling inefficace 30s × 3 mobiles = 90 req/min)

**Solution** : Redis Pub/Sub + WebSocket Socket.io

```javascript
// Backend WebSocket Server
const io = require('socket.io')(server);
const redis = require('redis');
const subscriber = redis.createClient();

subscriber.subscribe('dish_ready_channel');

io.on('connection', (socket) => {
  // Serveur s'inscrit aux notifications de sa table
  socket.on('subscribe_table', (tableId) => {
    socket.join(`table_${tableId}`);
    logger.info(`Socket ${socket.id} subscribed to table ${tableId}`);
  });
  
  socket.on('disconnect', () => {
    logger.info(`Socket ${socket.id} disconnected`);
  });
});

// Subscriber Redis → Broadcast WebSocket
subscriber.on('message', (channel, message) => {
  const event = JSON.parse(message);
  
  if (channel === 'dish_ready_channel') {
    io.to(`table_${event.table_id}`).emit('dish_ready', {
      dish_name: event.dish_name,
      table_id: event.table_id,
      timestamp: Date.now()
    });
  }
});

// Endpoint callback ERP
app.post('/api/kitchen/notify', (req, res) => {
  const { table_id, dish_name, status } = req.body;
  
  // Publish Redis
  publisher.publish('dish_ready_channel', JSON.stringify({
    table_id,
    dish_name,
    status
  }));
  
  res.status(200).json({ notified: true });
});
```

**Flux Complet** :

```
ERP QuiCuisineIci (Chef marque plat prêt)
         ↓
    POST /api/kitchen/notify {table_id: 5, dish_name: "Plat Principal"}
         ↓
Backend Node.js → Redis PUBLISH 'dish_ready_channel'
         ↓
Subscriber Redis → Event 'message' détecté
         ↓
WebSocket Broadcast → io.to('table_5').emit('dish_ready')
         ↓
Mobile Serveur Table 5 → Event listener 'dish_ready'
         ↓
Notification Push Locale "🍽️ Plat Principal table 5 prêt"
```

**Bénéfices** :
- ✅ **Temps réel** : Latence \<100ms (vs polling 30s = latence moyenne 15s)
- ✅ **Efficacité réseau** : 0 req/min idle (vs polling 90 req/min)
- ✅ **Scalabilité** : Redis Pub/Sub gère 100k messages/s (overkill restaurant)

---

### F.4.4. Pattern Middleware Chain (Pipeline Requêtes)

**Problème** : Logique transversale dupliquée (auth, logs, validation) dans chaque Controller

**Solution** : Express Middleware Chain

```javascript
// Middlewares
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Missing token' });
  
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    res.status(403).json({ error: 'Invalid token' });
  }
};

const rbacMiddleware = (allowedRoles) => (req, res, next) => {
  if (!allowedRoles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  next();
};

const validateMiddleware = (schema) => (req, res, next) => {
  const { error } = schema.validate(req.body);
  if (error) return res.status(400).json({ error: error.details[0].message });
  next();
};

const logMiddleware = (req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    logger.info(`${req.method} ${req.path} - ${res.statusCode} - ${Date.now() - start}ms`);
  });
  next();
};

// Application Pipeline
app.use(logMiddleware);  // Global : toutes requêtes loggées

app.post('/api/orders',
  authMiddleware,                           // 1. Vérifier JWT
  rbacMiddleware(['Serveur', 'Admin']),     // 2. Vérifier rôle
  validateMiddleware(orderSchema),          // 3. Valider payload Joi
  OrderController.create                    // 4. Logique métier
);
```

**Pipeline Visuel** :

```
HTTP POST /api/orders {items: [...], table_id: 5}
         ↓
[1] logMiddleware               → Winston log "POST /api/orders - START"
         ↓
[2] authMiddleware              → JWT décodé → req.user = {id: 12, role: 'Serveur'}
         ↓
[3] rbacMiddleware(['Serveur']) → Vérification req.user.role ∈ ['Serveur','Admin'] ✅
         ↓
[4] validateMiddleware(schema)  → Joi validation items[], table_id ✅
         ↓
[5] OrderController.create()    → Business logic
         ↓
[6] logMiddleware (finish)      → Winston log "POST /api/orders - 201 - 87ms"
         ↓
HTTP 201 Created {order_id: 42}
```

**Bénéfices** :
- ✅ **Réutilisabilité** : Middleware réutilisé 17 endpoints (DRY)
- ✅ **Testabilité** : Tests unitaires middleware isolés
- ✅ **Maintenabilité** : Ajout nouveau middleware = 1 ligne (ex: rate limit)

---

## F.5. Principes SOLID Appliqués

| Principe | Application Concrète | Bénéfice |
|:---------|:---------------------|:---------|
| **S**ingle Responsibility | `OrderService` ≠ `NF525Service` ≠ `WinePairingService` (services spécialisés) | Testabilité, Maintenabilité |
| **O**pen/Closed | Middleware extensibles (ajout `rateLimitMiddleware` sans modifier existants) | Évolutivité |
| **L**iskov Substitution | Interface `IRepository` → `OrderRepository`, `PaymentRepository` interchangeables | Portabilité |
| **I**nterface Segregation | Interface `IOrderService` ≠ `INF525Service` (pas interface monolithique) | Découplage |
| **D**ependency Inversion | Controllers dépendent d'interfaces `IOrderService`, pas implémentation concrète | Testabilité (mocks) |

---

## F.6. Justification Architecture 3-Tiers vs Alternatives

### Comparaison Architectures

| Architecture | Complexité | Coût Dev | Scalabilité | Convient Projet |
|:-------------|:----------:|:--------:|:-----------:|:---------------:|
| **3-Tiers Modulaire** | ⭐⭐☆☆☆ | 43k€ | ⭐⭐⭐☆☆ | ✅ **OUI** (1-50 restaurants) |
| **Microservices** | ⭐⭐⭐⭐⭐ | 85k€ | ⭐⭐⭐⭐⭐ | ❌ Overkill (\>50 restaurants) |
| **Serverless (Lambda)** | ⭐⭐⭐☆☆ | 52k€ | ⭐⭐⭐⭐☆ | ⚠️ Cold start problème |
| **Monolithe Classique** | ⭐⭐☆☆☆ | 38k€ | ⭐⭐☆☆☆ | ⚠️ Pas évolutif V2 |

**Décision : 3-Tiers Modulaire** 🏆

**Justification Holistique** :
1. **Échelle adaptée** : 1 restaurant = 300 req/h → Architecture simple suffisante
2. **Évolutivité future** : Modularité permet migration progressive microservices si besoin
3. **Coût optimal** : -49% vs microservices, +13% vs monolithe (acceptable ROI modularité)
4. **Équipe réduite** : 3-4 devs gèrent facilement (vs 8-10 devs microservices)

---

## F.7. Évolution Qualité IT1 → IT4

### Synthèse Progression

| Critère | IT1 MVP | IT2 Sécurité | IT3 Performance | IT4 Observabilité | Gain Total |
|:--------|:-------:|:------------:|:---------------:|:-----------------:|:----------:|
| **Fiabilité** | 🟡 3/5 (60%) | 🟡 3/5 (60%) | 🟢 5/5 (100%) | 🟢 5/5 (100%) | **+40%** |
| **Performance** | 🟢 4/5 (80%) | 🟢 4/5 (80%) | 🟢 5/5 (100%) | 🟢 5/5 (100%) | **+20%** |
| **Sécurité** | 🔴 2/5 (40%) | 🟢 5/5 (100%) | 🟢 5/5 (100%) | 🟢 5/5 (100%) | **+60%** |
| **Maintenabilité** | 🟢 4/5 (80%) | 🟢 4/5 (80%) | 🟢 4/5 (80%) | 🟢 5/5 (100%) | **+20%** |
| **Évolutivité** | 🟡 3/5 (60%) | 🟡 3/5 (60%) | 🟢 4/5 (80%) | 🟢 5/5 (100%) | **+40%** |
| **TOTAL** | **16/25** (64%) | **19/25** (76%) | **23/25** (92%) | **25/25** (100%) | **+36%** |

**Progression Globale** : **64% → 100%** (+36 points en 21 semaines)

---
