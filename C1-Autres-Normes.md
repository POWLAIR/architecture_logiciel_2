# C1 - Autres systèmes normatifs à envisager

En complément des normes fiscales (NF525/LNE) et de la protection des données (RGPD), l’analyse du contexte "Restaurant Connecté" révèle quatre autres cadres réglementaires majeurs impactant l'architecture.

## 1. Sécurité des Paiements Bancaires : Norme PCI DSS
*Contexte : Le restaurant accepte les cartes bancaires via un TPE relié à la caisse.*

La norme **PCI DSS** (Payment Card Industry Data Security Standard) s'impose à toute entité traitant des données cartes. Même si le restaurant utilise des TPE externes, le logiciel de caisse (POS) entrant en interaction avec la transaction est soumis à des contraintes strictes.

### Impacts Logiciels & Architecture :
*   **Interdiction stricte de stockage** : Le logiciel ne doit **JAMAIS** stocker, même crypté, le cryptogramme visuel (CVV2) ou le code PIN.
*   **Masquage du PAN** : Si le numéro de carte (PAN) apparaît sur une facture ou un log, il doit être tronqué (ex: `**** **** **** 1234`).
*   **Sécurisation des flux** : La communication Caisse <-> TPE doit être chiffrée (TLS 1.2+).
*   **Segmentation Réseau (VLAN)** : Isolement obligatoire du réseau monétique (CDE - Cardholder Data Environment) vis-à-vis du Wifi "Serveurs" et du Wifi "Invités".

*En cas de faille de sécurité, le restaurant serait tenu responsable s'il n'est pas "PCI DSS Compliant".*

## 2. Législation sur l'Alcool : Loi Evin
*Contexte : L'application mobile propose une fonctionnalité de suggestion "Accord Plat-Vin".*

Cette fonctionnalité d'aide à la vente tombe sous le coup de la **Loi Evin** (1991) et du Code de la Santé Publique encadrant la promotion de l'alcool.

### Impacts sur la fonctionnalité "Conseil Vin" :
*   **Objectivité de l'Algorithme** : La recommandation doit se fonder sur des critères organoleptiques techniques (cépage, origine, accord gustatif) et non sur des incitations à la consommation ("Fêtez ça avec...", "À boire sans soif").
*   **Affichage Légal** : Chaque écran proposant une boisson alcoolisée doit afficher de manière lisible et contrastée la mention sanitaire :
    > *"L'abus d'alcool est dangereux pour la santé, à consommer avec modération."*
*   **Non-incitation automatique** : L'outil ne doit pas suggérer systématiquement une quantité supérieure (bouteille) si le client demande un verre.

## 3. Information Consommateur : Règlement INCO (Allergènes)
*Contexte : Gestion de la Carte et des "Restrictions Nutritionnelles" (cités dans les cas d'utilisation).*

Le Règlement Européen **INCO** (1169/2011) rend obligatoire l'information sur la présence de **14 allergènes majeurs** (Gluten, Arachides, Fruits à coque, etc.) pour les denrées non-emballées (plats servis au restaurant).

### Impacts Base de Données & UI :
*   **Modélisation** : La BDD doit impérativement lier chaque `Plat` à une liste d'`Allergènes`.
*   **Interface Serveur** : L'application mobile doit permettre au serveur de filtrer la carte ou de consulter instantanément les allergènes d'un plat pour répondre au client.
*   **Mise à jour** : Si la recette change (Chef), les allergènes doivent être mis à jour en temps réel sur les terminaux.

## 4. Législation Internet / Wifi : LCEN & Anti-Terrorisme
*Contexte : Mise à disposition de bornes Wifi pour les serveurs et l'ERP.*

Le restaurant, en fournissant un accès réseau (même à ses salariés), est soumis aux obligations des Opérateurs de Communications Électroniques (Loi pour la Confiance dans l'Économie Numérique - LCEN et Décret 2006-358).

### Impacts Architecture Réseau :
*   **Conservation des Logs (1 an)** : Obligation de stocker les données de trafic (Qui se connecte ? Quelle MAC Address ? Quelle IP ? Quand ?) pendant 12 mois.
*   **Identification des terminaux** : Le système doit pouvoir dire quel serveur (tablette) a effectué quelle requête via le Wifi.
*   **Sécurité** : Nécessité d'un portail captif ou d'une authentification WPA2/WPA3 Enterprise pour lier chaque terminal à une personne physique identifiée.

---

## Synthèse Architecture (Impact Normatif Global)

| Norme | Domaine | Composant Impacté | Contrainte Technique Majeure |
| :--- | :--- | :--- | :--- |
| **PCI DSS** | Bancaire | Réseau / TPE | Segmentation VLAN + Pas de stockage PAN/CVV |
| **Loi Evin** | Santé | Mobile (UX/UI) | Mention sanitaire + Algo objectif |
| **INCO** | Santé | BDD / Mobile | Champs allergènes obligatoires + Affichage |
| **LCEN** | Légal | Routeur / Wifi | Stockage des logs de connexion (1 an) |
