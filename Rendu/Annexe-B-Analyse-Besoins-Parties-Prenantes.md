# Annexe B : Analyse Besoins et Parties Prenantes

*Ce document est un complément détaillé du Cahier des Charges Principal*

---

## Introduction

Cette annexe consolide :
1. **L'analyse critique du besoin client** (contradictions, oublis, ambiguïtés identifiés)
2. **Les profils détaillés des parties prenantes** (acteurs internes et externes, motivations, influence)

---

## B.1. Analyse Critique du Besoin Client

### B.1.1. Objectif de l'Analyse

En tant que prestataire chargé d'analyser le besoin et de proposer une solution logicielle, il est crucial de procéder à une **reprise critique** de la demande initiale. Cette démarche permet d'identifier :
*   Les **contradictions** dans les exigences
*   Les **oublis** ou **manques évidents**
*   Les **zones d'ambiguïté** nécessitant des clarifications

---

## B.2. CONTRADICTIONS Identifiées

### B.2.1. Titre vs Périmètre Réel

**Énoncé** : *« Gestion d'un restaurant, salle et cuisine »*

**Réalité** : Le Chef Cuisinier refuse que le projet touche à la cuisine (ERP "QuiCuisineIci" déjà en place).

**Contradiction** :  
Le titre suggère une gestion intégrée **salle + cuisine**, alors que le périmètre réel se limite à la **salle uniquement** (prise de commande, encaissement).

**Recommandation** :  
Renommer le projet : *« Gestion de la Salle et Interface ERP Cuisine »* pour clarifier le scope.

---

### B.2.2. Bar "Non Actif" mais TPE Présent

**Énoncé** :  
> *« Le restaurant dispose d'un bar non actif à ce jour. À cet emplacement, il existe un unique terminal de paiement placé au niveau de la caisse enregistreuse. »*

**Contradiction** :  
*   Le bar est dit "non actif", pourtant un TPE y est installé
*   Aucune mention de la gestion des **apéritifs** (pourtant cités comme possibles) ni de leur lieu de préparation (bar fermé ? cuisine ?)

**Ambiguïté** :  
*   Qui prépare les apéritifs si le bar est fermé ?
*   Faut-il prévoir un module "Réactivation du Bar" dans la roadmap ?

**Recommandation** :  
Clarifier avec le client :  
→ *Les apéritifs sont-ils servis depuis la cuisine ou le bar est-il partiellement actif ?*  
→ *Le TPE est-il déjà relié à un système de caisse fonctionnel ?*

---

### B.2.3. Paiement Individuel vs Partage Possible

**Énoncé 1** :  
> *« Chaque individu paye sa part individuellement, il n'y a pas de partage d'assiette ou de bouteille. »*

**Énoncé 2** :  
> *« Une personne peut régler plusieurs commandes d'une même table, voire sa table intégralement. »*

**Clarification nécessaire** :  
Ces deux énoncés **ne sont pas contradictoires** mais nécessitent une modélisation précise :
*   Le système doit permettre de **splitter l'addition par article** (chacun paie ses propres plats)
*   Mais aussi de **regrouper plusieurs articles** si quelqu'un paie pour d'autres

**Impact Architecture** :  
La base de données doit lier chaque **Article** (ligne de commande) à un **Convive** (position à table) pour permettre la facturation individuelle ou groupée.

---

## B.3. OUBLIS ET MANQUES ÉVIDENTS

### B.3.1. Gestion des Réservations

**Constat** :  
Le document ne mentionne **jamais** de système de réservation, alors que :
*   Le Responsable de Salle "prend en charge le placement des clients"
*   Les cas d'utilisation incluaient une phase "Avant l'arrivée du client"

**Question** :  
*   Les réservations sont-elles gérées **manuellement** (cahier papier) ou faut-il prévoir un module numérique ?
*   Faut-il un système de confirmation SMS/Email ?

**Recommandation** :  
Intégrer une **fonctionnalité optionnelle** de gestion des réservations (planning des tables) dans la roadmap.

**→ Implémenté dans l'Annexe A (UC A.1.1 - Gérer Réservations avec RGPD)**

---

### B.3.2. Mode Dégradé (Perte de Connexion) - **BESOIN CRITIQUE**

**Constat** :  
Le système repose entièrement sur le WiFi (Fibre optique + bornes). **Aucune mention** d'un mode dégradé en cas de panne réseau.

**Risque** :  
*   Si le WiFi tombe, les serveurs ne peuvent plus :
    *   Consulter les stocks
    *   Envoyer les commandes
    *   Recevoir les notifications "plat prêt"

**Cas d'Usage Critique (Scénario Réel)** :

> [!CAUTION]
> **Scenario : WiFi Down Pendant Service du Soir (19h-22h30)**
> 
> 1. **19h15** : Borne WiFi tombe en panne (surcharge, coupure électrique, etc.)
> 2. **Impact immédiat** :
>    - 3 serveurs bloqués → **Impossible de prendre commandes** (app inutilisable)
>    - 20 tables actives → **Clients attendent** (temps service × 3)
>    - Cuisine ne reçoit pas commandes → **Perte CA estimée : 2 000€ sur 3h30**
> 3. **Sans mode offline** :
>    - Retour papier/stylo (régression totale)
>    - Risque erreurs retranscription (12% taux actuel)
>    - Perte recommandations vin → **CA vins -100%** pendant panne
> 4. **Durée panne WiFi moyenne constatée** : **45 minutes à 2 heures**

**Impact Business Sans Mode Offline** :

| Indicateur | Impact | Calcul |
|:-----------|:------:|:-------|
| **Perte CA** | 2 000€/panne | 20 tables × 4 couverts × 25€ moyen |
| **Temps blocage** | 45-120 min | Historique pannes restaurant |
| **Satisfaction client** | -40% | Attente prolongée, service ralenti |
| **Charge serveurs** | ×2 | Retour saisie papier + ressaisie post-panne |

**Recommandation HAUTE PRIORITÉ** :  
Prévoir un **mode offline SQLite** avec :

*   ✅ **Cache local persistant** des stocks (dernière valeur connue avant panne)
*   ✅ **Base de données SQLite locale** sur mobile Android (tables, menu, commandes en attente)
*   ✅ **File d'attente automatique** : Commandes stockées localement → Envoi auto à la reconnexion
*   ✅ **Synchronisation différée intelligente** :
    - Détection reconnexion réseau
    - Upload queue commandes (FIFO)
    - Résolution conflits (last-write-wins ou merge manuel)
*   ✅ **Indicateur visuel clair** "🔴 Mode Offline" sur interface mobile
*   ✅ **Limitations acceptées en mode offline** :
    - Stocks affichés = dernière valeur connue (risque obsolescence acceptable)
    - Notifications "plat prêt" désactivées (fallback sonnette cuisine)

**Contraintes Techniques** :

| Contrainte | Solution IT3 |
|:-----------|:-------------|
| Stockage local limité (Android) | SQLite max 500 Mo → Suffisant 1000 commandes offline |
| Conflits sync (2 serveurs modifient même table) | Last-write-wins avec timestamp + alerte admin |
| Sécurité données locales | Chiffrement AES-256 base SQLite (clé device) |
| Persistance après crash app | SQLite persist filesystem → Récupération auto |

**→ Implémenté dans IT3 (4 semaines) : [Annexe-A UC A.7 - Mode Offline SQLite + Sync](Annexe-A-Cas-Usage.md) + [Annexe-D IT3 Résilience](Annexe-D-Iterations-Architecture.md) + [Annexe-G DS3 Offline Sync](Annexe-G-Modelisation-Systeme.md)**

**→ Tests : 60 tests offline (Annexe-H) validant queue, sync, conflicts**

---

### B.3.3. Gestion des Pourboires

**Constat** :  
Aucune mention de la gestion des pourboires, alors que :
*   Le paiement par carte est privilégié
*   En France, le "service inclus" est la norme mais les pourboires volontaires existent

**Question** :  
*   Le TPE permet-il l'ajout d'un pourboire avant validation ?
*   Faut-il tracer les pourboires séparément (comptabilité, partage entre serveurs) ?

**Recommandation** :  
Interroger le client sur la politique de pourboire et prévoir une interface dédiée si nécessaire.

**→ Non implémenté (V2 potentielle)**

---

### B.3.4. Gestion des Annulations / Modifications de Commande

**Constat** :  
Rien n'est dit sur la possibilité d'annuler ou modifier une commande **après envoi** à la cuisine.

**Scénarios courants** :
*   Client change d'avis avant que le plat ne soit commencé
*   Erreur de saisie du serveur (mauvais plat sélectionné)

**Impact** :  
*   L'ERP "QuiCuisineIci" doit-il recevoir une notification d'annulation ?
*   Comment tracer ces modifications pour la conformité NF525 (inaltérabilité) ?

**Recommandation** :  
Prévoir un **workflow d'annulation** avec :
*   Notification vers l'ERP
*   Log de la modification (qui, quand, pourquoi) pour audit NF525

**→ Tracé dans AUDIT_LOGS (event_type='ORDER_MODIFIED')**

---

### B.3.5. Formation des Utilisateurs

**Constat** :  
Rien n'est dit sur la **formation** des serveurs, du responsable de salle et du caissier.

**Risque** :  
*   Adoption difficile de l'outil (résistance au changement)
*   Erreurs de manipulation (mauvaises commandes, encaissements erronés)

**Recommandation** :  
Inclure dans le projet :
*   Une **documentation utilisateur** (guides illustrés)
*   Une **session de formation** (1 jour en présentiel)
*   Un **mode démo/sandbox** pour s'entraîner

**→ Planifiée : Workshop TDD 2j (inclut formation système)**

---

### B.3.6. Maintenance et Support Technique

**Constat** :  
Aucune mention de la maintenance post-déploiement.

**Questions** :  
*   Qui assure le support en cas de bug (hotline ?) ?
*   Quelle est la SLA (délai de résolution) ?
*   Les mises à jour (nouvelles normes, évolutions) sont-elles comprises dans le forfait ?

**Recommandation** :  
Proposer un **contrat de mainten ance** (ex: TMA - Tierce Maintenance Applicative) avec :
*   Support niveau 1 (email/téléphone)
*   Mises à jour réglementaires incluses (NF525, RGPD)

**→ Budget 200€/mois infrastructure cloud (monitoring Grafana inclus)**

---

### B.3.7. Interface "Propriétaire" (Reporting)

**Constat** :  
Le propriétaire est identifié comme partie prenante critique, mais **aucune interface** dédiée n'est prévue pour lui.

**Besoin implicite** :  
*   Consulter le chiffre d'affaires journalier/mensuel
*   Voir les plats les plus vendus
*   Exporter les données comptables

**Recommandation** :  
Prévoir un **tableau de bord web** (Dashboard Analytics) accessible depuis un PC/Tablette.

**→ Implémenté : UC A.6.2 Rapports CA (IT4)**

---

## B.4. ZONES D'AMBIGUÏTÉ (Clarifications Requises)

### B.4.1. Carte Changeante "Tous les Jours"

**Énoncé** :  
> *« La carte évolue selon la saison, tous les jours. »*

**Ambiguïté** :  
*   La carte change-t-elle **littéralement** chaque jour (365 cartes différentes/an) ?
*   Ou s'agit-il de **variations saisonnières** (4 cartes/an avec ajustements hebdomadaires) ?

**Impact Technique** :  
*   Si changement quotidien → Interface de mise à jour ultra-simple pour le Chef (ou Responsable)
*   Si changement hebdo → Processus de validation plus classique

**Question au client** :  
*« À quelle fréquence réelle la carte est-elle mise à jour ? Par qui (Chef/Responsable) ? »*

**→ Hypothèse retenue : Mise à jour hebdomadaire par Admin (UC A.6.1 Gestion Menu)**

---

### B.4.2. "Une Unique Proposition" de Vin

**Énoncé** :  
> *« Une (unique proposition de) bouteille pourra être suggérée en fonction des plats principaux. »*

**Ambiguïté** :  
*   Suggérée **par qui** ? Le système automatiquement via algorithme ? Le serveur manuellement ?
*   **Une seule** bouteille proposée par table ou **une par plat principal** ?

**Clarification** :  
Hypothèse retenue : **Algorithme automatique** suggérant 1 vin **par plat principal** (pas 1 vin pour toute la table).

**Justification** :  
*   Table 4 personnes = potentiellement 4 plats différents → 4 suggestions vins possibles
*   Serveur libre d'accepter/refuser la suggestion

**→ Implémenté : UC A.3.1 Recommandation Vin (FK wine_pairing_id sur MENU_ITEMS)**

---

### B.4.3. Rôle du "Responsable de Salle"

**Énoncé** :  
> *« Le responsable de salle prend en charge le placement et le service en salle lorsqu'un groupe de clients se présente. »*

**Ambiguïté** :  
*   Le Responsable participe-t-il au **service** (prend des commandes comme les serveurs) ?
*   Ou son rôle se limite-t-il au **placement** et à la supervision ?

**Hypothèse retenue** :  
- **Rôle superviseur uniquement** : Assigne tables aux serveurs, gère planning réservations
- Ne prend PAS de commandes (privilège serveurs uniquement)

**Impact Architecture** :  
- Rôle `ROLE_FLOOR_MANAGER` distinct de `ROLE_SERVER` (RBAC)

**→ Implémenté : UC A.2.1 Accueillir et Assigner Table (Responsable de Salle)**

---

### B.4.4. Quantité de Stocks "Diminuée en Temps Réel"

**Énoncé** :  
> *« La quantité restante en stock sera diminuée en temps réel. »*

**Ambiguïté** :  
*   Qui gère le stock ? Le système actuel (notre projet) ou l'ERP "QuiCuisineIci" ?
*   Si c'est l'ERP, notre système doit-il **écrire** dans la base de l'ERP ou simplement **lire** ?

**Clarification** :  
Hypothèse retenue : **Notre système lit les stocks (API GET)**. L'ERP gère seul les stocks (décrémentation à la validation commande).

**Justification** :  
*   Évite conflits de synchronisation (deux systèmes écrivent dans même base)
*   ERP "QuiCuisineIci" reste authoritative pour stocks

**→ Implémenté : Cache Redis 30s pour réduire appels API Stocks (UC A.3.2)**

---

### B.4.5. Définition "Cuisine" vs "Salle"

**Constat** :  
Le document mélange parfois les termes "cuisine" et "salle" sans délimiter clairement les responsabilités.

**Clarifiation proposée** :  
- **Cuisine** : Tout ce qui touche à la préparation culinaire → Géré par ERP "QuiCuisineIci" (hors scope)
- **Salle** : Tout ce qui touche au contact client → **Scope de notre projet** :
  - Prise commandes (serveurs mobiles)
  - Encaissements (caissiers web)
  - Rapports admin (gérant web)

**→ Périmètre clarifié dans C9 Section 1.1 Présentation**

---

## B.5. PARTIES PRENANTES DÉTAILLÉES

### B.5.1. Parties Prenantes INTERNES

#### B.5.1.1. Le Propriétaire / Gérant du Restaurant

**Rôle** : Commanditaire du projet, décideur final.

**Enjeux** :
*   Rentabilité (augmentation du chiffre d'affaires, réduction des erreurs de commande)
*   Contrôle de gestion (reporting des ventes, statistiques via NF525)
*   Conformité légale (éviter les amendes fiscales et sanitaires)

**Besoins exprimés** :
*   Solution low-cost (pas d'investissement matériel autre que 3 téléphones)
*   Interface simple pour consultation des rapports (CA, plats les plus vendus)

**Motivations** :
- Moderniser restaurant (image innovante auprès clients)
- Augmenter CA vins (+15% escompté grâce recommandations)
- Conformité NF525 (éviter amende 7 500€)

**Contraintes** :
- Budget limité : ~10 000€ pour projet complet
- Deadline ferme : MVP 14 semaines (début saison touristique)

**Influence** : 🔴 **CRITIQUE** (décide Go/No-Go projet)

**Impact projet** : Définit les contraintes budgétaires et valide les choix technologiques.

---

#### B.5.1.2. Le Responsable de Salle

**Rôle** : Organisateur du service, interface entre clients et serveurs.

**Enjeux** :
*   Optimisation du placement (maximiser le taux d'occupation, gérer les flux)
*   Qualité de service (pas d'oubli, temps d'attente maîtrisé)

**Besoins** :
*   Accès au plan de salle en temps réel (tables occupées/libres)
*   Interface pour assigner les serveurs aux tables
*   Gestion des réservations (si implémentée)

**Motivations** :
- Réduire stress service (moins d'improvisation placement)
- Traçabilité demandes spécifiques clients (allergies, PMR)

**Contraintes** :
- Utilisation poste fixe seulement (pas de mobile)
- Résistance au changement (habitué cahier papier réservations)

**Fréquence utilisation** : 2x/jour (midi + soir), ~15 min/service

**Influence** : 🟠 **HAUTE** (peut bloquer adoption si UX complexe)

**Impact projet** : Utilisateur principal de l'**Interface Caisse/Ordinateur** (Module "Gestion Tables").

---

#### B.5.1.3. Les Serveurs (x3)

**Rôle** : Prise de commande, service en salle, relation client directe.

**Enjeux** :
*   Rapidité et efficacité (pas de retour en cuisine pour vérifier les stocks)
*   Précision de la commande (éviter les allergènes non signalés)
*   Conseil client (accords mets-vins)

**Besoins** :
*   Application mobile intuitive et réactive
*   Consultation des stocks en temps réel
*   Affichage des allergènes par plat
*   Suggestion automatique de vin (1 recommandation par plat)
*   Notifications "plat prêt" pour synchroniser le service

**Contraintes spécifiques** :
*   Chacun gère **sa** zone en exclusivité (pas d'intervention croisée)
*   Terminal Android personnel assigné
*   Pas de temps pour formation longue (max 2h)

**Motivations** :
- Augmenter pourboires (service plus pro)
- Moins d'erreurs commandes (réputation personnelle)

**Craintes** :
- Complexité outil (peur échec face client)
- Panne WiFi (pas de plan B)

**Fréquence utilisation** : 180 commandes/jour totales (60/serveur moy)

**Influence** : 🔴 **CRITIQUE** (échec adoption = échec projet)

**Impact projet** : Utilisateurs intensifs de l'**Application Mobile Serveur**.

---

#### B.5.1.4. Le Caissier / Hôte d'Accueil

**Rôle** : Encaissement des paiements, édition des tickets de caisse.

**Enjeux** :
*   Conformité fiscale (NF525)
*   Rapidité d'encaissement (éviter les files d'attente à la sortie)
*   Gestion des paiements divisés (chacun sa part)

**Besoins** :
*   Interface claire pour sélectionner les articles à facturer par client
*   Liaison TPE (Terminal de Paiement Électronique)
*   Impression tickets NF525

**Motivations** :
- Automatisation split bill (gain temps ×3)
- Traçabilité paiements (moins d'erreurs caisse)

**Contraintes** :
- Doit maîtriser conformité NF525 (responsabilité légale)
- Utilisation TPE obligatoire (pas contournement)

**Fréquence utilisation** : 95 paiements/jour moy

**Influence** : 🔴 **CRITIQUE** (garant légalité système)

**Impact projet** : Utilisateur de l'**Interface Caisse/Ordinateur** (Module "Encaissement").

---

#### B.5.1.5. Le Chef Cuisinier

**Rôle** : Production culinaire, gestion des stocks (via ERP externe "QuiCuisineIci").

**Enjeux** :
*   Réception en temps réel des commandes validées
*   Contrôle du rythme de production (éviter la surcharge)

**Besoins** :
*   **PAS de développement spécifique** dans ce projet (déjà couvert par l'ERP)
*   L'ERP doit **recevoir** les commandes depuis l'application mobile des serveurs
*   L'ERP doit **envoyer** les notifications "plat prêt" vers le serveur concerné

**Motivations** :
- Commandes numériques (lisibilité vs tickets papier)
- Respect rythme cuisine (pas de rush imprévu)

**Contraintes** :
- **Refus absolu** modification ERP actuel
- Communication API REST uniquement

**Fréquence interaction** : 180 commandes/jour reçues

**Influence** : 🟠 **HAUTE** (peut bloquer si API ERP incompatible)

**Impact projet** : Interface système (API) entre notre SI et l'ERP "QuiCuisineIci".

---

### B.5.2. Parties Prenantes EXTERNES

#### B.5.2.1. Les Clients Finaux

**Rôle** : Consommateurs du service, générateurs de revenus.

**Enjeux** :
*   Qualité de service (rapidité, précision des commandes)
*   Transparence (affichage des prix, allergènes, origine des produits)
*   Expérience fluide (paiement facile, respect des demandes spécifiques)

**Besoins** :
*   Serveurs informés et réactifs (grâce à l'application)
*   Respect des restrictions alimentaires (allergènes)
*   Paiement individuel possible

**Attentes** :
- Temps attente < 10 min (prise commande)
- Recommandations vins pertinentes

**Craintes** :
- Erreurs commandes (allergies non respectées)
- Paiement complexe (split bill mal géré)

**Fréquence** : 180 couverts/jour

**Influence** : 🟡 **MOYENNE** (indirecte via avis Google/TripAdvisor)

**Impact projet** : Bénéficiaires indirects de la qualité du système (pas d'interaction directe avec le SI).

---

#### B.5.2.2. L'Administration Fiscale (DGFIP)

**Rôle** : Contrôle de la conformité fiscale (TVA, lutte anti-fraude).

**Enjeux** :
*   Vérifier l'inaltérabilité des transactions
*   Auditer les archives (6 ans légalement)

**Besoins** :
*   Certification NF525 ou LNE du logiciel de caisse
*   Accès aux exports de données en cas de contrôle

**Contraintes imposées** :
- Hash chaîné SHA-256 obligatoire
- Signature électronique RSA-2048
- Archivage 6 ans minimum
- Tickets Z quotidiens

**Fréquence audit** : Contrôle probabiliste (1 resto/50 par an)

**Influence** : 🔴 **CRITIQUE** (amende 7 500€ si non-conforme)

**Impact projet** : **Contrainte réglementaire forte** (conception NF525-compliant dès l'origine).

---

#### B.5.2.3. Les Organismes de Certification (AFNOR, LNE)

**Rôle** : Audit et certification du logiciel de caisse.

**Enjeux** :
*   S'assurer que l'application respecte les critères ISCA (Inaltérabilité, Sécurisation, Conservation, Archivage).

**Besoins** :
*   Documentation technique complète
*   Accès au code source / architecture pour audit

**Process certification** :
1. Dossier technique (architecture, tests)
2. Audit code source (3-6 mois)
3. Certification délivrée (validité 3 ans)

**Coût** : 500€/an (renouvellement)

**Influence** : 🟠 **HAUTE** (pas de certification = pas de production)

**Impact projet** : Phase de certification **post-développement** (prévoir 3-6 mois).

---

#### B.5.2.4. Le Prestataire ERP "QuiCuisineIci"

**Rôle** : Éditeur du logiciel de gestion cuisine déjà en place.

**Enjeux** :
*   Maintenir la compatibilité avec ses clients existants
*   Éviter les modifications coûteuses de son API

**Besoins** :
*   **Documentation de l'API** (endpoints, formats, authentification)
*   Stabilité des flux (pas de surcharge réseau)

**SLA attendu** :
- Disponibilité API ≥ 99% (uptime)
- Latence P95 < 500ms

**Risques** :
- Documentation API incomplète (bloquant IT1)
- Refus modification API (incompatibilité)

**Fréquence interaction** : 180 POST + 60 callbacks/jour

**Influence** : 🔴 **CRITIQUE** (dépendance technique totale)

**Impact projet** : **Dépendance technique critique** → Nécessite une phase d'analyse de l'API existante.

---

#### B.5.2.5. Les Fournisseurs d'Équipements

**Terminal de Paiement Électronique (TPE)** :
- Fournisseur : Banque du restaurant (ex: Ingenico/V érifone)
- Contraintes : VLAN 10 isolé (PCI DSS)
- SLA : Support 24/7 (panne critique métier)

**Mobiles Android (x3)** :
- Budget : ~200€/appareil (Samsung Galaxy A54 ou similaire)
- OS : Android 12 minimum (React Native 0.73 compatible)

**Imprimante Thermique** :
- Format tickets : 80mm
- Connectivité : USB ou Ethernet
- Compatible ESC/POS (standard tickets NF525)

**Influence** : 🟢 **BASSE** (équipements standards)

---

#### B.5.2.6. Hébergeur / Cloud Provider

**Prérequis infrastructure** :
- Serveur dédié 4 CPU, 16 Go RAM
- PostgreSQL 15, Redis 7.2 cluster
- Bande passante : ~100 Go/mois

**Options** :
1. **OVH** : ~80€/mois (serveur dédié France)
2. **AWS** : ~150€/mois (EC2 t3.medium + RDS + ElastiCache)
3. **Hetzner** : ~50€/mois (VPS Cloud)

**Recommandation** : Hetzner (rapport qualité/prix)

**Influence** : 🟢 **BASSE** (choix technique interne)

---

## B.6. Tableau Récapitulatif Parties Prenantes

### Acteurs Internes

| Acteur | Nb | Rôle | Fréquence Usage | Influence | Criticité Projet |
|:-------|:--:|:-----|:---------------:|:---------:|:----------------:|
| **Gérant** | 1 | Sponsor, décisions | Hebdo (rapports) | 🔴 CRITIQUE | Valide budget/délais |
| **Responsable Salle** | 1 | Placement tables, réservations | 2×/jour (15 min) | 🟠 HAUTE | Adoption clé |
| **Serveurs** | 3 | Prise commandes mobile | 60 cmd/jour chacun | 🔴 CRITIQUE | Utilisateurs principaux |
| **Caissiers** | 2 | Encaissements, NF525 | 95 paiements/jour | 🔴 CRITIQUE | Garants légalité |
| **Chef Cuisinier** | 1 | Production (ERP externe) | 180 cmd/jour reçues | 🟠 HAUTE | Dépendance API |

### Acteurs Externes

| Acteur | Type | Interface | Fréquence | Criticité | Risque Principal |
|:-------|:-----|:----------|:----------|:----------|:-----------------|
| **Clients** | Humain | Indirect (serveurs) | 180/jour | 🟡 MOYENNE | Avis négatifs si bugs |
| **DGFIP** | Réglementaire | Audit (occasionnel) | 1/50/an | 🔴 CRITIQUE | Amende 7 500€ |
| **AFNOR/LNE** | Certification | Audit code (3-6 mois) | Unique | 🟠 HAUTE | Blocage prod |
| **ERP QuiCuisineIci** | Système | REST API bidirectionnel | 240 req/jour | 🔴 CRITIQUE | API indisponible |
| **TPE Bancaire** | Système | Protocole propriétaire | 95 tran/jour | 🔴 CRITIQUE | Panne paiements |
| **API Stocks** | Système | REST lecture seule | 500 req/jour | 🟡 MOYENNE | Fallback base locale |
| **Prometheus/Grafana** | Monitoring | Scraping métriques | 1/15s | 🟢 BASSE | Perte visibilité (non bloquant) |
| **Elasticsearch/Kibana** | Logs | Push JSON | 2000/jour | 🟢 BASSE | Perte logs (non bloquant) |

---

## B.7. Analyse Influence vs Intérêt (Matrice Pouvoir/Intérêt)

### Haute Influence + Haut Intérêt (Gestion Active)
- 🔴 **Gérant** : Sponsorship total
- 🔴 **Serveurs** : Adoption critique
- 🔴 **Caissiers** : Conformité NF525
- 🔴 **ERP QuiCuisineIci** : Dépendance technique

**→ Communication fréquente, implication décision**

### Haute Influence + Bas Intérêt (Satisfaire)
- 🟠 **DGFIP** : Conformité obligatoire mais contrôle rare
- 🟠 **Certification (AFNOR)** : Process post-dev

**→ Informer régulièrement, éviter surprises**

### Basse Influence + Haut Intérêt (Informer)
- 🟡 **Clients finaux** : Bénéficiaires indirects
- 🟡 **Responsable Salle** : Utilisateur secondaire

**→ Communication transparente, feedback écouté**

### Basse Influence + Bas Intérêt (Surveiller)
- 🟢 **Hébergeur** : Prestataire technique
- 🟢 **Fournisseurs équipements** : Commodités

**→ Monitoring minimal**

---

## B.8. Risques Liés aux Parties Prenantes

| Risque | Partie Prenante | Probabilité | Impact | Mitigation |
|:-------|:----------------|:-----------:|:------:|:-----------|
| Refus adoption serveurs | Serveurs | MOYENNE | 🔴 CRITIQUE | Formation 2j, UX intuitive, tests terrain |
| Documentation API ERP manquante | ERP QuiCuisineIci | HAUTE | 🔴 CRITIQUE | Plan B Scénario B (tickets papier temporaires) |
| Échec certification NF525 | AFNOR/LNE | FAIBLE | 🔴 CRITIQUE | POC crypto semaine 1, bibliothèque certifiée |
| Audit PCI DSS réseau TPE | DGFIP | MOYENNE | 🟠 HAUTE | Audit externe IT2, VLAN 10 dédié |
| Client insatisfait (allergies) | Clients | FAIBLE | 🟡 MOYENNE | Affichage allergènes prominent, validation serveur |

---
