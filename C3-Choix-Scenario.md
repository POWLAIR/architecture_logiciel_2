# C3 - Choix du Scénario et Hypothèses de Travail

## Objectif de ce Document
Suite à l'analyse de faisabilité détaillée (`C3-Faisabilite.md`), ce document **valide formellement** le scénario retenu pour la conception et le développement, et **établit les hypothèses de travail** qui serviront de base pour les phases suivantes (C4 à C9).

---

## 1. Rappel des Contraintes Identifiées

### Contraintes BLOQUANTES (Priorité Critique)
| Contrainte | Source | Impact si Non Résolue |
| :--- | :--- | :--- |
| **Documentation API ERP "QuiCuisineIci"** | C3 §5.1 | Impossibilité d'intégration temps réel |
| **Conformité réseau PCI DSS** | C3 §3.1 | Risque juridique paiements CB |
| **Certification NF525** | C1, C3 §4.2 | Illégalité du logiciel de caisse |

### Contraintes IMPORTANTES (Priorité Haute)
| Contrainte | Source | Mitigation Possible |
| :--- | :--- | :--- |
| **Mode dégradé (perte Wifi)** | C2 §2.2 | Architecture Offline-First |
| **Fréquence MAJ carte** | C2 §3.1 | Clarification client requise |
| **Synchronisation multi-dispositifs** | C3 §3.2 | Transaction ACID + WebSocket |

---

## 2. Scénario Retenu : **SCÉNARIO A (Projet Complet)**

### 2.1. Justification du Choix

Après analyse des 4 scénarios alternatifs (voir `C3-Faisabilite.md` §8), le **Scénario A** est retenu car :

✅ **Il répond intégralement au besoin initial** :
- Application mobile serveurs avec stocks temps réel.
- Intégration bidirectionnelle avec l'ERP cuisine.
- Caisse certifiée NF525 avec liaison TPE.
- Notifications automatiques "plat prêt".

✅ **Il offre la meilleure valeur ajoutée** :
- Digitalisation complète du service en salle.
- Réduction des erreurs (stocks synchronisés).
- Expérience utilisateur optimale.

✅ **Il est techniquement réalisable** (sous réserve levée des blocages).

⚠️ **Scénarios B/C/D** : Rejetés car :
- **B** : Perte de la fonctionnalité temps réel (cœur de valeur).
- **C** : Ne répond qu'à l'obligation légale NF525 (pas de digitalisation).
- **D** : Hors scope initial (Chef refuse de changer d'ERP).

---

### 2.2. Conditions de Réalisation (Pré-requis)

Le Scénario A est **conditionnellement réalisable** si et seulement si :

#### ✅ PRÉ-REQUIS 1 : Documentation API ERP Obtenue
**Exigence** : L'éditeur de "QuiCuisineIci" fournit :
- Documentation complète API (Swagger/OpenAPI ou équivalent).
- Environnement de test/sandbox avec credentials.
- Engagement stabilité (pas de breaking change sans préavis 6 mois).

**Action** : Réunion tripartite (Client + Chef + Éditeur ERP) à organiser **AVANT signature contrat**.

**Plan B si refus** : Basculement Scénario B (mode simplifié) ou D (ERP maison) → Nécessite réapprobation client.

---

#### ✅ PRÉ-REQUIS 2 : Infrastructure Réseau Conforme PCI DSS
**Exigence** : Audit réseau par un QSA (Qualified Security Assessor) validant :
- Segmentation VLAN (réseau monétique isolé).
- Chiffrement TLS 1.2+ pour communications TPE.
- Pas de stockage PAN/CVV dans le logiciel.

**Action** : Audit à réaliser **Semaine 1** du projet (Phase 0).

**Plan B si non-conformité** : Mise à niveau infrastructure (switch, firewall) → Budget additionnel 5-12k€.

---

#### ✅ PRÉ-REQUIS 3 : Engagement Processus Certification NF525
**Exigence** : Le client accepte :
- Délai certification : 3-6 mois post-développement.
- Coût audit : ~3 500 € (inclus dans budget global).
- Phase de pré-audit avec expert NF525.

**Action** : Validation formelle client + engagement consultant NF525 à J+30.

**Plan B si refus** : Utilisation framework Open Source pré-certifié (OdooPOS) → Moins flexible.

---

## 3. Hypothèses de Travail pour la Suite du Projet

Les phases C4 à C9 se baseront sur les **hypothèses suivantes**, validées dans le cadre du Scénario A :

---

### 3.1. Périmètre Fonctionnel (Version 1)

#### ✅ Fonctionnalités INCLUSES
- [x] **Prise de commande mobile** (3 terminaux Android).
- [x] **Consultation stocks temps réel** (lecture API ERP).
- [x] **Suggestion accord mets-vins** (table statique éditable).
- [x] **Affichage allergènes** par plat (conformité INCO).
- [x] **Envoi commandes vers ERP** cuisine (REST API).
- [x] **Réception notifications** "plat prêt" (WebSocket/Push).
- [x] **Gestion plan de salle** (assignation tables/serveurs).
- [x] **Encaissement certifié NF525** (ISCA complet).
- [x] **Paiements divisés/groupés** (par article ou global).
- [x] **Liaison TPE** (protocole sécurisé PCI DSS).
- [x] **Mode offline de secours** (cache local + sync différée).

#### ❌ Fonctionnalités EXCLUES (V2)
- [ ] Gestion des réservations (web/mobile).
- [ ] Dashboard analytics propriétaire (reporting avancé).
- [ ] Gestion pourboires (répartition serveurs).
- [ ] Historique client / CRM.
- [ ] Module bar (hors scope - bar non actif).

---

### 3.2. Stack Technique (Présélection)

> **Note** : Le choix définitif sera validé en C5 après étude comparative.

#### Backend
**Présomption** : Node.js (Express/NestJS) ou Python (FastAPI/Django).
- Justification : Maturité, performances, compatibilité REST/WebSocket.

#### Frontend Mobile (Android)
**Présomption** : React Native ou Flutter.
- Justification : Cross-platform (si extension iOS future), temps de développement réduit.

#### Base de Données
**Présomption** : PostgreSQL.
- Justification : ACID (synchronisation multi-users), robustesse, Open Source.

#### Module NF525
**Présomption** : Framework dédié (ex: `js-caisse-nf525`) ou développement sur-mesure avec pré-audit.

#### Hébergement
**Présomption** : Cloud privé (OVH, Scaleway) en France (RGPD).
- Justification : Souveraineté données, conformité RGPD, coûts maîtrisés.

---

### 3.3. Architecture Cible Présumée

```
┌──────────────────────────────────────────────────────────┐
│                    SALLE (Frontend)                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────┐          ┌──────────────────┐      │
│  │ Mobile Serveur 1│          │  Caisse (Fixe)   │      │
│  │   (Android)     │          │   (Windows)      │      │
│  └────────┬────────┘          └────────┬─────────┘      │
│           │                            │                │
│  ┌────────┴────────┐          ┌────────┴─────────┐      │
│  │ Mobile Serveur 2│          │                  │      │
│  └────────┬────────┘          │                  │      │
│           │                   │                  │      │
│  ┌────────┴────────┐          │                  │      │
│  │ Mobile Serveur 3│          │                  │      │
│  └────────┬────────┘          │                  │      │
│           │                   │                  │      │
└───────────┼───────────────────┼──────────────────┘      
            │                   │                         
            │   ┌───────────────▼──────────────┐          
            │   │       Réseau Wifi            │          
            │   │  (3 Bornes - VLAN Métier)    │          
            │   └───────────────┬──────────────┘          
            │                   │                         
┌───────────▼───────────────────▼──────────────────────────┐
│                  BACKEND (API Centrale)                  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────┐  ┌────────────────┐  ┌───────────┐  │
│  │ API REST/WS    │  │ Logique Métier │  │ Auth/     │  │
│  │ (Express/Fast) │  │ (Services)     │  │ Security  │  │
│  └───────┬────────┘  └───────┬────────┘  └─────┬─────┘  │
│          │                   │                 │        │
│  ┌───────▼───────────────────▼─────────────────▼─────┐  │
│  │         Base de Données PostgreSQL               │  │
│  │  - Tables: Menu, Commandes, Stocks, Users...     │  │
│  │  - Module NF525 (Journal inaltérable)            │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
└─────────┬────────────────────────────────┬───────────────┘
          │                                │               
          │                                │               
┌─────────▼─────────┐          ┌───────────▼──────────────┐
│  ERP "QuiCuisine  │          │   TPE Bancaire           │
│       Ici"        │          │  (PCI DSS VLAN Monétique)│
│  (Cuisine)        │          │                          │
└───────────────────┘          └──────────────────────────┘
```

---

### 3.4. Volumétrie et Performances (Hypothèses)

| Métrique | Valeur Estimée | Base de Calcul |
| :--- | ---: | :--- |
| **Tables totales** | 20 | 12 tables 4p + 8 tables 6p |
| **Couverts/service** | ~80 | Taux occupation 80% × Capacité |
| **Services/jour** | 2 | Déjeuner + Dîner |
| **Commandes/jour** | ~180 | ~2,2 commandes/couvert (E+P+D) |
| **Transactions caisse/jour** | ~70 | ~1 transaction/table/service |
| **Requêtes API/heure (pointe)** | ~300 | 3 mobiles × 100 req/h |
| **Taille BDD (1 an)** | ~5 Go | 180 cmd/j × 365j × log verbose |

**Cibles de performance** :
- Temps de réponse API : < 200 ms (95e percentile).
- Temps chargement menu mobile : < 1 s.
- Latence notification "plat prêt" : < 3 s.
- Disponibilité : 99,5% (tolérance 1,8 jour/an).

---

### 3.5. Conformité et Sécurité (Exigences)

| Norme | Niveau | Validation |
| :--- | :--- | :--- |
| **NF525** | Certification AFNOR | Audit officiel J+90 |
| **RGPD** | Conformité totale | DPO + Registre CNIL |
| **PCI DSS** | SAQ A-EP | Audit QSA initial |
| **INCO** (Allergènes) | Obligatoire | Champs BDD + Affichage mobile |
| **Loi Evin** | Mention sanitaire | Validation UI/UX |
| **LCEN** (Logs Wifi) | 12 mois conservation | Serveur logs dédié |

---

## 4. Points de Vigilance Reportés aux Phases Suivantes

### 4.1. À Clarifier en C4 (Conception Architecture)
- [ ] Choix précis du pattern architectural (MVC, Hexagonal, Microservices ?).
- [ ] Stratégie de cache (Redis, memcached ?).
- [ ] Gestion des sessions (JWT, OAuth2 ?).

### 4.2. À Valider en C5 (Sélection Technologies)
- [ ] Framework mobile final (React Native vs Flutter vs Natif).
- [ ] ORM backend (Prisma, TypeORM, SQLAlchemy ?).
- [ ] Solution monitoring (Prometheus, Grafana, Sentry).

### 4.3. À Détailler en C7 (Modélisation)
- [ ] MCD complet (Entités, Relations, Cardinalités).
- [ ] Diagrammes de séquence (scénarios critiques).
- [ ] Diagramme de contexte (interactions ERP/TPE).

---

## 5. Validation Formelle et Gouvernance

### 5.1. Décision Projet
**Scénario retenu** : ✅ **Scénario A (Projet Complet)**  
**Budget prévisionnel** : 43 700 € HT  
**Délai prévisionnel** : 5 mois (incluant certification)  
**Date document** : 02/02/2026

### 5.2. Conditions Suspensives
Le démarrage effectif du développement est **suspendu** à la levée des 2 blocages critiques :
1. ✅ Obtention documentation API ERP (Délai : 2 semaines).
2. ✅ Validation conformité réseau PCI DSS (Délai : 1 semaine).

### 5.3. Points de Contrôle (Jalons)
| Jalon | Livrable | Date Cible | Validation |
| :--- | :--- | :--- | :--- |
| **J0** | Signature contrat + Pré-requis OK | - | Client + Prestataire |
| **J+30** | Dossier Architecture Technique (C4) | S5 | Comité projet |
| **J+45** | Spécifications Fonctionnelles (C5, C6) | S7 | Client |
| **J+60** | Pré-audit NF525 (blanc) | S9 | Expert NF525 |
| **J+90** | V1 développée (tests unitaires OK) | S13 | QA interne |
| **J+120** | Certification NF525 soumise | S17 | AFNOR/LNE |
| **J+150** | Mise en production | S22 | Client |

---

## 6. Récapitulatif pour les Phases Suivantes

### Ce qui est ACQUIS (Base de Travail)
✅ Périmètre fonctionnel V1 défini et figé.  
✅ Scénario A validé comme architecture cible.  
✅ Contraintes normatives identifiées et intégrées.  
✅ Parties prenantes cartographiées.  
✅ Risques majeurs analysés avec plans de mitigation.

### Ce qui RESTE à FAIRE (C4 → C9)
🔲 **C4** : Concevoir l'architecture logicielle détaillée.  
🔲 **C5** : Sélectionner les technologies précises (benchmark).  
🔲 **C6** : Valider l'architecture (critères de qualité).  
🔲 **C7** : Créer les diagrammes et modèles de données.  
🔲 **C8** : Intégrer la démarche TDD.  
🔲 **C9** : Finaliser le cahier des charges complet.

---

## 7. Signature Validation (Symbolique)

**Prestataire (Nous)** : ✅ Scénario A retenu après analyse de faisabilité exhaustive.  
**Client (Restaurant)** : ⏳ En attente validation (après levée pré-requis).  
**Éditeur ERP** : ⏳ En attente réponse (documentation API).  

**Prochaine action** : Organiser réunion de cadrage (Atelier J+3) pour valider formellement ce choix et débloquer les verrous critiques.

---

## Conclusion

Ce document acte le **choix du Scénario A** comme base de travail pour la suite du projet. Les hypothèses techniques, fonctionnelles et organisationnelles posées ici serviront de **référentiel** pour les phases de conception (C4), sélection technologique (C5) et modélisation (C7).

**⚠️ Point d'attention** : Le Scénario A reste **conditionnel** à la levée des blocages API ERP et PCI DSS. En cas d'impossibilité, un repli sur Scénario B (simplifié) ou D (ERP maison) devra être décidé en comité de projet.
