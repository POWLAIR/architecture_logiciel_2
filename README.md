# 🍽️ Projet Architecture Logicielle - Système Gestion Restaurant

> **Cahier des Charges Complet** pour un système de gestion de salle restaurant avec intégration ERP, conformité NF525 et mode offline.

---

## 📋 Table des Matières

- [Vue d'Ensemble](#-vue-densemble)
- [Structure du Projet](#-structure-du-projet)
- [Documents Sources](#-documents-sources)
- [Cahier des Charges Final](#-cahier-des-charges-final)
- [Utilisation](#-utilisation)
- [Métriques du Projet](#-métriques-du-projet)
- [Technologies Retenues](#-technologies-retenues)
- [Conformité et Normes](#-conformité-et-normes)

---

## 🎯 Vue d'Ensemble

### Contexte

Digitalisation complète de la gestion de salle d'un restaurant (180 couverts/jour, 20 tables) comprenant :

- 📱 **Application mobile** React Native pour 3 serveurs (prise commandes offline)
- 💻 **Application web** React.js pour 2 caissiers (split bill, NF525)
- 🔗 **Intégration ERP** "QuiCuisineIci" (REST API bidirectionnelle)
- 🔐 **Conformité NF525** (hash chaîné SHA-256, signature RSA)

### Objectifs

1. ✅ **Digitalisation complète** : Élimination tickets papier (12% erreurs actuelles)
2. ✅ **Split Bill automatique** : Division paiements par couvert (temps × 3 gagné)
3. ✅ **Recommandations vins** : Optimisation CA vins (+15% estimé)
4. ✅ **Mode offline résilient** : Continuité service si WiFi down
5. ✅ **Conformité légale** : NF525 + PCI DSS + RGPD

### Chiffres Clés

| Métrique | Valeur |
|:---------|:-------|
| **Budget Total** | 43 700€ HT |
| **Délai Projet** | 21 semaines (~5 mois) |
| **Fonctionnalités** | 166 UC (110 IT1-IT4 + 56 V2) |
| **Tests** | 2850 (pyramide 70/25/5) |
| **Score Conformité** | 99% (9/9 sections Cx) |
| **Score Fiabilité** | 100/100 |

---

## 📂 Structure du Projet

```
architecture_logiciel_2/
├── README.md                           # Ce fichier
│
├── Rendu/                              # 📦 CAHIER DES CHARGES FINAL
│   ├── README.md                       # Guide utilisation cahier des charges
│   ├── C9-Cahier-Charges-Complet.md    # Document maître (661 lignes)
│   ├── Annexe-A-Cas-Usage.md           # Cas d'usage détaillés
│   ├── Annexe-B-Analyse-Besoins-Parties-Prenantes.md  # (743 lignes)
│   ├── Annexe-C-Faisabilite-Scenarios.md              # (713 lignes)
│   ├── Annexe-D-Iterations-Architecture.md            # (1035 lignes)
│   ├── Annexe-E-Technologies-UML.md                   # (973 lignes)
│   ├── Annexe-F-Validation-Architecture.md            # Validation ISO 25010
│   ├── Annexe-G-Modelisation-Systeme.md               # (1113 lignes, 7 diagrammes Mermaid)
│   └── Annexe-H-Pratiques-TDD.md                      # (1029 lignes)
│
├── C1-cas_d'utilisation.md             # 13 UC détaillés
├── C1-Normes-conformité.md             # NF525, RGPD, PCI DSS
├── C1-Autres-Normes.md                 # ISO 27001, WCAG 2.1
├── C2-Synthese.md                      # Contexte, objectifs, périmètre
├── C2-Analyse-Besoins.md               # Problématiques détaillées
├── C2-Parties-Prenantes.md             # Profils acteurs (13 parties prenantes)
├── C3-Faisabilite.md                   # Risques, contraintes, planning
├── C3-Choix-Scenario.md                # Comparaison 4 scénarios (A/B/C/D)
├── C4-IT1-MVP-Fonctionnel.md           # Itération 1 (8 semaines, 39 UC)
├── C4-IT2-Securite-Conformite.md       # Itération 2 (6 semaines, 43 UC)
├── C4-IT3-Performance-Resilience.md    # Itération 3 (4 semaines, 13 UC)
├── C4-IT4-Scalabilite-Observabilite.md # Itération 4 (3 semaines, 15 UC)
├── C5-Selection-Technologies.md        # Matrices comparatives
├── C5-Liste-Fonctionnalites.md         # 166 fonctionnalités détaillées
├── C5-Diagrammes-UML.md                # 11 diagrammes UC (Mermaid)
├── C5-Diagramme-UML-Global.md          # Vue globale architecture UML
├── C6-Validation-Architecture.md       # Architecture 3-tiers modulaire
├── C7-Diagrammes-Sequences.md          # 5 DS majeurs (Mermaid)
├── C7-MCD.md                           # ERD 8 entités PostgreSQL
├── C7-Interactions-Environnement.md    # Diagramme contexte C4
├── C8-Pratiques-TDD.md                 # Pyramide tests, couverture 90%
│
└── architecture-logicielle-Consigne.md # Consigne projet (référentiel)
```

**Total** : **~14 500 lignes** de documentation technique

---

## 📚 Documents Sources

### Phase 1 : Analyse et Conception (C1-C4)

| Document | Contenu | Lignes | Statut |
|:---------|:--------|:------:|:------:|
| **C1** | Cas d'usage + Normes (NF525, RGPD, PCI DSS, OWASP) | ~800 | ✅ |
| **C2** | Synthèse + Besoins + Parties prenantes (13 acteurs) | ~900 | ✅ |
| **C3** | Faisabilité + Scénarios (A/B/C/D) + Choix justifié | ~650 | ✅ |
| **C4** | 4 Itérations (IT1-IT4 : MVP → Sécurité → Résilience → Observabilité) | ~1200 | ✅ |

### Phase 2 : Technologies et UML (C5)

| Document | Contenu | Lignes | Statut |
|:---------|:--------|:------:|:------:|
| **C5** | Stack technique + 166 fonctionnalités + 11 diagrammes UML | ~1100 | ✅ |

### Phase 3 : Validation et Modélisation (C6-C7)

| Document | Contenu | Lignes | Statut |
|:---------|:--------|:------:|:------:|
| **C6** | Validation architecture 3-tiers + Dilemmes justifiés | ~450 | ✅ |
| **C7** | 5 DS + ERD 8 entités + Interactions environnement | ~1300 | ✅ |

### Phase 4 : Tests (C8)

| Document | Contenu | Lignes | Statut |
|:---------|:--------|:------:|:------:|
| **C8** | TDD + Pyramide 70/25/5 + Couverture 90% | ~600 | ✅ |

---

## 📦 Cahier des Charges Final

### Localisation

Le cahier des charges consolidé et finalisé se trouve dans le dossier **[`Rendu/`](./Rendu/)**.

### Composition

- **1 document maître** : [`C9-Cahier-Charges-Complet.md`](./Rendu/C9-Cahier-Charges-Complet.md)
- **8 annexes** : A (Cas d'usage) à H (TDD)
- **Total** : **7392 lignes** + **19 diagrammes Mermaid**

### Score Qualité

| Critère | Score | Statut |
|:--------|:-----:|:------:|
| **Conformité Cx (C1-C9)** | 99% | ✅ Conforme |
| **Cohérence interne** | 100/100 | ✅ Parfait |
| **Traçabilité Besoins→Tests** | 95% | ✅ Excellent |
| **Couverture diagrammes** | 19 Mermaid | ✅ Complet |

**Voir** : [`Rendu/README.md`](./Rendu/README.md) pour guide d'utilisation détaillé

---

## 🚀 Utilisation

### Pour Validation Client

```bash
# Lire sections contexte et exigences
cat Rendu/C9-Cahier-Charges-Complet.md | head -300

# Vérifier budget et délais
grep -E "43 700|21 sem" Rendu/C9-Cahier-Charges-Complet.md
```

**Documents à présenter** :
1. C9 § 1-3 (Introduction, Exigences fonctionnelles/non-fonctionnelles)
2. Annexe-C (Faisabilité, Scénario A retenu)
3. Annexe-D (Planning 4 itérations, 21 semaines)

### Pour Équipe Développement

```bash
# Architecture technique
cat Rendu/Annexe-D-Iterations-Architecture.md  # IT1-IT4 détaillées
cat Rendu/Annexe-E-Technologies-UML.md         # Stack + 166 fonctionnalités

# Modélisation système
cat Rendu/Annexe-G-Modelisation-Systeme.md     # 7 diagrammes (DS1-5, ERD, Interactions)
```

### Pour Équipe QA

```bash
# Stratégie TDD
cat Rendu/Annexe-H-Pratiques-TDD.md  # Pyramide 2850 tests (70/25/5)
```

### Pour Chef de Projet

```bash
# Risques et contraintes
cat Rendu/Annexe-C-Faisabilite-Scenarios.md  # Phase 0 pré-requis
cat Rendu/Annexe-B-Analyse-Besoins-Parties-Prenantes.md  # 13 parties prenantes
```

---

## 📊 Métriques du Projet

### Documentation

| Métrique | Valeur |
|:---------|:------:|
| **Documents sources** | 21 fichiers |
| **Documents finaux** | 9 fichiers (C9 + 8 annexes) |
| **Lignes totales** | ~14 500 lignes |
| **Diagrammes Mermaid** | 19 (7 DS + 11 UC + 1 ERD) |
| **Tableaux** | 150+ tableaux comparatifs |

### Fonctionnalités

| Itération | UC | Délai | Budget |
|:----------|:--:|:-----:|:------:|
| **IT1 - MVP** | 39 | 8 sem | 18 000€ |
| **IT2 - Sécurité** | 43 | 6 sem | 12 000€ |
| **IT3 - Résilience** | 13 | 4 sem | 8 700€ |
| **IT4 - Observabilité** | 15 | 3 sem | 5 000€ |
| **Sous-total** | **110** | **21 sem** | **43 700€** |
| **V2 - Extensions** | 56 | - | - |
| **TOTAL** | **166** | - | - |

### Tests

| Type | Nombre | % |
|:-----|:------:|:-:|
| **Unitaires** | 2000 | 70% |
| **Intégration** | 700 | 25% |
| **E2E** | 150 | 5% |
| **TOTAL** | **2850** | **100%** |

**Couverture cible** : **90%** (code coverage)

---

## 💻 Technologies Retenues

### Frontend

| Couche | Technologie | Version | Justification |
|:-------|:------------|:-------:|:--------------|
| **Mobile** | React Native | 0.73 | Cross-platform (Android prioritaire, iOS V2) |
| **Web** | React.js | 18 | Écosystème mature, composants réutilisables |
| **État** | Redux Toolkit | 1.9 | Prévisibilité, DevTools, middleware |

### Backend

| Couche | Technologie | Version | Justification |
|:-------|:------------|:-------:|:--------------|
| **Runtime** | Node.js | 20 LTS | Non-blocking I/O, écosystème NPM |
| **Framework** | Express.js | 4.18 | Léger, flexible, middleware riche |
| **ORM** | Prisma | 5.8 | Type-safe, migrations, génération TS |
| **Base de données** | PostgreSQL | 15 | ACID, robustesse, JSONB, triggers |
| **Cache** | Redis | 7.2 | In-memory, Pub/Sub, TTL |
| **WebSocket** | Socket.io | 4.6 | Temps réel, fallback auto, rooms |

### Infrastructure

| Couche | Technologie | Justification |
|:-------|:------------|:--------------|
| **Reverse Proxy** | NGINX | Performance C10k, SSL termination |
| **Monitoring** | Prometheus + Grafana | Métriques temps réel, alertes Slack |
| **Logs** | ELK Stack | Centralisation, recherche, rétention 90j |
| **CI/CD** | GitHub Actions | Automatisation tests, déploiement |

**Score Stack Moyen** : **8.9/10** ⭐⭐⭐⭐

---

## 🔐 Conformité et Normes

### Normes Appliquées

| Norme | Domaine | Implémentation | Statut |
|:------|:--------|:---------------|:------:|
| **NF525** | Fiscalité anti-fraude | Hash chaîné SHA-256, signature RSA, archivage 6 ans | ✅ IT2 |
| **PCI DSS** | Sécurité paiements | VLAN 10 isolé, pas stockage PAN, TLS 1.3 | ✅ IT2 |
| **RGPD** | Protection données | Consentement, export/suppression, durée conservation | ✅ IT2 |
| **OWASP Top 10** | Sécurité web | JWT, HTTPS, input validation, rate limiting | ✅ IT2 |
| **ISO 25010** | Qualité logicielle | 25 critères évalués (score 100% IT4) | ✅ IT4 |
| **WCAG 2.1** | Accessibilité | Contrastes AA (partiel IT1, complet V2) | ⏳ V2 |

### Certifications Prévues

| Certification | Organisme | Délai | Coût |
|:--------------|:----------|:-----:|:----:|
| **NF525** | AFNOR/LNE | 3-6 mois post-dev | 3 500€ |
| **PCI DSS** | QSA | 2-4 mois | 2 000€ (audit réseau) |

**Audit IT2** : Prévoir POC NF525 semaine 1 pour validation crypto

---

## 📖 Guides Complémentaires

- **[Rendu/README.md](./Rendu/README.md)** : Guide utilisation cahier des charges final
- **[architecture-logicielle-Consigne.md](./architecture-logicielle-Consigne.md)** : Consigne projet (référentiel)
- **[Guide-Compilation-Cahier-Charges.md](./Guide-Compilation-Cahier-Charges.md)** : Processus compilation (si existe)

---

## 🤝 Contribution

### Mise à Jour Documentation

```bash
# Modifier documents sources C1-C8 (si évolution besoins)
vim C2-Analyse-Besoins.md

# Régénérer annexes consolidées dans Rendu/
# (processus manuel : copier sections pertinentes dans Annexe-B)

# Vérifier cohérence
grep "43 700" Rendu/*.md  # Budget
grep "21 sem" Rendu/*.md  # Délai
```

### Validation Modifications

1. Vérifier score conformité (99%+)
2. Vérifier score cohérence (95/100+)
3. Mettre à jour `Rendu/C9-Cahier-Charges-Complet.md` (références)

---

## 📄 Licence

**Propriétaire** : EFREI - Projet Architecture Logicielle 2

**Usage** : Éducatif uniquement

---

## 📞 Contact

**Équipe Projet** : Architecture Logicielle - Promotion 2026  
**Contexte** : Projet académique EFREI

---

**Date Dernière Mise à Jour** : 2026-02-03  
**Version** : 1.0 Final - Cahier des Charges Complet et Validé
