# C5 - Sélection des Technologies et Outils

## Partie 1 : Liste Complète des Fonctionnalités de l'Application

### Vue d'Ensemble
L'application de gestion de salle de restaurant intègre **3 interfaces principales** (Mobile Serveur, Caisse Web, Interface Admin) et communique avec un **ERP cuisine externe** ("QuiCuisineIci"). La liste ci-dessous recense **toutes les fonctionnalités** identifiées dans les phases C1-C4.

---

## 1. GESTION DES TABLES

> **Contexte Restaurant** : 12 tables de 4 personnes + 8 tables de 6 personnes (20 tables total). Pas de rapprochement ou extension possible.

### 1.1. Consultation État des Tables
- **F-TAB-001** : Visualiser le plan de salle (disposition des tables)
- **F-TAB-002** : Afficher le statut de chaque table (Libre / Occupée / Réservée / Sale)
- **F-TAB-003** : Voir le nombre de couverts par table
- **F-TAB-004** : Identifier le serveur assigné à chaque table
- **F-TAB-005** : Consulter l'heure d'arrivée des clients (temps écoulé)

### 1.2. Gestion Occupation
- **F-TAB-006** : Ouvrir une table (passer de "Libre" à "Occupée")
- **F-TAB-007** : Fermer une table (après paiement → "Sale")
- **F-TAB-008** : Réassigner une table à un autre serveur
- **F-TAB-009** : Marquer une table comme "Prête" (après nettoyage)
- **F-TAB-010** : Modifier le nombre de couverts d'une table active

### 1.3. Réservations (V2 - Hors MVP)
- **F-TAB-011** : Créer une réservation (nom, téléphone, date/heure, nb personnes)
- **F-TAB-012** : Consulter les réservations du jour
- **F-TAB-013** : Modifier/annuler une réservation
- **F-TAB-014** : Envoyer SMS de confirmation automatique
- **F-TAB-015** : Marquer une réservation comme "Arrivée" (convertir en table occupée)

---

## 2. GESTION DU MENU

> **Contexte Carte** : Menu quotidien « fait maison » avec produits frais : 3 entrées, 2 plats, 5 desserts. Carte renouvelée chaque jour. Service au verre (vin/bière).

### 2.1. Consultation Menu
- **F-MENU-001** : Afficher la carte complète (Entrées, Plats, Desserts, Boissons)
- **F-MENU-002** : Voir les prix de chaque plat
- **F-MENU-003** : Consulter les allergènes (INCO) pour chaque plat
- **F-MENU-004** : Afficher les suggestions du jour / plats du jour
- **F-MENU-005** : Voir la disponibilité en temps réel (stocks ERP)
- **F-MENU-006** : Filtrer par catégorie (Entrées / Plats / Desserts / Végétarien / Sans gluten)
- **F-MENU-007** : Rechercher un plat par nom

### 2.2. Recommandations Vin (Automatiques)
- **F-MENU-008** : Afficher suggestion vin pour un plat sélectionné (1 seule proposition)
- **F-MENU-009** : Consulter prix/description du vin recommandé
- **F-MENU-010** : Ajouter directement le vin suggéré au panier (service au verre)

### 2.3. Gestion Administrative Menu (Admin)
- **F-MENU-011** : Ajouter un nouveau plat au menu
- **F-MENU-012** : Modifier un plat existant (nom, prix, description)
- **F-MENU-013** : Associer un vin recommandé à un plat
- **F-MENU-014** : Désactiver/réactiver un plat temporairement
- **F-MENU-015** : Supprimer un plat définitivement
- **F-MENU-016** : Définir les allergènes pour un plat
- **F-MENU-017** : Télécharger une photo du plat
- **F-MENU-018** : Créer des menus spéciaux (Menu du Jour, Menu Groupe)

---

## 3. GESTION DES COMMANDES

### 3.1. Prise de Commande (Mobile Serveur)
- **F-CMD-001** : Créer une nouvelle commande pour une table
- **F-CMD-002** : Ajouter des plats au panier (quantité, variantes)
- **F-CMD-003** : Ajouter des commentaires/personnalisations par plat ("Sans oignons", "Bien cuit")
- **F-CMD-004** : Modifier une commande avant envoi
- **F-CMD-005** : Supprimer un article du panier
- **F-CMD-006** : Valider et envoyer la commande à la cuisine (ERP)
- **F-CMD-007** : Afficher récapitulatif de la commande (prix total, détail)

### 3.2. Suivi Commandes
- **F-CMD-008** : Voir toutes les commandes actives d'une table
- **F-CMD-009** : Consulter le statut de chaque plat (En préparation / Prêt / Servi)
- **F-CMD-010** : Recevoir notification push "Plat prêt" (WebSocket)
- **F-CMD-011** : Marquer un plat comme "Servi"
- **F-CMD-012** : Envoyer un rappel cuisine si délai trop long (V2)
- **F-CMD-013** : Annuler un plat avant préparation (demande cuisine)

### 3.3. Historique Commandes
- **F-CMD-014** : Consulter l'historique des commandes d'une table
- **F-CMD-015** : Réimprimer un ticket de commande cuisine
- **F-CMD-016** : Exporter les commandes du jour (CSV/PDF) (Admin)

---

## 4. GESTION DES PAIEMENTS (CAISSE)

### 4.1. Encaissement
- **F-PAY-001** : Afficher l'addition d'une table (détail plats + total)
- **F-PAY-002** : Calculer automatiquement le total TTC (TVA incluse)
- **F-PAY-003** : **Split Bill** : Diviser l'addition par personne (chaque client paye sa propre consommation)
- **F-PAY-004** : Encaisser par Carte Bancaire (intégration TPE)
- **F-PAY-005** : Encaisser en Espèces (saisie montant, calcul monnaie à rendre)
- **F-PAY-006** : Encaisser par Ticket Restaurant / Chèque-Déjeuner
- **F-PAY-007** : Paiement global : une personne paye pour plusieurs couverts d'une même table
- **F-PAY-008** : Appliquer une remise manuelle (%, €) avec justification (Admin)

### 4.2. Tickets et Justificatifs
- **F-PAY-009** : Imprimer ticket NF525 (montant, date, hash, signature)
- **F-PAY-010** : Envoyer ticket par email (si client fournit adresse)
- **F-PAY-011** : Réimprimer dernier ticket (en cas d'erreur imprimante)
- **F-PAY-012** : Imprimer facture détaillée (pour entreprises)

### 4.3. Gestion Pourboires (V2)
- **F-PAY-013** : Enregistrer un pourboire (montant, serveur bénéficiaire)
- **F-PAY-014** : Répartir automatiquement pourboires entre équipe
- **F-PAY-015** : Exporter relevé pourboires mensuel par serveur

---

## 5. CONFORMITÉ FISCALE (NF525)

### 5.1. Inaltérabilité
- **F-NF525-001** : Enregistrer chaque transaction dans journal inaltérable (`audit_logs`)
- **F-NF525-002** : Générer hash SHA-256 pour chaque transaction
- **F-NF525-003** : Chaîner les hashes (hash N dépend de hash N-1)
- **F-NF525-004** : Bloquer toute suppression/modification dans `audit_logs` (trigger SQL)

### 5.2. Sécurisation
- **F-NF525-005** : Signer cryptographiquement les transactions (RSA)
- **F-NF525-006** : Vérifier automatiquement l'intégrité de la chaîne de hashes (cron quotidien)
- **F-NF525-007** : Alerter admin si rupture de chaîne détectée

### 5.3. Conservation et Archivage
- **F-NF525-008** : Effectuer clôture journalière automatique (Z de caisse)
- **F-NF525-009** : Archiver les clôtures (conservation 6 ans minimum)
- **F-NF525-010** : Exporter données fiscales au format XML/JSON (demande administration)
- **F-NF525-011** : Générer rapport mensuel/annuel pour expert-comptable

---

## 6. AUTHENTIFICATION ET AUTORISATIONS (RBAC)

### 6.1. Authentification
- **F-AUTH-001** : Connexion par login/mot de passe
- **F-AUTH-002** : Génération token JWT (expiration 8h)
- **F-AUTH-003** : Déconnexion (invalidation token)
- **F-AUTH-004** : Récupération mot de passe oublié (email)
- **F-AUTH-005** : Changement mot de passe forcé (premier login)

### 6.2. Gestion des Rôles (RBAC)
- **F-AUTH-006** : Rôle **Serveur** : Accès menu, commandes, tables (lecture seule paiements)
- **F-AUTH-007** : Rôle **Caissier** : Accès paiements, clôtures, tickets
- **F-AUTH-008** : Rôle **Admin** : Accès complet (gestion menu, utilisateurs, rapports)
- **F-AUTH-009** : Bloquer accès endpoints selon rôle (middleware)

### 6.3. Gestion Utilisateurs (Admin)
- **F-AUTH-010** : Créer un nouvel utilisateur (serveur/caissier)
- **F-AUTH-011** : Modifier rôle d'un utilisateur
- **F-AUTH-012** : Désactiver/réactiver un compte utilisateur
- **F-AUTH-013** : Consulter logs de connexion (audit sécurité)

---

## 7. MODE HORS LIGNE (OFFLINE)

### 7.1. Synchronisation Données
- **F-OFFLINE-001** : Synchroniser menu en local (SQLite) toutes les 5 minutes
- **F-OFFLINE-002** : Synchroniser état des tables en local toutes les 10 secondes
- **F-OFFLINE-003** : Synchroniser stocks ERP en local toutes les 30 secondes

### 7.2. Fonctionnement Dégradé
- **F-OFFLINE-004** : Détecter perte de connexion réseau (NetInfo)
- **F-OFFLINE-005** : Afficher bannière "Mode Hors Ligne" en UI
- **F-OFFLINE-006** : Enregistrer commandes localement (file d'attente SQLite)
- **F-OFFLINE-007** : Bloquer paiements en mode offline (alerte caissier)

### 7.3. Réconciliation
- **F-OFFLINE-008** : Synchroniser automatiquement commandes à reconnexion
- **F-OFFLINE-009** : Gérer conflits (si table modifiée ailleurs pendant offline)
- **F-OFFLINE-010** : Notifier serveur des commandes synchronisées avec succès

---

## 8. NOTIFICATIONS TEMPS RÉEL (WEBSOCKET)

### 8.1. Notifications Cuisine → Serveurs
- **F-NOTIF-001** : Notifier serveur quand plat prêt (push mobile)
- **F-NOTIF-002** : Notifier serveur si plat annulé par cuisine (rupture stock)
- **F-NOTIF-003** : Notifier tous serveurs si menu modifié (ajout/retrait plat)

### 8.2. Notifications Internes
- **F-NOTIF-004** : Notifier caissier quand table demande addition
- **F-NOTIF-005** : Notifier admin en cas d'erreur système critique
- **F-NOTIF-006** : Afficher notification en temps réel état des tables (occupée/libre)

---

## 9. RAPPORTS ET STATISTIQUES (ADMIN)

### 9.1. Rapports Journaliers
- **F-STAT-001** : Consulter chiffre d'affaires du jour
- **F-STAT-002** : Voir nombre de couverts servis
- **F-STAT-003** : Lister les plats les plus vendus
- **F-STAT-004** : Calculer ticket moyen par table
- **F-STAT-005** : Consulter répartition paiements (CB/Espèces)

### 9.2. Rapports Périodiques
- **F-STAT-006** : Générer rapport hebdomadaire (PDF)
- **F-STAT-007** : Générer rapport mensuel (export Excel)
- **F-STAT-008** : Comparer performances (semaine N vs semaine N-1)
- **F-STAT-009** : Analyser heures de pointe (affluence par tranche horaire)

### 9.3. Indicateurs Métier
- **F-STAT-010** : Calculer taux de remplissage salle (% tables occupées)
- **F-STAT-011** : Mesurer temps moyen de service par table
- **F-STAT-012** : Identifier serveur le plus performant (nb commandes)

---

## 10. INTÉGRATION ERP "QUICUISINEICI"

### 10.1. Envoi Commandes
- **F-ERP-001** : Envoyer commande cuisine via API REST (`POST /kitchen/orders`)
- **F-ERP-002** : Inclure détails (table, plats, quantités, commentaires)
- **F-ERP-003** : Recevoir accusé réception ERP (order_id cuisine)
- **F-ERP-004** : Gérer retry automatique si ERP indisponible (circuit breaker)

### 10.2. Réception Mises à Jour
- **F-ERP-005** : Recevoir notification "Plat Prêt" de l'ERP
- **F-ERP-006** : Recevoir notification "Rupture stock" (plat indisponible)
- **F-ERP-007** : Synchroniser stocks en temps réel (polling toutes les 30s)

### 10.3. Synchronisation Menu
- **F-ERP-008** : Importer menu initial depuis ERP (plats, prix)
- **F-ERP-009** : Synchroniser modifications menu si chef update carte
- **F-ERP-010** : Détecter plats absents ERP mais présents salle (alerte incohérence)

---

## 11. CONFORMITÉ RGPD

### 11.1. Consentement
- **F-RGPD-001** : Afficher formulaire consentement lors réservation
- **F-RGPD-002** : Enregistrer consentement (date, IP, type)
- **F-RGPD-003** : Permettre révocation consentement (opt-out marketing)

### 11.2. Droits Utilisateurs
- **F-RGPD-004** : Exporter données personnelles client (portabilité)
- **F-RGPD-005** : Anonymiser données client (droit à l'oubli)
- **F-RGPD-006** : Supprimer automatiquement données > 3 ans (cron)

### 11.3. Sécurité Données
- **F-RGPD-007** : Chiffrer données sensibles en BDD (AES-256)
- **F-RGPD-008** : Logs accès données personnelles (audit CNIL)
- **F-RGPD-009** : Afficher politique de confidentialité accessible

---

## 12. ADMINISTRATION SYSTÈME

### 12.1. Monitoring et Logs
- **F-ADMIN-001** : Consulter logs applicatifs centralisés (Kibana)
- **F-ADMIN-002** : Visualiser métriques temps réel (Grafana)
- **F-ADMIN-003** : Recevoir alertes automatiques (Slack) si erreur critique
- **F-ADMIN-004** : Consulter health checks systèmes (`/health` endpoint)

### 12.2. Maintenance
- **F-ADMIN-005** : Effectuer backup manuel BDD (PostgreSQL dump)
- **F-ADMIN-006** : Restaurer backup en cas de corruption
- **F-ADMIN-007** : Vider cache Redis manuellement (invalidation forcée)
- **F-ADMIN-008** : Redémarrer services API sans downtime (load balancer)

### 12.3. Configuration
- **F-ADMIN-009** : Modifier paramètres globaux (horaires service, nb tables)
- **F-ADMIN-010** : Configurer taux TVA par catégorie de plats
- **F-ADMIN-011** : Définir langues disponibles interface (FR/EN)

---

## 13. SÉCURITÉ RÉSEAU (PCI DSS)

### 13.1. Segmentation Réseau
- **F-SEC-001** : Isoler VLAN monétique (Caisse + TPE) du VLAN métier (Mobiles)
- **F-SEC-002** : Configurer firewall (règles inter-VLAN)
- **F-SEC-003** : Bloquer accès Internet depuis VLAN monétique (sauf TPE bancaire)

### 13.2. Chiffrement
- **F-SEC-004** : Forcer HTTPS/TLS 1.3 pour toutes communications API
- **F-SEC-005** : Chiffrer données en transit (certificat SSL valide)
- **F-SEC-006** : Chiffrer données sensibles au repos (BDD)

### 13.3. Audit Sécurité
- **F-SEC-007** : Effectuer scan vulnérabilités mensuel (OWASP Top 10)
- **F-SEC-008** : Tester pénétration annuelle (pentesting)
- **F-SEC-009** : Tenir registre des incidents sécurité

---

## 14. PERFORMANCE ET RÉSILIENCE

### 14.1. Cache
- **F-PERF-001** : Mettre en cache Redis le menu (TTL 5 min)
- **F-PERF-002** : Mettre en cache Redis les stocks ERP (TTL 30s)
- **F-PERF-003** : Mettre en cache Redis l'état des tables (TTL 10s)
- **F-PERF-004** : Invalider cache manuellement si modification admin

### 14.2. Optimisation
- **F-PERF-005** : Compresser payloads HTTP (Gzip niveau 6)
- **F-PERF-006** : Utiliser connection pooling PostgreSQL (max 50 connexions)
- **F-PERF-007** : Indexer tables BDD (orders, audit_logs)
- **F-PERF-008** : Limiter taille uploads (photos plats < 2 MB)

### 14.3. Tolérance Pannes
- **F-PERF-009** : Implémenter circuit breaker sur appels ERP (Opossum)
- **F-PERF-010** : Retry automatique exponentiel (max 3 tentatives)
- **F-PERF-011** : Load balancer NGINX (2 backends API minimum)
- **F-PERF-012** : Réplication PostgreSQL (Master + Replica)

---

## RÉCAPITULATIF QUANTITATIF

| Domaine Fonctionnel | Nombre de Fonctionnalités | Évolution |
|:-------------------|:------------------------:|:----------|
| **Gestion Tables** | 15 | = |
| **Gestion Menu** | **17** | **+3** (recommandations vin) |
| **Gestion Commandes** | 16 | = |
| **Gestion Paiements** | **15** | **+1** (split bill) |
| **Conformité NF525** | 11 | = |
| **Authentification/RBAC** | 13 | = |
| **Mode Offline** | 10 | = |
| **Notifications WebSocket** | 6 | = |
| **Rapports/Statistiques** | 12 | = |
| **Intégration ERP** | 10 | = |
| **Conformité RGPD** | 9 | = |
| **Administration Système** | 11 | = |
| **Sécurité Réseau** | 9 | = |
| **Performance/Résilience** | 12 | = |
| **TOTAL** | **166 fonctionnalités** | **+4** |

---

## PRIORISATION PAR ITÉRATION

### MVP (IT1) - 39 fonctionnalités critiques
- Tables : F-TAB-001 à F-TAB-010
- Menu : F-MENU-001 à F-MENU-010 (inclut recommandations vin)
- Commandes : F-CMD-001 à F-CMD-011
- Paiements : F-PAY-001 à F-PAY-009 (inclut split bill + tickets NF525)

### IT2 (Sécurité/Conformité) - 43 fonctionnalités
- NF525 : Toutes (11)
- Auth/RBAC : Toutes (13)
- RGPD : Toutes (9)
- Sécurité Réseau : Toutes (9)
- Notifications : F-NOTIF-001 à F-NOTIF-003

### IT3 (Performance/Résilience) - 32 fonctionnalités
- Mode Offline : Toutes (10)
- Performance : Toutes (12)
- ERP : F-ERP-001 à F-ERP-010

### IT4 (Observabilité/Admin) - 23 fonctionnalités
- Rapports : Toutes (12)
- Admin Système : Toutes (11)

### V2 (Extensions Futures) - 31 fonctionnalités
- Réservations : F-TAB-011 à F-TAB-015
- Menu Admin : F-MENU-011 à F-MENU-018 (gestion plats + associations vin)
- Pourboires : F-PAY-013 à F-PAY-015
- Autres extensions (rappel cuisine, analyses avancées)

---

## CORRECTIONS ET AMÉLIORATIONS APPORTÉES

### ✅ Ajouts suite à relecture des consignes

**1. Recommandations Vin Automatiques** (+3 fonctionnalités)
- **Justification** : Consigne explicite : *"l'application doit proposer les associations plat-vin recommandé (une unique proposition)"*
- **Impact** : F-MENU-008 à F-MENU-010 (affichage suggestion, détails vin, ajout au panier)
- **Priorité** : MVP (IT1) - Fonctionnalité critique métier

**2. Split Bill (Paiement Individuel)** (+1 fonctionnalité)
- **Justification** : Consigne : *"chaque individu paye sa part individuellement"*
- **Impact** : F-PAY-003 (division addition par personne)
- **Priorité** : MVP (IT1) - Mode de paiement principal

**3. Contexte Restaurant** (ajout notes informatives)
- **Tables** : 12 tables 4 pers + 8 tables 6 pers (20 total)
- **Menu** : Quotidien « fait maison » (3 entrées, 2 plats, 5 desserts)
- **Service** : Au verre uniquement (vin/bière)

### 🔄 Ajustements Scope

**Moyens de Paiement** :
- ✅ Carte Bancaire
- ✅ Espèces
- ✅ Tickets Restaurant / Chèques-Déjeuner
- ❌ Chèques (explicitement exclus dans consigne)

**Réservations** :
- Maintenues en V2 (non mentionnées dans scope initial)
- Cohérent avec focus "gestion salle côté service"

### 📊 Impact Quantitatif

| Élément | Version Initiale | Version Corrigée | Évolution |
|:--------|:----------------:|:----------------:|:---------:|
| **Gestion Menu** | 14 | **17** | +3 |
| **Gestion Paiements** | 14 | **15** | +1 |
| **Total Fonctionnalités** | 162 | **166** | **+4** |
| **MVP (IT1)** | 35 | **39** | +4 |
| **V2** | 29 | **31** | +2 |

---

## PROCHAINES ÉTAPES C5

1. ✅ **Liste fonctionnalités** : Complétée et corrigée (166 fonctionnalités)
2. ⏳ **Diagramme UML** : Créer diagramme de cas d'utilisation (Use Case)
3. ⏳ **Technologies présélection** : Benchmarker alternatives
4. ⏳ **Sélection finale** : Justifier choix technologiques définitifs
