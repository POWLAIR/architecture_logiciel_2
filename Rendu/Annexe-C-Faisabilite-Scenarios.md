# Annexe C : Faisabilité Technique et Choix du Scénario

*Ce document est un complément détaillé du Cahier des Charges Principal*

---

## Introduction

Cette annexe consolide :
1. **L'analyse de faisabilité technique** :Elements faisables, risques conditionnels, points critiques
2. **Les scénarios alternatifs** : Comparaison architecturale et budgétaire de 4 approches
3. **Le choix du scénario retenu** (Scénario A) : Justification et hypothèses de réalisation

---

## C.1. Éléments FAISABLES (Complexité Maîtrisable)

### C.1.1. Application Mobile Serveur (Android)

**Besoin** : 3 serveurs avec des téléphones Android pour prise de commande en salle.

**Analyse de faisabilité** :
- ✅ **Faisable** : Technologies matures disponibles
  - React Native 0.73 (choix retenu)
  - Flutter 3.16 (option alternative)
  - Kotlin natif (maximum performance)
- ✅ Écosystème Android bien documenté
- ✅ Accès aux API natives (Notifications push Firebase, stockage local SQLite)

**Complexité** : ⭐⭐ Faible (Standard industriel)

**Technologies identifiées** :
- Framework : React Native (cross-platform Android prioritaire)
- État : Redux Toolkit ou Context API
- Navigation : React Navigation 6.x
- Storage offline : SQLite + react-native-sqlite-storage
- Notifications : Firebase Cloud Messaging (FCM)

---

### C.1.2. Système de Caisse avec Certification NF525

**Besoin** : Encaissement conforme à la loi anti-fraude TVA.

**Analyse de faisabilité** :
- ✅ **Faisable** : Nombreuses bibliothèques/frameworks certifiables existent
- ✅ Référentiels NF525 publics (critères ISCA documentés)
- ⚠️ **Contrainte** : Nécessite audit par organisme agréé (AFNOR/LNE)
  - Délai : 3-6 mois post-développement
  - Coût : ~3 500€ (audit + redevance 1ère année)

**Complexité** : ⭐⭐⭐ Moyenne (Certification longue mais processus connu)

**Obligations techniques NF525** :
- ✅ **Inaltérabilité** : Hash SHA-256 chaîné (chaque clôture référence précédente)
- ✅ **Sécurisation** : Signature RSA-2048 avec certificat qualifié
- ✅ **Conservation** : Archivage 6 ans minimum (base PostgreSQL)
- ✅ **Archivage** : Tickets Z quotidiens (PDF imprimables)

**Technologies identifiées** :
- Crypto : Node.js `crypto` module (SHA-256, RSA built-in)
- Stockage immutable : PostgreSQL triggers `BEFORE UPDATE/DELETE` → Exception
- PDF : `pdfkit` ou `puppeteer` pour génération tickets Z
- Archivage : FileSystem local + backup S3/Backblaze B2

---

### C.1.3. Gestion des Stocks en Temps Réel

**Besoin** : Affichage instantané du nombre de plats disponibles sur les mobiles serveurs.

**Analyse de faisabilité** :
- ✅ **Faisable** : Lecture de données via API REST depuis l'ERP "QuiCuisineIci"
- ✅ Technologies de push/notification (WebSockets, Firebase Cloud Messaging)
- ✅ Cache Redis pour réduire latence (TTL 30s)

**Complexité** : ⭐⭐ Faible (Architecture classique)

**Architecture proposée** :
```
Mobile Serveur
    ↓ GET /api/menu/availability
Backend API
    ↓ Cache HIT/MISS
Redis Cache (TTL 30s)
    ↓ Si MISS → fetch
API Stocks Externe (ERP)
```

**Métriques cibles** :
- Latence P95 < 200ms (cache hit : <50ms)
- Taux cache hit ≥ 80%
- Fallback base de données si API Stocks down

---

### C.1.4. Paiement Divisé / Groupé (Split Bill)

**Besoin** : Chacun paie sa part OU une personne paie pour plusieurs.

**Analyse de faisabilité** :
- ✅ **Faisable** : Modélisation base de données avec liaison `ORDER_ITEMS → Couvert`
- ✅ Interface de sélection d'articles pour facturation (déjà implémenté dans nombreux POS)
- ✅ Validation cohérence via transaction ACID PostgreSQL

**Complexité** : ⭐⭐ Faible (Logique métier classique)

**Modèle de données clé** :
```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY,
  order_id UUID REFERENCES orders(id),
  cover_number INTEGER NOT NULL,  -- Couvert 1, 2, 3...
  amount DECIMAL(10,2) NOT NULL,
  method VARCHAR(20),             -- 'CB', 'CASH', 'TR'
  CONSTRAINT uk_payment_per_cover UNIQUE (order_id, cover_number)  -- Anti-doublon
);
```

**Règles métier** :
- ✅ Somme `payments.amount` doit égaler `orders.total` (Trigger PostgreSQL)
- ✅ Interdiction paiement double même couvert (Contrainte UK)
- ✅ Items partagés (ex: bouteille vin) → Division automatique équitable

---

## C.2. Éléments à RISQUE (Faisabilité Conditionnelle)

### C.2.1. Intégration avec l'ERP "QuiCuisineIci"

**Besoin** : Interface bidirectionnelle (Envoi commandes + Réception notifications "plat prêt").

**Analyse de risque** :
- ⚠️ **Faisabilité CONDITIONNÉE** à l'obtention de :
  1. **Documentation complète de l'API** (Swagger/OpenAPI ou équivalent)
  2. **Environnement de test/sandbox** avec credentials d'accès
  3. **SLA garantie** (latence < 500ms P95, uptime ≥ 99%)
  4. **Engagement stabilité** (pas de breaking change sans préavis 6 mois)

**Points de blocage possibles** :
- 🔴 API non documentée ou fermée (pasAccès développeurs)
- 🔴 Authentification propriétaire inaccessible
- 🔴 Pas d'environnement de test (risque casser prod cuisine)

**Complexité** : ⭐⭐⭐⭐ Élevée (Dépendance externe critique)

**Stratégies de mitigation** :

#### Niveau 1 : Solution Idéale ✅
- **Action** : Obtenir documentation officielle API + sandbox
- **Délai** : +0 semaine (si fourni immédiatement)
- **Coût** : 0 € additionnel

#### Niveau 2 : Solution Acceptable ⚠️
- **Action** : Reverse Engineering de l'API (capture trafic réseau `mitmproxy`)
- **Risque juridique** : Possible violation CGU ERP
- **Délai** : +2 semaines (analyse + développement)
- **Coût** : +3 000 € (expert sécurité)

#### Niveau 3 : Plan B (Mode Manuel) 🔴
- **Action** : Abandon intégration temps réel → Impression tickets papier cuisine
- **Impact** : Perte valeur ajoutée principale
- **Délai** : -3 semaines (simplification architecture)
- **Coût** : -8 000 € (module intégration supprimé)
- ⚠️ **Non recommandé** : Ne répond pas au besoin initial

**→ Décision requise AVANT démarrage projet**

---

### C.2.2. Conseil Automatique "Accord Mets-Vins"

**Besoin** : L'application propose **automatiquement** un vin unique par plat.

**Analyse de faisabilité** :

#### Version Simple (Retenue) ✅
- **Implémentation** : Table de correspondance statique `MENU_ITEMS.wine_pairing_id → WINES.id`
- **Gestion** : Mise à jour manuelle par Admin via interface web
- **Complexité** : ⭐⭐ Faible
- **Avantages** :
  - Rapide à développer (1 semaine)
  - Contrôle total restaurateur
  - Évolutif (V2 algorithme intelligent)

**Modèle de données** :
```sql
CREATE TABLE menu_items (
  id UUID PRIMARY KEY,
  name VARCHAR(255),
  wine_pairing_id UUID REFERENCES wines(id) NULL  -- 1 vin suggéré/plat
);

CREATE TABLE wines (
  id UUID PRIMARY KEY,
  name VARCHAR(255),
  vintage INTEGER,
  region VARCHAR(100),
  price DECIMAL(10,2)
);
```

#### Version Avancée (V2 Potentielle) 🔮
- **Implémentation** : Moteur de recommandation Machine Learning
  - Prise en compte stocks vins temps réel
  - Algorithme apprentissage préférences clients (historique commandes)
  - Suggestions personnalisées par profil
- **Complexité** : ⭐⭐⭐⭐⭐ Très élevée
- **Prérequis** :
  - Data Scientist (6-12 mois données historiques)
  - Infrastructure ML (Python scikit-learn/TensorFlow)
  - Budget +15 000€

**→ Recommandation : Version Simple V1, évolution V2 si ROI démontré**

---

### C.2.3. Carte Changeante "Tous les Jours"

**Besoin** : *« La carte évolue selon la saison, tous les jours. »*

**Analyse de faisabilité** :
- ✅ **Techniquement Faisable** : Interface d'administration pour modifier la carte
- ⚠️ **Risque Opérationnel** : Si changement quotidien **littéral** :
  - Qui met à jour ? (Chef ou Admin)
  - Quand ? (le matin avant 11h obligatoire)
  - Synchronisation avec ERP stocks → Timing critique
  - Formation nécessaire (interface simple et rapide <5 min)

**Ambiguïté non levée** :
> 🔶 **Clarification requise avec client** : La carte change-t-elle réellement **365 fois/an** ou s'agit-il de **variations hebdomadaires/saisonnières** ?

**Hypothèse de travail retenue** :
- **Mise à jour hebdomadaire** (chaque lundi matin)
- Interface admin simplifiée : Import CSV ou saisie formulaire
- Historique versions carte (audit trail)

**Complexité** :
- Si quotidien : ⭐⭐⭐ Moyenne (Contrainte opérationnelle forte)
- Si hebdo/saison : ⭐⭐ Faible

---

### C.2.4. Mode Dégradé (Perte de Wifi)

**Besoin** : (**Non exprimé** dans le cahier des charges initial, mais **critique**)

**Analyse de risque** :
- ⚠️ **Risque d'Infaisabilité Totale** en cas de panne réseau si aucun mode offline
- ✅ **Faisable** avec architecture Offline-First :
  - Cache local stocks (dernière valeur connue)
  - File d'attente locale commandes (envoi différé)
  - Synchronisation automatique à la reconnexion

**Architecture Offline-First proposée** :
```
Mobile Serveur
    ↓ Détection perte réseau (timeout >5s)
SQLite Local
    ↓ Enregistrement commande `offline_orders`
    ↓ Ajout queue `sync_queue` (attente réseau)
    ↓ **RÉSEAU RÉTABLI**
Backend API
    ↓ POST /api/orders/sync (batch 10 commandes)
PostgreSQL + ERP
```

**Technologies** :
- SQLitesent local (react-native-sqlite-storage)
- Queue persistante (Workmanager Android ou Background Tasks API)
- Détection réseau : `@react-native-community/netinfo`

**Limitations acceptables** :
- ⚠️ Stocks affichés en mode dégradé peuvent être **obsolètes** (risque vente plat épuisé)
- UI : Alerte visuelle "📶 Mode Offline - Données non synchronisées depuis 14:32"

**Complexité** : ⭐⭐⭐ Moyenne (Architecture complexe mais bien documentée)

**→ Implémentation obligatoire (Itération IT3 - Résilience)**

---

## C.3. Éléments POTENTIELLEMENT INFAISABLES

### C.3.1. Garantie Conformité PCI DSS sans Audit Initial

**Contexte** : Le restaurant accepte les cartes bancaires via TPE.

**Problème identifié** :
- 🔴 **Infaisable** de garantir la conformité PCI DSS **sans audit réseau préalable**
- Le cahier des charges ne mentionne pas l'état actuel du réseau :
  - Le Wifi est-il segmenté (VLAN) ?
  - Le TPE est-il déjà isolé du réseau public ?
  - Existe-t-il un pare-feu?

**Impact si non-conformité** :
- Risque juridique (amende 7 500€ par an si contrôle DGFIP)
- Blocage certification NF525 (audit QSA requis)
- Impossibilité légale d'utiliser le système en production

**Exigences PCI DSS** :
1. **Isolation réseau TPE** : VLAN dédié (VLAN 10 Monétique)
2. **Chiffrement transit** : TLS 1.3 minimum
3. **Firewall** : Blocage VLAN 20 (métier) → VLAN 10 (monétique) 
4. **Pas de stockage PAN** : Uniquement `card_last4` dans logs (masquage)

**Solution** :
```
PRÉ-REQUIS BLOQUANT : Audit réseau PCI DSS par un QSA (Qualified Security Assessor) AVANT démarrage projet.
```

**Plan de mise en conformité si nécessaire** :
- Équipements : Switch manageable + Firewall (Ubiquiti EdgeRouter ou PfSense)
- Configuration : Segmentation 3 VLAN
  - VLAN 10 : Réseau Monétique (TPE + Caisse)
  - VLAN 20 : Réseau Métier (Mobiles Serveurs + ERP)
  - VLAN 30 : Réseau Invités (Wifi public si futur)
- Budget : 5 000€ - 12 000€ selon état initial
- Délai : +4 semaines

**Faisabilité** : ❌ **Conditionnée** à la mise en conformité préalable de l'infrastructure.

---

### C.3.2. Synchronisation Parfaite Multi-Dispositifs Temps Réel

**Problème théorique** :
- 3 mobiles + 1 caisse + ERP → Risque de **race conditions** (deux serveurs commandent simultanément le dernier plat disponible)

**Analyse de faisabilité** :
- ⚠️ **Complexe** mais **faisable** avec :
  - **Verrouillage optimiste** (Optimistic Locking) en base de données
  - **Transaction ACID** (Atomicité, Cohérence, Isolation, Durabilité)
  - **WebSockets** pour notification instantanée de changements stocks

**Architecture proposée** :
```sql
-- Exemple Optimistic Locking PostgreSQL
CREATE TABLE menu_items (
  id UUID PRIMARY KEY,
  stock INTEGER NOT NULL,
  version INTEGER NOT NULL DEFAULT 0  -- Version counter
);

-- Transaction sécurisée
BEGIN;
SELECT stock, version FROM menu_items WHERE id = '...' FOR UPDATE;  -- Lock
UPDATE menu_items 
SET stock = stock - 1, version = version + 1
WHERE id = '...' AND version = <current_version>;
-- Si version ≠ : ROLLBACK (conflit détecté)
COMMIT;
```

**Notification temps réel** :
- Backend : Redis Pub/Sub → WebSocket (Socket.io)
- Mobile : Écoute channel `stock_updates` → Refresh UI automatique

**Risque résiduel acceptable** :
- Latence réseau WiFi → Décalage ~500ms (acceptable)
- Si latence >2s → Expérience légèrement dégradée (tolérable)

**Faisabilité** : ✅ **Faisable** avec architecture robuste (transaction BDD + notifications push)

**Complexité** : ⭐⭐⭐⭐ Élevée (Système distribué temps réel)

---

### C.3.3. Interface "Sans Formation" pour Personnel Non Technique

**Besoin implicite** : Les serveurs doivent adopter l'outil rapidement.

**Analyse de réalisme** :
- 🔴 **Infaisable** de concevoir une interface **zéro formation** pour un système métier complexe
- Même les meilleurs UX/UI nécessitent une **prise en main minimale**

**Recommandation obligatoire** :
- **1 journée de formation en présentiel** (8h)
  - Matin : Mobile serveurs (prise commandes, stocks, split bill)
  - Après-midi : Web caissiers (encaissements, clôture NF525)
- **Tutoriels vidéo intégrés** dans l'app (5 vidéos × 3 min)
- **Mode démo/sandbox** pour s'entraîner sans impacter production

**Budget formation** :
- Formateur : 800€/jour
- Documentation utilisateur illustrée : 1 200€
- Vidéos tutoriels : 1 500€ (freelance motion design)
- **Total** : ~3 500€ (inclus dans budget global)

**Faisabilité** : ✅ Faisable **avec formation** (non négociable)

---

## C.4. Synthèse des Risques d'Infaisabilité

| Élément | Faisabilité | Condition(s) Bloquante(s) | Priorité Action | Itération |
| :--- | :---: | :--- | :---: | :---: |
| **Intégration ERP** | 🟡 Conditionnelle | Documentation API + Sandbox | ⚠️ **CRITIQUE** | IT1 |
| **Conformité PCI DSS** | 🟡 Conditionnelle | Audit réseau préalable | ⚠️ **CRITIQUE** | IT0 (Phase 0) |
| **Certification NF525** | 🟢 Faisable | Délai 3-6 mois à intégrer | 🔵 Important | IT2 |
| **Mode Dégradé Offline** | 🟢 Faisable | Architecture Offline-First | 🔵 Important | IT3 |
| **Conseil Vin Automatique** | 🟢 Faisable | Version Simple (Pas ML en V1) | 🟢 Normal | IT1 |
| **Sync Temps Réel** | 🟢 Faisable | Architecture robuste (ACID + WebSocket) | 🔵 Important | IT1-IT2 |
| **Carte Quotidienne** | 🟡 Ambiguë | Clarifier fréquence réelle (client) | 🔵 Important | IT0 |
| **Formation Utilisateurs** | 🟢 Faisable | Budget formation 3 500€ | 🔵 Important | IT4 (Livraison) |

**Légende** :
- 🟢 Faisable (risque maîtrisé)
- 🟡 Conditionnelle (blocage possible)
- 🔴 Infaisable (en l'état actuel)

---

## C.5. Scénarios Alternatifs Comparés

### Scénario A : Projet Complet (Architecture Cible) ✅

**Conditions de réalisation** :
- ✅ API ERP documentée et accessible
- ✅ Infrastructure réseau conforme PCI DSS
- ✅ Budget et délais respectés

**Architecture** :
```
┌─────────────┐
│ Mobile      │──┬──▶ API Backend (Node.js Express)
│ Serveur (x3)│  │         ↓
└─────────────┘  │    PostgreSQL 15 (ACID)
                 │         ↓
┌─────────────┐  │    Module NF525 (Hash SHA-256 + RSA)
│ Caisse      │──┘         ↓
│ (Web React) │──── TPE (VLAN 10 PCI DSS)
└─────────────┘     
                 ↔ ERP QuiCuisineIci (REST bidirectionnel)
```

**Fonctionnalités** :
- ✅ Temps réel (stocks, notifications WebSocket)
- ✅ Mode offline de secours (SQLite mobile)
- ✅ Paiements CB + Certification NF525
- ✅ Dashboard propriétaire (rapports CA)

**Budget** : **43 700 € HT**  
**Détail budget** :
- Développement (500h × 70€/h) : 35 000€
- Audit PCI DSS : 2 000€
- Certification NF525 : 3 500€
- Formation (documenteur + formateur) : 3 200€

**Délai** : **5 mois**
- Mois 1-2 : IT1 MVP Fonctionnel (8 sem)
- Mois 3 : IT2 Sécurité + NF525 (6 sem)
- Mois 4 : IT3 Résilience Offline (4 sem)
- Mois 5 : IT4 Scalabilité + Formation (3 sem)

**Risques** :
- 🟢 Technique : Faible (si API ERP accessible)
- 🟡 Planning : Moyen (dépend certification NF525)

**→ SCÉNARIO RETENU**

---

### Scénario B : Projet Simplifié (Sans Intégration ERP Temps Réel) ⚠️

**Conditions** :
- 🔴 API ERP inaccessible OU Reverse engineering refusé
- ✅ Infrastructure réseau OK

**Architecture** :
```
┌─────────────┐
│ Mobile      │──▶ API Backend (Stocks Manuels MAJ 9h)
│ Serveur (x3)│           ↓
└─────────────┘      PostgreSQL
                          ↓
┌─────────────┐      Module NF525
│ Caisse      │───────────┤
└─────────────┘          TPE

Cuisine : Impression papier commandes (pas de lien ERP)
```

**Fonctionnalités** :
- ✅ Prise de commande mobile (stocks statiques journaliers)
- ✅ Encaissement certifié NF525
- ❌ Pas de notification "plat prêt" automatique
- ❌ Pas de synchronisation temps réel avec cuisine

**Workflow** :
1. Chef met à jour stocks manuellement 9h dans interface web
2. Serveurs prennent commandes sur mobile
3. **Impression ticket cuisine** (imprimante thermique)
4. Chef sonne cloche quand plat prêt (pas de notification mobile)

**Budget** : **32 000 € HT** (-26%)  
**Délai** : **3,5 mois**

**Inconvénients majeurs** :
- ⚠️ Risque vendre plat épuisé (stocks non temps réel)
- ⚠️ Retour fonctionnement semi-manuel
- ⚠️ Perte valeur ajoutée principale

**→ NON RETENU (plan B silen API ERP échoue définitivement)**

---

### Scénario C : Solution Minimale (Caisse Certifiée Uniquement) 🔴

**Conditions** :
- 🔴 API ERP inaccessible
- 🔴 Budget réduit de moitié (<20k€)
- 🔴 Délai court (<2 mois)

**Architecture** :
```
┌─────────────┐
│ Caisse      │──▶ Module NF525 ──▶ PostgreSQL
│ (Web React) │           ↓
└─────────────┘          TPE

Mobiles Serveurs : ❌ ABANDON (commandes carnet papier)
```

**Fonctionnalités** :
- ✅ Encaissement certifié NF525 (obligation légale)
- ✅ Paiements CB sécurisés
- ❌ Pas d'application mobile serveurs
- ❌ Retour au papier/crayon pour commandes

**Budget** : **18 000 € HT** (-59%)  
**Délai** : **2 mois**

**Impact** :
- ❌ Perte totale de la valeur ajoutée "digitalisation salle"
- ❌ Ne répond qu'à l'obligation légale (NF525)

**→ NON RETENU (ne satisfait pas besoin initial client)**

---

### Scénario D : Hybride (ERP Interne Maison) 🟡

**Conditions** :
- 🔴 API ERP externe définitivement inaccessible
- ✅ Client accepte budget augmenté
- ✅ Délai étendu (+2 mois)

**Solution** : Développer un **mini-ERP cuisine interne** pour remplacer "QuiCuisineIci"

**Architecture** :
```
┌─────────────┐
│ Mobile      │──▶ API Backend ──┬──▶ BDD Salle + Cuisine
│ Serveur (x3)│                  │
└─────────────┘                  ├──▶ Module NF525
                                 │
┌─────────────┐                  │
│ Caisse      │──────────────────┤
└─────────────┘                  ▼
                 ┌─────────────────────────────────┐
                 │ Interface Cuisine (Web/Tablette)│
                 │ - Réception commandes           │
                 │ - Gestion stocks                │
                 │ - Notification "Plat prêt"      │
                 └─────────────────────────────────┘
```

**Fonctionnalités** :
- ✅ Autonomie totale (pas de dépendance externe)
- ✅ Intégration native temps réel
- ✅ Possibilité d'ajouter features spécifiques restaurant

**Inconvénients** :
- ⚠️ Double développement (salle + cuisine)
- ⚠️ Formation Chef à nouvel outil (résistance potentielle)
- ⚠️ Pas de gestion avancée stocks (contrairement à ERP mature)

**Budget** : **58 000 € HT** (+33%)  
**Délai** : **7 mois**

**Risque majeur** :
- 🔴 Le Chef a explicitement refusé de changer son ERP actuel
- Approche diplomatique requise (démonstration valeur ajoutée)

**→ NON RETENU (option ultime si Scénario A impossible + Chef accepte changement)**

---

## C.6. Tableau Comparatif Final des Scénarios

| Critère | **Scénario A** (Complet) ✅ | Scénario B (Simplifié) | Scénario C (Minimal) | Scénario D (Hybride) |
| :--- | :---: | :---: | :---: | :---: |
| **Intégration ERP** | ✅ Temps réel | ❌ Manuel (MAJ 9h) | ❌ Aucune | ✅ ERP Maison |
| **App Mobile Serveurs** | ✅ Complète | ✅ Simplifiée | ❌ Absente | ✅ Complète+ |
| **Certification NF525** | ✅ Oui | ✅ Oui | ✅ Oui | ✅ Oui |
| **Notifications Cuisine** | ✅ Auto WebSocket | ❌ Cloche manuelle | ❌ Aucune | ✅ Auto WebSocket |
| **Conformité PCI DSS** | ✅ Oui | ✅ Oui | ✅ Oui | ✅ Oui |
| **Stocks Temps Réel** | ✅ Cache 30s | ❌ Statique jour | ❌ N/A | ✅ Natif |
| **Mode Offline** | ✅ SQLite sync | ✅ SQLite sync | ❌ N/A | ✅ SQLite sync |
| **Budget HT** | **43 700 €** | 32 000 € | 18 000 € | 58 000 € |
| **Délai** | **5 mois** | 3,5 mois | 2 mois | 7 mois |
| **Risque Technique** | 🟢 Faible* | 🟡 Moyen | 🟢 Faible | 🟡 Moyen |
| **Satisfaction Besoin** | **100%** | 60% | 20% | 105% |
| **Recommandation** | ✅ **RETENU** | ⚠️ Plan B | 🔴 Éviter | 🟡 Option ultime |

*Si conditions bloquantes levées (API ERP + Audit PCI DSS)*

---

## C.7. Décision Finale : Scénario A Retenu

### C.7.1. Justification du Choix

Le **Scénario A (Projet Complet)** est retenu car :

✅ **Répond intégralement au besoin initial** :
- Application mobile serveurs avec stocks temps réel
- Intégration bidirectionnelle avec l'ERP cuisine
- Caisse certifiée NF525 avec liaison TPE
- Notifications automatiques "plat prêt"

✅ **Offre la meilleure valeur ajoutée** :
- Digitalisation complète du service en salle
- Réduction des erreurs (stocks synchronisés)
- Expérience utilisateur optimale
- R ROI élevé (gain efficacité ×1.5, CA vins +15% attendu)

✅ **Est techniquement réalisable** :
- Technologies matures et éprouvées
- Architecture 3-tiers modulaire extensible
- Équipe de 4 personnes suffisante

**Scénarios alternatifs rejetés** :
- **Scénario B** : Perte valeur ajoutée temps réel (-40% fonctionnalités)
- **Scénario C** : Ne répond qu'à obligation NF525 (pas de digitalisation)
- **Scénario D** : Chef refuse changer ERP + Budget dépassé (+33%)

---

### C.7.2. Conditions de Réalisation (Pré-requis BLOQUANTS)

Le Scénario A est **conditionnellement réalisable** si et seulement si :

#### ✅ PRÉ-REQUIS 1 : Documentation API ERP Obtenue

**Exigence** : L'éditeur de "QuiCuisineIci" fournit :
- Documentation complète API (Swagger/OpenAPI ou équivalent)
- Environnement de test/sandbox avec credentials
- Engagement stabilité (pas de breaking change sans préavis 6 mois)

**Action** : Réunion tripartite (Client + Chef + Éditeur ERP) à organiser **AVANT signature contrat**

**Deadline** : Semaine 0 (avant démarrage projet)

**Plan B si refus** : Basculement Scénario B (mode simplifié) → Nécessite réapprobation client

---

#### ✅ PRÉ-REQUIS 2 : Infrastructure Réseau Conforme PCI DSS

**Exigence** : Audit réseau par un QSA (Qualified Security Assessor) validant :
- Segmentation VLAN (réseau monétique isolé VLAN 10)
- Chiffrement TLS 1.3 pour communications TPE
- Pas de stockage PAN/CVV dans le logiciel

**Action** : Audit à réaliser **Semaine 1** du projet (Phase 0)

**Deadline** : Avant IT1 (démarrage développement)

**Plan B si non-conformité** : Mise à niveau infrastructure (switch Ubiquiti, firewall PfSense) → Budget additionnel 5-12k€

---

#### ✅ PRÉ-REQUIS 3 : Engagement Processus Certification NF525

**Exigence** : Le client accepte :
- Délai certification : 3-6 mois post-développement
- Coût audit : ~3 500 € (inclus dans budget global)
- Phase de pré-audit avec expert NF525

**Action** : Validation formelle client + engagement consultant NF525 à J+30

**Plan B si refus** : Utilisation framework Open Source pré-certifié (OdooPOS) → Moins flexible

---

## C.8. Conclusion Faisabilité

### ✅ Verdict : Projet FAISABLE sous conditions

Le projet est **techniquement réalisable** avec les technologies actuelles,  **MAIS** comporte **deux risques bloquants** :

1. **Dépendance à l'ERP externe** (API non documentée = infaisable)
2. **Conformité réseau PCI DSS** (infrastructure actuelle inconnue)

**Recommandation finale** :
> Ne pas démarrer le développement avant d'avoir **levé ces deux incertitudes**. Un démarrage prématuré pourrait conduire à un projet **non livrable** ou **non conforme légalement**.

**Plan d'action proposé** :
1. **Phase 0 (2 semaines)** : Audit technique (API ERP + Réseau PCI DSS)
2. **Go / No-Go** : Décision de démarrage basée sur résultats audit
3. **Phase 1 (IT1)** : Si Go → Démarrer conception détaillée + développement

**Livrables Phase 0** :
- Rapport audit API ERP (documentation disponibilité)
- Rapport audit réseau PCI DSS (plan mise en conformité si nécessairesalle)
-Validation hypothèses techniques (stack, architecture)

**Budget Phase 0** : 4 500€ HT (audit QSA 2k€ + expert API 1,5k€ + PM 1k€)

---
