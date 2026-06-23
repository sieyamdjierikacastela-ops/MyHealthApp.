# MyHealth Pro — Spécifications Techniques et Documentation d'Architecture

> **Slogan :** La santé, partout, pour tous...
> **Statut du Document :** Version 1.0.0 — Validée pour la Phase 2
> **Accès :** Restreint aux membres de l'équipe de développement MyHealth Pro. Tout usage externe est strictement interdit.

---

## 1. 📝 Vision Globale du Projet & Alignement Business

**MyHealth Pro** est un écosystème numérique de télémédecine et de gestion de données de santé conçu spécifiquement pour répondre aux enjeux de désertification médicale et d'accès aux soins en Afrique (avec un ancrage initial au Cameroun). La plateforme structure ses services autour de trois applications distinctes interconnectées :

### 👥 Profil 1 : L'Espace Patient (Application Mobile)
*   **Axe Majeur :** Le Carnet de Santé Numérique Universel.
*   **Parcours Obligatoire :** Remplissage d'un questionnaire de santé initial complet à l'inscription pour établir le profil de risque épidémiologique.
*   **Fonctionnalités Clés :** Recherche de praticiens parmi 87 spécialités, prise de rendez-vous en ligne (consultation en cabinet / téléconsultation / visite à domicile), paiement mobile intégré (Orange Money, MTN Mobile Money), téléconsultation par visioconférence sécurisée, messagerie asynchrone cryptée avec le médecin traitant, et notation/évaluation anonymisée des praticiens.

### 🩺 Profil 2 : Le Portail Médecin (Application Web)
*   **Axe Majeur :** L'Espace de Pratique Clinique et de Télémédecine.
*   **Vérification Réglementaire :** Inscription soumise à la validation stricte du numéro d'ordre national (Format attendu : `CM-YYYY-XXXX`).
*   **Fonctionnalités Clés :** Tableau de bord d'activité quotidienne, gestionnaire d'agenda dynamique avec création automatique de créneaux de consultation ajustables (durée par défaut : 30 minutes), accès au dossier médical partagé (DMP) des patients ayant autorisé l'accès, module de prescription électronique avec génération d'ordonnances sécurisées (PDF avec signature numérique), et édition automatique des feuilles de soins.

### ⚙️ Profil 3 : Le Panneau d'Administration (Dashboard Web)
*   **Axe Majeur :** Modération, Sécurité et Gestion Financière.
*   **Fonctionnalités Clés :** Interface de KYC (Know Your Customer) pour la validation des diplômes et numéros d'ordre des médecins, système de modération des commentaires et des évaluations laissés par les patients, gestion de la facturation et rétrocession des honoraires médicaux (prélèvement de la commission plateforme), suivi de la disponibilité du serveur et des indicateurs de performance (KPI) globaux.

---

## 🏗️ 2. Architecture Technique Extrêmement Détaillée

L'écosystème repose sur une architecture découplée orientée API (REST & WebSockets) permettant l'indépendance technologique des interfaces clientes.

### 🗄️ Couche Base de Données (PostgreSQL)
*   **Choix Technologique :** PostgreSQL (v14+), retenu pour sa gestion native de l'intégrité référentielle, sa robustesse sur les types de données géographiques (PostGIS pour les visites à domicile à terme) et sa vélocité de traitement des données structurées/semi-structurées.
*   **Gestion des Données Flexibles (JSONB) :** Utilisé pour stocker les structures de données variables sans altérer le schéma relationnel (Exemples : les diplômes d'un médecin `education`, ses certifications `certifications`, et sa grille horaire hebdomadaire mouvante `working_hours`).
*   **Optimisation des Performances :** Indexation systématique par B-Tree sur les clés étrangères et les colonnes fréquemment requêtées (villes, spécialités, statuts) afin de garantir un temps de réponse de la base de données inférieur à 50ms sur les lectures.

### ⚙️ Couche Backend & API (Node.js + Express + TypeScript)
*   **Runtime & Langage :** Node.js configuré exclusivement avec TypeScript. Le typage strict (`strict: true` dans le `tsconfig.json`) élimine à la compilation les erreurs d'incohérence de données médicales.
*   **Framework :** Express.js géré en architecture en couches (Routes ➡️ Contrôleurs ➡️ Services ➡️ Modèles/Triggers BDD).
*   **Sécurité et Cryptographie :** 
    *   Hachage irréversible des mots de passe des utilisateurs via l'algorithme `bcrypt` (facteur de coût : 12).
    *   Authentification sans état (Stateless) basée sur des jetons JSON Web Tokens (JWT) signés avec l'algorithme `HS256`, stockés côté client dans des cookies sécurisés `HttpOnly`, `Secure` et `SameSite=Strict`.
    *   Protection contre les attaques de force brute via un middleware de *Rate Limiting*.

### 📱 Couche Frontend & Interfaces Utilisateurs
*   **Web (Médecins & Admin) :** React.js + TypeScript alimenté par TailwindCSS pour des interfaces denses, réactives et accessibles sur les navigateurs de bureau.
*   **Mobile (Patients) :** Flutter (Dart) afin de distribuer une seule et unique base de code hautement performante sur iOS et Android, garantissant une fluidité parfaite à 60 FPS lors de la navigation dans les carnets de santé.

---

## 🗺️ 3. Feuille de Route Chronologique Intégrale (Phases du Projet)

Le projet observe une méthodologie de développement incrémentale stricte : **La structure des données dicte la logique de programmation, qui dicte l'expérience visuelle.**

[Phase 0: Legal] ➔ [Phase 2: BDD] ➔ [Phase 3: Backend] ➔ [Phase 4: Design UX] ➔ [Phase 5: Frontend]


### 🗄️ PHASE 2 : Conception, Modélisation et Déploiement de la Base de Données (Étape Actuelle)
*   **Objectif :** Établir la fondation immuable du stockage de l'information.
*   **Livrables :** Scripts SQL d'initialisation des extensions, création des types énumérés (`ENUM`), création des schémas de tables avec clés primaires/étrangères, indexations et mise en place des fonctions triggers PL/pgSQL.
*   **Échéance :** Semaines 1 à 4.

### ⚙️ PHASE 3 : Développement des Fondations Backend & API REST
*   **Objectif :** Rendre la base de données accessible via des points de terminaison (Endpoints) sécurisés.
*   **Tâches Précises :**
    1.  Mise en place de l'arborescence du projet Node.js/TypeScript et connexion au pool PostgreSQL via le pilote `pg`.
    2.  Développement des routes d'authentification (`/api/auth/register` et `/api/auth/login`) avec implémentation de `bcrypt` et génération des payloads JWT.
    3.  Développement des contrôleurs CRUD pour la gestion du profil médecin (`/api/doctors`) et de ses disponibilités (`/api/slots`).
*   **Échéance :** Semaines 5 à 12.

### 🎨 PHASE 4 : Maquettage Graphique et validation UI/UX sur Figma
*   **Objectif :** Concevoir des interfaces ergonomiques adaptées à une utilisation rapide en milieu clinique et par le grand public.
*   **Tâches Précises :**
    1.  Création du *Design System* complet (typographies, icônes standardisées, palettes de couleurs).
    2.  Prototypage haute fidélité des parcours utilisateurs (Prise de rendez-vous patient, tableau de consultation médecin, validation de documents administrateur).
    3.  Tests d'utilisabilité sur un panel de 10 patients et 5 médecins pour validation définitive avant codage.
*   **Échéance :** Semaines 13 à 18.

### 💻 PHASE 5 : Développement des Applications Clientes & Intégration Finale
*   **Objectif :** Codage des interfaces visuelles et liaison finale avec les API de la Phase 3.
*   **Tâches Précises :**
    1.  Développement de la plateforme Web React pour les médecins et les administrateurs (Semaines 19 à 27).
    2.  Développement de l'application mobile Flutter pour les patients (Semaines 22 à 31).
    3.  Tests d'intégration de bout en bout (End-to-End), audits de sécurité sur la gestion des flux de santé, et correction des anomalies (Semaines 32 à 36).

---

## 🔒 4. Normes de Code, Flux Git et Gouvernance de l'Équipe

Afin d'assurer la maintenabilité du code au sein de notre groupe de développement, les règles suivantes s'appliquent immédiatement à tous les membres :

### 🛡️ Règle de Sécurité Absolue
*   **Aucun secret en ligne :** Il est strictement interdit de pousser des fichiers d'environnement `.env`, des mots de passe en clair, des clés d'API ou des certificats SSL sur les dépôts distants. 
*   Le fichier `.gitignore` doit être vérifié avant chaque commit pour inclure les dossiers `/node_modules`, `/dist` et les fichiers `.env`.

### 🔀 Gestion des Branches (Git Flow)
*   **`main` / `master` :** Branche sacrée. Représente le code stable en production. Aucun commit direct n'est toléré.
*   **`develop` :** Branche d'intégration des fonctionnalités validées.
*   **`feature/` :** Toute tâche ou sous-tâche du projet doit être développée sur une branche éphémère isolée (Exemple : `feature/auth-jwt-implementation` ou `feature/migration-002-doctors`).
*   **Processus de Fusion :** Pour fusionner une branche `feature/` vers `develop`, le développeur doit ouvrir une *Pull Request* (PR). Cette PR doit obligatoirement être relue, testée localement et validée par au moins **un autre membre de l'équipe** avant d'être fusionnée.

### 💾 Conventions d'Évolution de la Base de Données
*   Chaque modification ou ajout de table doit être consigné dans un fichier SQL de migration unique et incrémental (Exemple : `003_create_appointments_table.sql`).
*   Chaque script doit utiliser des structures de contrôle d'existence (`CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`) pour garantir que la réexécution des scripts sur une base de données existante ne génère pas de crash ou de doublons.

---

## 🚀 5. Guide de Déploiement Local de l'Environnement de Développement

### Étape 1 : Installation des Dépendances Système et Outils
Assurez-vous que votre machine dispose des versions minimales suivantes :
*   **Node.js :** Version stable LTS (v18.x.x ou v20.x.x requise).
*   **PostgreSQL :** Instance locale ou distante (v14, v15 ou v16) en cours d'exécution.

### Étape 2 : Récupération du Code Source
Ouvrez votre terminal et exécutez les commandes suivantes :
```bash
git clone [https://github.com/votre-compte/myhealth-pro.git](https://github.com/votre-compte/myhealth-pro.git)
cd myhealth-pro/backend
Étape 3 : Installation des Modules Node.js
Installez l'ensemble des paquets requis décrits dans le fichier package.json :

Bash
npm install
Étape 4 : Configuration des Variables d'Environnement Local
Créez un fichier physique nommé .env à la racine de votre dossier /backend :

Extrait de code
PORT=5000
NODE_ENV=development
DATABASE_URL=postgres://votre_utilisateur_pg:votre_mot_de_passe_pg@localhost:5432/myhealth_db
JWT_SECRET=CHAINE_DE_CARACTERES_COMPLEXE_ET_LONGUE_POUR_LA_SIGNATURE_DES_TOKENS_JWT_2026
(Remplacez votre_utilisateur_pg and votre_mot_de_passe_pg par vos identifiants réels de connexion PostgreSQL).

Étape 5 : Exécution des Scripts de Base de Données
Connectez-vous à votre interface de gestion PostgreSQL (pgAdmin, DBeaver ou via le terminal psql).

Créez une nouvelle base de données nommée myhealth_db.

Ouvrez un éditeur de requête SQL, puis exécutez dans l'ordre chronologique strict :

Le contenu du fichier 001_init_users.sql (Création de la table users).

Le contenu du fichier 002_doctor_profiles.sql (Création des types, des spécialités, des profils médecins et des triggers de notation).

Étape 6 : Lancement du Serveur de Développement
Exécutez la commande suivante pour démarrer l'application avec rechargement automatique à chaque modification de code (Hot Reload) :

Bash
npm run dev
Le terminal doit afficher la confirmation de connexion à la base de données et indiquer : [server]: Server is running at http://localhost:5000.

📞 6. Matrice de Gouvernance & Support Technique
👥 Répartition des Rôles au sein du Groupe de Développement
Chef de Projet / Garant du Cahier des Charges : [Insérer Nom]

Responsable Architecture de Base de Données (DBA) : [Insérer Nom]

Développeur Principal Backend (API Node.js) : [Insérer Nom]

Développeur Intégrateur Frontend (React/Flutter) : [Insérer Nom]

💰 Plan de Maintenance Applicative
Un audit de sécurité annuel de la base de données et du code source est budgétisé par la direction de MyHealth Pro pour garantir la mise à jour des dépendances critiques, la correction des failles de sécurité émergentes et l'optimisation des index PostgreSQL au fur et à mesure de la croissance du volume de patients.
