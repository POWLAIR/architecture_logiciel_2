# C1 - Veille Technologique, Normative et Législative

## 1. Mise en conformité du logiciel d’encaissement

Dans le cadre du développement d'un logiciel de caisse pour un restaurant en France, la législation est stricte afin de lutter contre la fraude à la TVA.

### A. La Loi Anti-Fraude à la TVA (Article 88 de la Loi de Finances 2016)
Depuis le **1er janvier 2018**, tout assujetti à la TVA (dont les restaurants) qui enregistre les règlements de ses clients au moyen d'un logiciel de caisse doit utiliser un système sécurisé certifié.

Le logiciel doit garantir 4 conditions fondamentales (**ISCA**) :
*   **I - Inaltérabilité** : Impossibilité de modifier ou supprimer une transaction sans laisser de trace (journal technique).
*   **S - Sécurisation** : Sécurisation des données de règlement, de la conservation et de l'archivage.
*   **C - Conservation** : Obligation de conserver les données (clôtures journalières, mensuelles, annuelles) pendant une durée légale (souvent 6 ans fiscalement).
*   **A - Archivage** : Capacité à figer les données et produire une archive lisible par l'administration fiscale.

### B. Les Certifications Reconnues (NF525 vs LNE)
Pour prouver cette conformité, le logiciel doit être certifié par un organisme tiers accrédité. Deux certifications dominent le marché :

1.  **Certification NF525 (AFNOR)** :
    *   C'est la norme la plus courante pour les logiciels d'encaissement.
    *   Elle impose des règles strictes sur la gestion des tickets, les totaux, les clôtures (Z de caisse) et la signature numérique des transactions.
    *   *Recommandation pour le projet* : Viser la compatibilité **NF525**, qui est une référence rassurante pour les restaurateurs.

2.  **Certification LNE (Laboratoire National de Métrologie et d'Essais)** :
    *   Alternative à la NF525, elle valide également le respect des conditions ISCA sous le référentiel "Systèmes de caisse".

### C. Évolution 2024-2026 (Nouveauté Loi de Finances 2025)
*Note importante pour la veille actuelle :*
La Loi de Finances 2025 durcit les règles. À partir de **2026** (transition dès fin 2025), les **auto-attestations** par les éditeurs ne suffiront plus. Le logiciel devra obligatoirement détenir un certificat délivré par un organisme tiers (AFNOR ou LNE).
=> **Impact projet** : L'architecture doit être conçue dès le départ pour être "auditable" et certifiable (logs scellés, chaînage des transactions).

---

## 2. Autres systèmes normatifs à envisager

### A. RGPD (Règlement Général sur la Protection des Données)
Le système gère des données personnelles via :
*   La **réservation** (Nom, Téléphone, Email).
*   Le **programme de fidélité** éventuel (analyse des habitudes de consommation).
*   Le **paiement** (bien que les données bancaires soient souvent gérées par le TPE, le lien caisse-TPE doit être sécurisé).

**Actions requises :**
*   Recueil du consentement ("Opt-in") pour conserver les données client.
*   Droit à l'oubli (suppression des données client sur demande).
*   Sécurisation des bases de données.

### B. Normes Sanitaires (HACCP) - Lien Indirect
Bien que le logiciel ne cuisine pas, il participe à la traçabilité (HACCP) :
*   L'archivage des commandes peut servir de preuve en cas de contrôle sanitaire (quel client a mangé quel plat à quelle date).
*   Lien avec l'ERP "QuiCuisineIci" pour la traçabilité des lots (non traité dans ce module mais à prévoir en interface).

### C. Normes d'Accessibilité (RGAA)
*   Si l'application mobile serveur ou l'interface caisse est utilisée par du personnel en situation de handicap, elle doit respecter des critères d'accessibilité (contrastes, compatibilité lecteurs d'écran).

---

## Résumé pour le dossier (Livrable)
Pour le module C1, nous retiendrons principalement :
1.  La **certification NF525** (ou équivalent LNE) imposée par la **Loi Anti-Fraude TVA** pour l'encaissement.
2.  Le **RGPD** pour la gestion des clients et des réservations.
