# C2 - Analyse Critique du Besoin Client

## Introduction
En tant que prestataire chargé d'analyser le besoin et de proposer une solution logicielle, il est crucial de procéder à une **reprise critique** de la demande initiale. Cette démarche permet d'identifier :
*   Les **contradictions** dans les exigences.
*   Les **oublis** ou **manques évidents**.
*   Les **zones d'ambiguïté** nécessitant des clarifications.

---

## 1. CONTRADICTIONS Identifiées

### 1.1. Titre vs Périmètre Réel
**Énoncé** : *« Gestion d'un restaurant, salle et cuisine »*  
**Réalité** : Le Chef Cuisinier refuse que le projet touche à la cuisine (ERP "QuiCuisineIci" déjà en place).

**Contradiction** :  
Le titre suggère une gestion intégrée **salle + cuisine**, alors que le périmètre réel se limite à la **salle uniquement** (prise de commande, encaissement).

**Recommandation** :  
Renommer le projet : *« Gestion de la Salle et Interface ERP Cuisine »* pour clarifier le scope.

---

### 1.2. Bar "Non Actif" mais TPE Présent
**Énoncé** :  
> *« Le restaurant dispose d'un bar non actif à ce jour. À cet emplacement, il existe un unique terminal de paiement placé au niveau de la caisse enregistreuse. »*

**Contradiction** :  
*   Le bar est dit "non actif", pourtant un TPE y est installé.
*   Aucune mention de la gestion des **apéritifs** (pourtant cités comme possibles) ni de leur lieu de préparation (bar fermé ? cuisine ?).

**Ambiguïté** :  
*   Qui prépare les apéritifs si le bar est fermé ?
*   Faut-il prévoir un module "Réactivation du Bar" dans la roadmap ?

**Recommandation** :  
Clarifier avec le client :  
→ *Les apéritifs sont-ils servis depuis la cuisine ou le bar est-il partiellement actif ?*  
→ *Le TPE est-il déjà relié à un système de caisse fonctionnel ?*

---

### 1.3. Paiement Individuel vs Partage Possible
**Énoncé 1** :  
> *« Chaque individu paye sa part individuellement, il n'y a pas de partage d'assiette ou de bouteille. »*

**Énoncé 2** :  
> *« Une personne peut régler plusieurs commandes d'une même table, voire sa table intégralement. »*

**Clarification nécessaire** :  
Ces deux énoncés **ne sont pas contradictoires** mais nécessitent une modélisation précise :
*   Le système doit permettre de **splitter l'addition par article** (chacun paie ses propres plats).
*   Mais aussi de **regrouper plusieurs articles** si quelqu'un paie pour d'autres.

**Impact Architecture** :  
La base de données doit lier chaque **Article** (ligne de commande) à un **Convive** (position à table) pour permettre la facturation individuelle ou groupée.

---

## 2. OUBLIS ET MANQUES ÉVIDENTS

### 2.1. Gestion des Réservations
**Constat** :  
Le document ne mentionne **jamais** de système de réservation, alors que :
*   Le Responsable de Salle "prend en charge le placement des clients".
*   Les cas d'utilisation (C1) incluaient une phase "Avant l'arrivée du client".

**Question** :  
*   Les réservations sont-elles gérées **manuellement** (cahier papier) ou faut-il prévoir un module numérique ?
*   Faut-il un système de confirmation SMS/Email ?

**Recommandation** :  
Intégrer une **fonctionnalité optionnelle** de gestion des réservations (planning des tables) dans la roadmap.

---

### 2.2. Mode Dégradé (Perte de Connexion)
**Constat** :  
Le système repose entièrement sur le Wifi (Fibre optique + bornes). **Aucune mention** d'un mode dégradé en cas de panne réseau.

**Risque** :  
*   Si le Wifi tombe, les serveurs ne peuvent plus :
    *   Consulter les stocks.
    *   Envoyer les commandes.
    *   Recevoir les notifications "plat prêt".

**Manque évident** :  
*   Pas de procédure de repli (commandes papier ? sauvegarde locale sur mobile ?).
*   Pas de synchronisation différée une fois le réseau rétabli.

**Recommandation** :  
Prévoir un **mode offline** avec :
*   **Cache local** des stocks (dernière valeur connue).
*   **File d'attente locale** des commandes (envoi automatique à la reconnexion).
*   **Alerte visuelle** "Mode Dégradé" sur l'interface mobile.

---

### 2.3. Gestion des Pourboires
**Constat** :  
Aucune mention de la gestion des pourboires, alors que :
*   Le paiement par carte est privilégié.
*   En France, le "service inclus" est la norme mais les pourboires volontaires existent.

**Question** :  
*   Le TPE permet-il l'ajout d'un pourboire avant validation ?
*   Faut-il tracer les pourboires séparément (comptabilité, partage entre serveurs) ?

**Recommandation** :  
Interroger le client sur la politique de pourboire et prévoir une interface dédiée si nécessaire.

---

### 2.4. Gestion des Annulations / Modifications de Commande
**Constat** :  
Rien n'est dit sur la possibilité d'annuler ou modifier une commande **après envoi** à la cuisine.

**Scénarios courants** :
*   Client change d'avis avant que le plat ne soit commencé.
*   Erreur de saisie du serveur (mauvais plat sélectionné).

**Impact** :  
*   L'ERP "QuiCuisineIci" doit-il recevoir une notification d'annulation ?
*   Comment tracer ces modifications pour la conformité NF525 (inaltérabilité) ?

**Recommandation** :  
Prévoir un **workflow d'annulation** avec :
*   Notification vers l'ERP.
*   Log de la modification (qui, quand, pourquoi) pour audit NF525.

---

### 2.5. Formation des Utilisateurs
**Constat** :  
Rien n'est dit sur la **formation** des serveurs, du responsable de salle et du caissier.

**Risque** :  
*   Adoption difficile de l'outil (résistance au changement).
*   Erreurs de manipulation (mauvaises commandes, encaissements erronés).

**Recommandation** :  
Inclure dans le projet :
*   Une **documentation utilisateur** (guides illustrés).
*   Une **session de formation** (1 jour en présentiel).
*   Un **mode démo/sandbox** pour s'entraîner.

---

### 2.6. Maintenance et Support Technique
**Constat** :  
Aucune mention de la maintenance post-déploiement.

**Questions** :  
*   Qui assure le support en cas de bug (hotline ?) ?
*   Quelle est la SLA (délai de résolution) ?
*   Les mises à jour (nouvelles normes, évolutions) sont-elles comprises dans le forfait ?

**Recommandation** :  
Proposer un **contrat de maintenance** (ex: TMA - Tierce Maintenance Applicative) avec :
*   Support niveau 1 (email/téléphone).
*   Mises à jour réglementaires incluses (NF525, RGPD).

---

### 2.7. Interface "Propriétaire" (Reporting)
**Constat** :  
Le propriétaire est identifié comme partie prenante critique, mais **aucune interface** dédiée n'est prévue pour lui.

**Besoin implicite** :  
*   Consulter le chiffre d'affaires journalier/mensuel.
*   Voir les plats les plus vendus.
*   Exporter les données comptables.

**Recommandation** :  
Prévoir un **tableau de bord web** (Dashboard Analytics) accessible depuis un PC/Tablette.

---

## 3. ZONES D'AMBIGUÏTÉ (Clarifications Requises)

### 3.1. Carte Changeante "Tous les Jours"
**Énoncé** :  
> *« La carte évolue selon la saison, tous les jours. »*

**Ambiguïté** :  
*   La carte change-t-elle **littéralement** chaque jour (365 cartes différentes/an) ?
*   Ou s'agit-il de **variations saisonnières** (4 cartes/an avec ajustements hebdomadaires) ?

**Impact Technique** :  
*   Si changement quotidien → Interface de mise à jour ultra-simple pour le Chef (ou Responsable).
*   Si changement hebdo → Processus de validation plus classique.

**Question au client** :  
*« À quelle fréquence réelle la carte est-elle mise à jour ? Par qui (Chef/Responsable) ? »*

---

### 3.2. "Une Unique Proposition" de Vin
**Énoncé** :  
> *« C'est l'application qui doit proposer les associations plat-vin recommandé (une unique proposition). »*

**Ambiguïté** :  
*   L'algorithme propose-t-il toujours **le même** vin pour un plat donné (base de données statique) ?
*   Ou y a-t-il une **logique dynamique** (ex: si ce vin est en rupture, proposer une alternative) ?

**Impact Technique** :  
*   Cas simple : Table de correspondance fixe `Plat → Vin`.
*   Cas avancé : Moteur de recommandation intelligent (Machine Learning ?).

**Recommandation** :  
Pour la V1, privilégier une **table statique** éditable par le Responsable/Chef.

---

### 3.3. Notifications "Plat Prêt" : Détail du Flux
**Énoncé** :  
> *« Recevoir les notifications sur le téléphone du serveur qui gère la table correspondante. »*

**Ambiguïté** :  
*   L'ERP "QuiCuisineIci" envoie-t-il une notification **par plat** ou **par table** (ticket complet) ?
*   Le serveur voit-il *"Plat Principal Table 5 prêt"* ou *"Commande Table 5 complète"* ?

**Impact UX** :  
La granularité de la notification influence l'organisation du service (service "à l'assiette" vs "synchronisé").

**Question au client** :  
*« L'ERP QuiCuisineIci peut-il notifier plat par plat ou seulement par ticket complet ? »*

---

## 4. POINTS DE VIGILANCE CONTRACTUELS

### 4.1. Dépendance à l'ERP "QuiCuisineIci"
**Risque** :  
*   Aucune documentation API fournie par le client.
*   Si l'éditeur de l'ERP fait évoluer son API → risque de rupture.

**Recommandation** :  
*   Exiger une **documentation officielle** de l'API avant démarrage.
*   Prévoir une **clause de responsabilité** : si l'API change, coût supplémentaire pour adaptation.

---

### 4.2. Certification NF525 : Délai et Coût
**Risque** :  
*   La certification peut prendre **3 à 6 mois** après développement.
*   Coût additionnel (audit, redevance annuelle).

**Recommandation** :  
*   Intégrer ces délais et coûts dans le planning et le budget.
*   Prévoir une **phase de pré-audit** (auto-évaluation) avant certification officielle.

---

## 5. SYNTHÈSE DES RECOMMANDATIONS

| Catégorie | Point à Clarifier | Priorité | Action |
| :--- | :--- | :---: | :--- |
| **Contradiction** | Titre "Salle + Cuisine" vs Scope réel | Moyenne | Renommer le projet |
| **Oubli** | Mode dégradé (perte Wifi) | **Haute** | Prévoir mode offline |
| **Oubli** | Interface Propriétaire (Reporting) | Moyenne | Ajouter Dashboard |
| **Oubli** | Gestion Réservations | Basse | Phase 2 (optionnel) |
| **Ambiguïté** | Fréquence MAJ Carte | **Haute** | Question client |
| **Ambiguïté** | Granularité Notifications ERP | **Haute** | Analyser API ERP |
| **Vigilance** | Dépendance API ERP | **Critique** | Clause contractuelle |
| **Vigilance** | Délai Certification NF525 | **Critique** | Intégrer au planning |

---

## Conclusion
Cette analyse critique met en évidence que **le besoin initial, bien que structuré, nécessite des clarifications importantes** avant de procéder à la conception détaillée. Les points marqués "Haute" ou "Critique" doivent faire l'objet d'un **échange formel avec le client** sous forme de *liste de questions* ou *atelier de cadrage*.
