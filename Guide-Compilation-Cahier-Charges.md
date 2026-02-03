# Guide de Compilation - Cahier des Charges Complet

## Objectif
Fournir un **guide précis** pour compiler le cahier des charges final en intégrant les sections pertinentes des documents C1-C8 comme annexes.

---

## Structure Finale Recommandée

```
📄 CAHIER DES CHARGES COMPLET.pdf (80-100 pages)
├── PARTIE 1 : DOCUMENT PRINCIPAL (30 pages)
│   └── C9-Cahier-Charges-Complet.md
│
└── PARTIE 2 : ANNEXES DÉTAILLÉES (50-70 pages)
    ├── Annexe A : Cas d'Usage Détaillés
    ├── Annexe B : Analyse Besoins et Parties Prenantes
    ├── Annexe C : Faisabilité et Scénarios
    ├── Annexe D : Itérations IT1-IT4 Détaillées
    ├── Annexe E : Fonctionnalités et Technologies
    ├── Annexe F : Diagrammes UML Complets
    ├── Annexe G : Modélisation Séquences et Données
    └── Annexe H : Pratiques TDD Détaillées
```

---

## PARTIE 1 : Document Principal

**Fichier source** : [C9-Cahier-Charges-Complet.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C9-Cahier-Charges-Complet.md)

**Action** : ✅ Utiliser tel quel (déjà complet)

**Contenu** : 9 sections avec tableaux synthétiques et références vers annexes

---

## PARTIE 2 : Annexes Détaillées

### Annexe A : Cas d'Usage Détaillés

**Fichier source** : [C1-cas_d'utilisation.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C1-cas_d'utilisation.md)

**Sections à intégrer** :

| Section Document Source | Lignes | Contenu à Intégrer | Titre Annexe |
|:------------------------|:------:|:-------------------|:-------------|
| **Toutes les sections** | 1-fin | 13 UC complets (flots nominaux, alternatifs, exceptions) | **A.1 à A.13** (1 UC par sous-section) |

**Format** :
```markdown
## Annexe A : Cas d'Usage Détaillés

### A.1. UC1 - Consulter Menu + Stocks Temps Réel

**Acteur** : Serveur (Mobile React Native)

**Préconditions** : 
- Serveur authentifié
- Application mobile lancée

**Flot Nominal** :
1. Serveur ouvre l'application mobile
2. Système affiche menu avec disponibilité temps réel (cache Redis 30s)
3. Serveur sélectionne catégorie (Entrées/Plats/Desserts/Vins)
...

**Flot Alternatif 1** : Stock épuisé
...

**Postconditions** :
...

[Répéter pour UC2 à UC13]
```

---

### Annexe B : Analyse Besoins et Parties Prenantes

**Fichiers sources** : 
- [C2-Analyse-Besoins.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C2-Analyse-Besoins.md)
- [C2-Parties-Prenantes.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C2-Parties-Prenantes.md)

**Sections à intégrer** :

#### De C2-Analyse-Besoins.md

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| Problématiques actuelles | ~20-50 | Tickets papier, erreurs, split bill manuel, non-conformité | **B.1** |
| Besoins fonctionnels | ~50-100 | Liste détaillée par acteur | **B.2** |
| Besoins non-fonctionnels | ~30-60 | Performance, sécurité, scalabilité détaillés | **B.3** |

#### De C2-Parties-Prenantes.md

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| Profils acteurs internes | ~40-80 | Serveurs, caissiers, admin, gérant (motivations, contraintes) | **B.4** |
| Profils systèmes externes | ~30-60 | ERP, TPE, API Stocks, Prometheus, ELK | **B.5** |
| Matrice influence/intérêt | ~10-20 | Tableau RACI + priorisation | **B.6** |

---

### Annexe C : Faisabilité et Scénarios

**Fichiers sources** :
- [C3-Faisabilite.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C3-Faisabilite.md)
- [C3-Choix-Scenario.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C3-Choix-Scenario.md)

**Sections à intégrer** :

#### De C3-Faisabilite.md

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| Analyse risques complète | ~100-150 | 15-20 risques (probabilité, impact, mitigation) | **C.1** |
| Planning détaillé | ~50-80 | Gantt IT1-IT4, jalons, dépendances | **C.2** |
| Budget détaillé | ~40-60 | Ventilation coûts (dev, infra, licences, audit) | **C.3** |
| Contraintes techniques | ~30-50 | Matériel, réseau, compatibilité | **C.4** |

#### De C3-Choix-Scenario.md

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| Scénario A (retenu) | ~50-80 | Intégration ERP REST détaillée | **C.5** |
| Scénario B (alternatif) | ~40-60 | Tickets papier cuisine (plan B) | **C.6** |
| Scénario C (écarté) | ~30-50 | API tierce externe (rejeté pourquoi) | **C.7** |
| Matrice décision | ~20-30 | Tableau comparatif 10 critères pondérés | **C.8** |

---

### Annexe D : Itérations IT1-IT4 Détaillées

**Fichiers sources** :
- [C4-IT1-MVP-Fonctionnel.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C4-IT1-MVP-Fonctionnel.md)
- [C4-IT2-Securite-Conformite.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C4-IT2-Securite-Conformite.md)
- [C4-IT3-Performance-Resilience.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C4-IT3-Performance-Resilience.md)
- [C4-IT4-Scalabilite-Observabilite.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C4-IT4-Scalabilite-Observabilite.md)

**Sections à intégrer** :

#### D.1. Itération IT1 (C4-IT1-MVP-Fonctionnel.md)

| Section | Lignes Estimées | Contenu | Points Clés |
|:--------|:---------------:|:--------|:------------|
| Objectifs IT1 | ~30-50 | Features MVP (commandes, paiements, vins) | Fonctionnalités P0 |
| Architecture IT1 | ~80-120 | Diagrammes C4 L1/L2 détaillés | Backend + Mobile + Web |
| Livrables IT1 | ~40-60 | Liste modules développés | `orders`, `payments`, `menu`, `wines` |
| Tests IT1 | ~30-50 | Stratégie tests MVP | 1200 tests (70/25/5) |
| Acceptance Criteria | ~20-30 | Critères validation IT1 | DoD avant IT2 |

#### D.2. Itération IT2 (C4-IT2-Securite-Conformite.md)

| Section | Lignes Estimées | Contenu | Points Clés |
|:--------|:---------------:|:--------|:------------|
| Objectifs IT2 | ~30-50 | Sécurité JWT/RBAC + NF525 | Conformité légale |
| Implémentation NF525 | ~100-150 | Hash chaîné, signature RSA, triggers PostgreSQL | Code crypto détaillé |
| Sécurité réseau | ~60-80 | VLAN TPE, HTTPS TLS 1.3, firewall | Architecture réseau |
| Tests IT2 | ~40-60 | Tests crypto + triggers | 500 tests NF525 |
| Audit PCI DSS | ~30-40 | Préparation audit réseau | Checklist compliance |

#### D.3. Itération IT3 (C4-IT3-Performance-Resilience.md)

| Section | Lignes Estimées | Contenu | Points Clés |
|:--------|:---------------:|:--------|:------------|
| Objectifs IT3 | ~30-50 | Offline mobile + Circuit Breaker | Résilience |
| Mode offline SQLite | ~80-120 | Architecture sync, queue, conflits | Schémas SQLite |
| Circuit Breaker Opossum | ~50-70 | Config states CLOSED/OPEN/HALF-OPEN | Code Opossum |
| Tests IT3 | ~40-60 | Tests offline + CB | 450 tests résilience |
| Métriques performance | ~30-40 | Latence P95, sync <5s | Objectifs chiffrés |

#### D.4. Itération IT4 (C4-IT4-Scalabilite-Observabilite.md)

| Section | Lignes Estimées | Contenu | Points Clés |
|:--------|:---------------:|:--------|:------------|
| Objectifs IT4 | ~30-50 | PM2 clustering, Grafana, logs ELK | Observabilité |
| PM2 Clustering | ~50-70 | Config 4 instances, load balancing | Horizontal scaling |
| Monitoring Prometheus | ~60-80 | Métriques custom, alertes | Dashboards Grafana |
| Logs centralisés ELK | ~50-70 | Logstash pipelines, Kibana dashboards | Aggregation logs |
| Tests IT4 | ~30-40 | Tests charge Artillery | 100 req/s |

---

### Annexe E : Fonctionnalités et Technologies

**Fichiers sources** :
- [C5-Liste-Fonctionnalites.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C5-Liste-Fonctionnalites.md)
- [C5-Selection-Technologies.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C5-Selection-Technologies.md)

**Sections à intégrer** :

#### E.1. Liste 166 Fonctionnalités (C5-Liste-Fonctionnalites.md)

| Domaine | Lignes Estimées | Nb Fonctionnalités | Titre Sous-Annexe |
|:--------|:---------------:|:------------------:|:------------------|
| Gestion Tables | ~30-50 | 15 | **E.1.1** |
| Menu + Vins | ~40-60 | 17 | **E.1.2** |
| Commandes | ~35-55 | 16 | **E.1.3** |
| Paiements + Split Bill | ~35-55 | 15 | **E.1.4** |
| Conformité NF525 | ~25-40 | 11 | **E.1.5** |
| Auth + RBAC | ~30-45 | 13 | **E.1.6** |
| Mode Offline | ~25-35 | 10 | **E.1.7** |
| Notifications WebSocket | ~15-25 | 6 | **E.1.8** |
| Rapports & Stats | ~25-40 | 12 | **E.1.9** |
| Intégration ERP | ~25-35 | 10 | **E.1.10** |
| Administration Système | ~25-40 | 11 | **E.1.11** |
| Autres modules | ~60-80 | 30 | **E.1.12** |

**Format** :
```markdown
### E.1.1. Gestion Tables (15 fonctionnalités)

**F001** : Consulter plan de salle
- **Description** : Afficher plan avec 20 tables, statut temps réel
- **Acteur** : Serveur
- **Itération** : IT1
- **Priorité** : P0
- **Critères acceptation** : 
  - Plan charge <1s
  - Statuts synchronisés <500ms
  - Légende couleurs (libre/occupée/réservée)

[Répéter F002 à F015]
```

#### E.2. Matrices Comparatives Technologies (C5-Selection-Technologies.md)

| Composant | Lignes Estimées | Contenu | Titre Sous-Annexe |
|:----------|:---------------:|:--------|:------------------|
| Backend (Node.js vs Java vs Python) | ~50-70 | Matrice 8 critères + justification | **E.2.1** |
| Framework (Express vs Fastify vs NestJS) | ~40-60 | Matrice 6 critères | **E.2.2** |
| Base données (PostgreSQL vs MySQL vs MongoDB) | ~60-80 | Matrice 10 critères ACID | **E.2.3** |
| Cache (Redis vs Memcached) | ~30-40 | Matrice 5 critères | **E.2.4** |
| Mobile (React Native vs Flutter vs Ionic) | ~50-70 | Matrice 7 critères cross-platform | **E.2.5** |
| Tests (Jest vs Mocha vs Vitest) | ~30-40 | Matrice 5 critères | **E.2.6** |

---

### Annexe F : Diagrammes UML Complets

**Fichiers sources** :
- [C5-Diagrammes-UML.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C5-Diagrammes-UML.md)
- [C5-Diagramme-UML-Global.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C5-Diagramme-UML-Global.md)

**Sections à intégrer** :

| Diagramme | Fichier Source | Lignes Estimées | Titre Annexe |
|:----------|:---------------|:---------------:|:-------------|
| Diagramme classes global | C5-Diagramme-UML-Global.md | ~100-150 | **F.1** |
| Diagramme classes module Orders | C5-Diagrammes-UML.md | ~80-120 | **F.2** |
| Diagramme classes module Payments | C5-Diagrammes-UML.md | ~70-100 | **F.3** |
| Diagramme séquence UC2 (Commande + Vin) | C5-Diagrammes-UML.md | ~60-80 | **F.4** |
| Diagramme activité Split Bill | C5-Diagrammes-UML.md | ~50-70 | **F.5** |
| Diagramme états Table | C5-Diagrammes-UML.md | ~40-60 | **F.6** |

**Action** : Intégrer **Mermaid complets en grand format** (pas versions réduites)

---

### Annexe G : Modélisation Séquences et Données

**Fichiers sources** :
- [C7-Diagrammes-Sequences.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C7-Diagrammes-Sequences.md)
- [C7-MCD.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C7-MCD.md)
- [C7-Interactions-Environnement.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C7-Interactions-Environnement.md)

**Sections à intégrer** :

#### G.1. Diagrammes Séquence (C7-Diagrammes-Sequences.md)

| Séquence | Lignes Estimées | Contenu | Titre Annexe |
|:---------|:---------------:|:--------|:-------------|
| Séq. 1 : Commande + Vin | ~120-160 | Mermaid complet + métriques (12 acteurs) | **G.1.1** |
| Séq. 2 : Split Bill ACID | ~100-140 | Mermaid + transactions PostgreSQL | **G.1.2** |
| Séq. 3 : Offline Sync | ~110-150 | Mermaid + détection conflits | **G.1.3** |
| Séq. 4 : Clôture NF525 | ~100-130 | Mermaid + crypto SHA-256/RSA | **G.1.4** |
| Séq. 5 : Notification "Plat Prêt" | ~80-100 | Mermaid + WebSocket flow | **G.1.5** |

#### G.2. Modèle Conceptuel Données (C7-MCD.md)

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| ERD complet 8 entités | ~50-70 | Mermaid grand format + cardinalités | **G.2.1** |
| Descriptions entités détaillées | ~150-200 | USERS, TABLES, ORDERS, ORDER_ITEMS, MENU_ITEMS, WINES, PAYMENTS, AUDIT_LOGS | **G.2.2** |
| Contraintes intégrité | ~80-100 | UK, FK, CHECK, triggers | **G.2.3** |
| Volumétrie détaillée | ~40-60 | Estimations 5 ans (~25 Go) | **G.2.4** |
| Index recommandés | ~30-50 | B-tree, GIN, partitioning | **G.2.5** |
| Triggers PL/pgSQL | ~60-80 | Code complet triggers NF525 | **G.2.6** |

#### G.3. Interactions Environnement (C7-Interactions-Environnement.md)

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| Diagramme C4 Contexte | ~50-70 | Mermaid grand format 8 interactions | **G.3.1** |
| Descriptions acteurs | ~80-100 | 3 humains + 5 systèmes détaillés | **G.3.2** |
| Tableaux protocoles | ~40-60 | REST/WebSocket/VLAN/Scraping | **G.3.3** |
| Patterns architecturaux | ~60-80 | 6 patterns (Repository, CB, Pub/Sub...) | **G.3.4** |
| Prérequis intégrations | ~40-60 | ERP/TPE/Infra checklists | **G.3.5** |
| Scénarios échec | ~50-70 | Pannes ERP/TPE/Stocks + mitigation | **G.3.6** |

---

### Annexe H : Pratiques TDD Détaillées

**Fichier source** : [C8-Pratiques-TDD.md](file:///wsl.localhost/Ubuntu/home/paul/efrei-project/architecture_logiciel_2/C8-Pratiques-TDD.md)

**Sections à intégrer** :

| Section | Lignes Estimées | Contenu | Titre Annexe |
|:--------|:---------------:|:--------|:-------------|
| Cycle Red-Green-Refactor | ~40-60 | Diagramme + workflow détaillé | **H.1** |
| Pyramide tests détaillée | ~80-100 | 70/25/5 avec exemples par niveau | **H.2** |
| Stratégie NF525 (68 tests) | ~60-80 | Tests crypto + triggers + E2E | **H.3** |
| Stratégie Offline (60 tests) | ~50-70 | Tests SQLite + sync + conflits | **H.4** |
| Configuration Jest complète | ~40-60 | `jest.config.js` + scripts NPM | **H.5** |
| Pipeline CI/CD GitLab | ~60-80 | YAML complet + gates qualité | **H.6** |
| Métriques couverture modules | ~30-40 | Tableau ≥95% critiques | **H.7** |
| Démarche opérationnelle | ~50-70 | Workflow dev + organisation équipe | **H.8** |
| Limitations TDD | ~40-50 | Ce que TDD ne couvre pas + ROI | **H.9** |

---

## Récapitulatif Volumétrie

| Partie | Pages Estimées | Contenu |
|:-------|:--------------:|:--------|
| **Document Principal C9** | **25-30** | Synthèse exécutive |
| **Annexe A** (UC) | 8-12 | 13 UC détaillés |
| **Annexe B** (Besoins/Parties) | 6-10 | Analyse + profils |
| **Annexe C** (Faisabilité) | 10-15 | Risques + budget + scénarios |
| **Annexe D** (IT1-IT4) | 15-20 | 4 itérations détaillées |
| **Annexe E** (Fonc/Techno) | 12-18 | 166 fonctionnalités + matrices |
| **Annexe F** (UML) | 6-10 | Diagrammes classes/séquences |
| **Annexe G** (Modélisation) | 12-18 | Séquences + MCD + environnement |
| **Annexe H** (TDD) | 8-12 | Pratiques tests détaillées |
| **TOTAL** | **80-120** | Document complet autonome |

---

## Instructions de Compilation

### Étape 1 : Préparer le Document Principal

**Fichier** : `C9-Cahier-Charges-Complet.md`

**Action** : ✅ Déjà prêt, utiliser tel quel

### Étape 2 : Créer les Annexes

Pour chaque annexe A à H :

1. **Créer fichier** : `Annexe-X-Titre.md`
2. **Format en-tête** :
   ```markdown
   # Annexe X : [Titre]
   
   *Ce document est un complément détaillé du Cahier des Charges Principal (C9)*
   
   ---
   ```

3. **Copier sections** depuis documents sources selon tableaux ci-dessus
4. **Numéroter sections** : X.1, X.2, X.3...
5. **Conserver Mermaid** : Diagrammes en grand format
6. **Ajouter index** : Table matières début annexe si >10 pages

### Étape 3 : Assembler le Document Final

**Outils recommandés** :

| Outil | Usage | Commande |
|:------|:------|:---------|
| **Pandoc** | Markdown → PDF | `pandoc C9-*.md Annexe-*.md -o CahierCharges.pdf --toc` |
| **Mermaid CLI** | Diagrammes → PNG | `mmdc -i diagram.mmd -o diagram.png` |
| **Markdown PDF (VSCode)** | Export simple | Ctrl+Shift+P → "Markdown PDF: Export" |

**Ordre assemblage** :
```bash
# Page de garde
# Table des matières (auto-générée)
# Document Principal C9 (sections 1-9)
# Annexe A : UC
# Annexe B : Besoins/Parties
# Annexe C : Faisabilité
# Annexe D : IT1-IT4
# Annexe E : Fonctionnalités/Technologies
# Annexe F : UML
# Annexe G : Modélisation
# Annexe H : TDD
# Index (optionnel)
```

### Étape 4 : Validation Finale

**Checklist** :
- [ ] Document principal C9 complet (9 sections)
- [ ] 8 annexes A-H créées
- [ ] Tous les diagrammes Mermaid rendus
- [ ] Numérotation cohérente (sections, pages, figures)
- [ ] Table des matières générée
- [ ] Liens internes fonctionnels (si PDF interactif)
- [ ] Taille PDF <50 Mo (compression images si besoin)
- [ ] Validation orthographe/grammaire
- [ ] Revue technique complétude

---

## Alternative : Approche Simplifiée

Si vous préférez **document autonome sans fichiers séparés** :

**Option** : Intégrer directement dans C9

**Action** : Pour chaque section C9, ajouter sous-section "Détails Complets" avec contenu annexe

**Exemple** :
```markdown
## 2. Exigences Fonctionnelles

### 2.1. Cas d'Usage Détaillés

[Tableau synthétique actuel]

#### 2.1.1. Détails UC1 - Consulter Menu

[Copier contenu complet C1 UC1 lignes X-Y]

#### 2.1.2. Détails UC2 - Prendre Commande

[Copier contenu complet C1 UC2 lignes X-Y]

...
```

**Inconvénient** : Document très lourd (150+ pages), difficile à naviguer

**Avantage** : 1 seul fichier, pas de gestion annexes

---

## Recommandation Finale

✅ **Privilégier approche Annexes** pour :
- Document principal concis (décideurs)
- Annexes détaillées (équipe technique)
- Maintenance facilitée (mise à jour sections séparées)
- Navigation claire (table matières + liens)

**Format livraison** : 
- 📄 **PDF compilé** (Pandoc) : Cahier-Charges-Complet.pdf (80-100 pages)
- 📂 **Sources Markdown** (optionnel) : C9-Principal.md + 8 annexes MD

---

## Prochaines Étapes

1. **Décider approche** : Annexes séparées ✅ (recommandé) ou Intégration directe
2. **Créer annexes** : 8 fichiers Annexe-A.md à Annexe-H.md
3. **Compiler PDF** : Pandoc avec template professionnel
4. **Validation finale** : Revue équipe + sponsor
