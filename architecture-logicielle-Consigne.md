# ARCHITECTURE LOGICIELLE

**Module :** M1-XDMFS-01-BX-M1-DEV  
**Année :** 2025-2026  
**Lieu :** Bordeaux  
**Intervenant :** Arnaud CUEILLE

## LIVRABLE

Au titre du suivi continu pour ce cours, le rendu du dossier aura lieu le **XX février 2026 à XXHXX** (vérification des contenus et de la présence de tous les dossiers).

Il est constitué d’un unique document au format PDF qui comporte votre nom et prénom suivi du code de la classe.

Il faudra transférer ce document via l’email de l’école à : `arnaud.cueille@intervenants.efrei.net`.

Le contenu du livrable à constituer se trouve indiqué au point **C9** en fin de document, tout en intégrant le contenu complet de ce document.

---

## CAS CONCRET

**Titre :** « Gestion d’un restaurant, salle et cuisine »

**Sujet :** Etude d’une application logicielle destinée à gérer un restaurant, côté salle et côté cuisine.

### Détails

Etant donné que le restaurant vient d’être racheté, le nouveau propriétaire ne désire pas acquérir du matériel dédié à cette application excepté **trois téléphones portables Androïd dernier modèle** d’une marque mondialement connue, à destination des trois serveurs.

Le cuisinier en chef n’a pas désiré que l’application à développer intègre la gestion des cuisines puisqu’il dispose d’une licence logicielle de gestion des stocks et des plats réalisables selon les stocks, qui prend en charge cette partie (ERP nommé **« QuiCuisineIci »**).

Le système de caisse devra aussi être compatible avec le système d’information à constituer, et le choix d’Androïd s’impose. Cette partie doit être développée en parallèle avec les applicatifs pour téléphone.

Le restaurant dispose maintenant de la fibre optique, et un réseau wifi est constitué de deux bornes en salle et une troisième en cuisine. Cela permet de passer les commandes via un service associé à l’ERP « QuiCuisineIci » et de recevoir les notifications sur le téléphone du serveur qui gère la table correspondante.

Les trois serveurs disposent de leur propre zone en salle et n’interviennent jamais sur les zones des collègues.

Le responsable de salle prend en charge le placement des clients, sélectionne la table la plus adaptée, distribue les cartes, et lors de cette étape, il présente le serveur qui prendra en charge la commande.

Le restaurant dispose d’un bar non actif à ce jour. À cet emplacement, il existe un unique terminal de paiement placé au niveau de la caisse enregistreuse développée pour l’occasion. Les téléphones ne disposent pas de cette fonctionnalité pour le moment.

Les serveurs, via l’application sur le téléphone, connaissent le nombre de plats encore disponibles à la vente en temps réel. Comme le restaurant n’a pas encore recruté de responsable de la cave à vin, c’est l’application qui doit proposer les associations plat-vin recommandé (une unique proposition).

Les processus de ravitaillement, de gestion de la propreté de la salle et des tables, de la plonge ne sont pas traités ici.

### Position
Vous prenez la place du prestataire à qui on a proposé d’analyser le besoin et de proposer une solution logicielle.

### Structure des données
Les données sont centralisées au sein d’une unique entité communément appelée base de données.

### Caractéristiques du logiciel
Le logiciel à concevoir doit donc prendre en charge uniquement les flux et leurs caractéristiques côté salle. Il ne doit pas être traité les caractéristiques de développement pour la partie cuisine, même s’il est nécessaire de considérer l’utilité de la connexion à l’ERP gérée par le chef cuisinier.

### Caractéristiques de la salle
La salle est constituée de **12 tables qui peuvent accueillir jusqu’à 4 personnes**, et **8 tables de maximum 6 personnes**. Il n’y a pas de rapprochement de table possible, ni d’extension de la salle, ni d’accueil d’une nouvelle personne sur un table saturée.

### Caractéristiques de la carte
La carte se veut moderne et constituée uniquement de produits très frais, pour une cuisine dite « fait maison ». Ainsi, il n’est préparé, par repas, que :
*   3 entrées différentes
*   2 plats distincts
*   5 desserts différents

La carte évolue selon la saison, tous les jours. L’apéritif est possible, et bien entendu facultatif, comme l’entrée, et le café ou le thé. Le vin ou la bière est servi au verre uniquement.

La formule minimale est donc innovante en proposant systématiquement le plat principal et le dessert, espérant attirer et fidéliser les clients gourmands du quartier.

### Caractéristiques relatives à la facturation
Lors du paiement, chaque individu paye sa part individuellement, il n’y a pas de partage d’assiette ou de bouteille puisque le service est au verre, mais une personne peut régler plusieurs commandes d’une même table, voire sa table intégralement.

### Caractéristiques relatives au paiement
Les cartes de crédit sont acceptées, ainsi que les chèques restaurant, mais pas les chèques.

### Interfaces utilisateurs
Il est identifié deux types d’utilisateurs, les services sur mobile, et le caissier sur ordinateur, par conséquent deux interfaces différentes. Il serait judicieux de présenter ces interfaces sous la forme de maquettes.

---

## Relation « blocs de compétences / travaux demandés »

### C1. Mettre en place une veille technologique, normative, et législative
*   Quelle(s) norme(s) permet(tent) de mettre en conformité le logiciel d’encaissement ?
*   Quel autre système normatif doit être envisagé ici, si cela est nécessaire ?

### C2. Analyser les besoins des utilisateurs et des parties prenantes
*   Lister les parties prenantes de l’entité, tant internes qu’externes.
*   Reprendre le besoin client si cela le nécessite (contradiction dans la demande, oubli ou manque évident, ...).

### C3. Étudier et évaluer la faisabilité du projet
*   Le projet comporte des spécificités qui peuvent paraître compliquées dans leur intégration. Existe-t-il des éléments parmi le besoin qui semblent infaisables ? Si oui, lesquelles ?

### C4. Concevoir l’architecture logicielle
*   Montrer itération après itération le cheminement qui vous permet de justifier la constitution d’une architecture fiable qui répond au besoin formulé initialement.
*   Quels critères vous permet de justifier cette architecture ?

### C5. Sélectionner les technologies et outils les plus adaptés
*   Après avoir listé les fonctionnalités de l’application logicielle, les présenter à l’aide d’un diagramme UML qui correspond le mieux.
*   Quelles sont les technologies actuelles qui répondent à ce besoin ? Justifier cette présélection.
*   Parmi ces technologies, quelle serait celle la mieux adaptée à votre avis ? Justifier cette sélection.

### C6. Présenter et valider l’architecture logicielle
*   Détailler chaque partie de l’architecture et répondre en quoi les critères de qualités sont validés, et si des dilemmes sont identifiés, préciser la raison de ce choix.

### C7. Créer des diagrammes et modèles de données et de flux
*   Représenter au moins un diagramme de séquence, prioritairement celui qui modélise un scénario, idéalement, un scénario majeur.
*   Représenter un modèle conceptuel de données (MCD) de la partie qui a pour gestion la salle.
*   Représenter à l’aide d’un diagramme adapté les relations ou interactions nécessaires avec l’environnement immédiat de l’application à concevoir.

### C8. Intégrer les pratiques de Test Driven Development (TDD)
*   Etant donné que vous avez opté pour une conception représentée avec un minimum de modularité, donner la démarche TDD déployée. Est-elle adaptée à cette conception ?

### C9. Collaborer activement à la rédaction du cahier des charges
*   Les éléments imposés ci-dessus doivent être répartis judicieusement au sein des parties judicieusement sélectionnées dans le cahier des charges à constituer.