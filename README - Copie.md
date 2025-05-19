
-- Base de données Sonelgaz Digitalisation
CREATE DATABASE IF NOT EXISTS sonelgaz_db;
USE sonelgaz_db;

-- TABLES
CREATE TABLE releve_elec (
    id_releve INT AUTO_INCREMENT PRIMARY KEY,
    id_client INT,
    date_releve DATE,
    ancien_index INT,
    nouvel_index INT,
    consommation INT
);

CREATE TABLE releve_gaz (
    id_releve INT AUTO_INCREMENT PRIMARY KEY,
    id_client INT,
    date_releve DATE,
    ancien_index INT,
    nouvel_index INT,
    consommation INT
);

CREATE TABLE intervention_elec (
    ID_intervention INT AUTO_INCREMENT PRIMARY KEY,
    id_client INT,
    date_intervention DATE,
    description TEXT
);

-- LOG
CREATE TABLE log_interventions (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    id_intervention INT,
    date_action DATETIME,
    user_action VARCHAR(50),
    action_type VARCHAR(50)
);

-- UTILISATEURS ET PRIVILÈGES
-- Note : Ces commandes doivent être exécutées avec les droits SUPER dans MySQL
-- CREATE USER 'admin_sonelgaz'@'localhost' IDENTIFIED BY 'admin2025!';
-- CREATE USER 'agent_releve'@'localhost' IDENTIFIED BY 'releve123';
-- GRANT ALL PRIVILEGES ON *.* TO 'admin_sonelgaz'@'localhost' WITH GRANT OPTION;
-- GRANT SELECT, INSERT, UPDATE ON sonelgaz_db.releve_elec TO 'agent_releve'@'localhost';
-- GRANT SELECT, INSERT, UPDATE ON sonelgaz_db.releve_gaz TO 'agent_releve'@'localhost';
-- FLUSH PRIVILEGES;

-- TRIGGERS

-- Calcul consommation électrique
DELIMITER $$
CREATE TRIGGER calcul_consommation_elec
BEFORE INSERT ON releve_elec
FOR EACH ROW
BEGIN
    SET NEW.consommation = NEW.nouvel_index - NEW.ancien_index;
END$$
DELIMITER ;

-- Vérification des index électrique
DELIMITER $$
CREATE TRIGGER verif_index_elec
BEFORE INSERT ON releve_elec
FOR EACH ROW
BEGIN
    IF NEW.nouvel_index < NEW.ancien_index THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Erreur : Nouvel index < Ancien index !';
    END IF;
END$$
DELIMITER ;

-- Log automatique après insertion d'une intervention électrique
DELIMITER $$
CREATE TRIGGER log_new_intervention_elec
AFTER INSERT ON intervention_elec
FOR EACH ROW
BEGIN
    INSERT INTO log_interventions (id_intervention, date_action, user_action, action_type)
    VALUES (NEW.ID_intervention, NOW(), CURRENT_USER(), 'INSERT');
END$$
DELIMITER ;

![MCD[Uploading sonelgaz_mcd.sql…]()
](https://github.com/user-attachments/assets/75103d5b-5e63-4229-9c24-42b0ec453b5b)


# Database-DBA-Project
Projet de gestion de base de données incluant la modélisation (MCD/MLD), la création de tables SQL, des requêtes complexes, la gestion des droits utilisateurs, l’optimisation des performances et des scripts de sauvegarde/restauration. Réalisé avec MySQL/PostgreSQL.


🗄️ Projet Base de Données / DBA
Ce projet met en œuvre les compétences fondamentales en conception, gestion et optimisation de bases de données relationnelles, dans un cadre professionnel simulé. Il s’adresse aux étudiants, développeurs ou administrateurs de bases de données souhaitant comprendre ou démontrer le cycle complet de vie d’une base de données.

🎯 Objectifs du projet![Uploading MCD.jpeg…]()

Concevoir un schéma relationnel optimal (modèle conceptuel, logique, physique).

Implémenter la base de données à l’aide de SQL (MySQL / PostgreSQL / Oracle).

Réaliser des requêtes complexes (jointures, agrégats, sous-requêtes).

Gérer la sécurité et les droits d’accès.

Optimiser les performances via les index, vues matérialisées, ou la normalisation.

Assurer la maintenance, la sauvegarde et la restauration de la base de données.

📂 Contenu du repository
📐 Conception : diagrammes E/A, MCD, MLD, MPD

🛠️ Scripts SQL : création de tables, insertion de données, requêtes

📊 Requêtes d’analyse : rapports, vues, fonctions stockées

🔐 Sécurité : gestion des utilisateurs et des rôles

📦 Sauvegarde & restauration : scripts de dump, fichiers de backup

📝 Documentation : manuel d’utilisation, justification des choix techniques

🧰 Technologies utilisées
SGBD : MySQL, PostgreSQL ou Oracle

Outils de modélisation : DBDesigner, MySQL Workbench, ou draw.io

Langages : Transact SQL


🔧 1. Création des Tables (avec commentaires)

-- Table: Abonné
CREATE TABLE abonne (
    ref_abonne INT PRIMARY KEY,
    nom_abn VARCHAR(100),
    prenom_abn VARCHAR(100),
    num_tel_abn VARCHAR(15),
    carte_idn VARCHAR(20),
    rue_abn VARCHAR(100),
    commune_abn VARCHAR(100),
    wilaya_abn VARCHAR(100),
    code_postal VARCHAR(10)
);

-- Table: Compteur Electricité
CREATE TABLE compteur_elec (
    ref_compteur INT PRIMARY KEY,
    Tarif VARCHAR(50),
    nbr_fil INT,
    type_compteur VARCHAR(50),
    lib_act_eco VARCHAR(100),
    annee_fab INT
);

-- Table: Compteur Gaz
CREATE TABLE compteur_gaz (
    ref_compteur INT PRIMARY KEY,
    Tarif VARCHAR(50),
    annee_fab INT,
    lib_act_eco VARCHAR(100)
);

-- Table: Géolocalisation
CREATE TABLE geolocalisation (
    id_geo INT PRIMARY KEY AUTO_INCREMENT,
    groupe VARCHAR(50),
    tournee VARCHAR(50),
    circuit VARCHAR(50)
);

-- Table: Relève Electricité
CREATE TABLE releve_elec (
    id_releve_elec INT PRIMARY KEY AUTO_INCREMENT,
    cycl VARCHAR(10),
    date_releve DATE,
    ancien_index INT,
    nouvel_index INT,
    ref_compteur INT,
    FOREIGN KEY (ref_compteur) REFERENCES compteur_elec(ref_compteur)
);

-- Table: Relève Gaz
CREATE TABLE releve_gaz (
    id_releve_gaz INT PRIMARY KEY AUTO_INCREMENT,
    cycl VARCHAR(10),
    date_releve DATE,
    ancien_index INT,
    nouvel_index INT,
    ref_compteur INT,
    FOREIGN KEY (ref_compteur) REFERENCES compteur_gaz(ref_compteur)
);

-- Table: Anomalie
CREATE TABLE anomalie (
    id_anomalie INT PRIMARY KEY,
    lib_anomalie VARCHAR(100)
);

-- Table: Agent
CREATE TABLE agent (
    ID_agent INT PRIMARY KEY,
    nom_agent VARCHAR(100),
    prenom_agent VARCHAR(100),
    rue_agent VARCHAR(100),
    commune_agent VARCHAR(100),
    wilaya_agent VARCHAR(100),
    num_tel_agent VARCHAR(15),
    sex_agent CHAR(1),
    situation_fami_agent VARCHAR(50)
);

-- Table: Intervention Electricité
CREATE TABLE intervention_elec (
    ID_intervention INT PRIMARY KEY,
    date_inter DATE
);

-- Table: Intervention Gaz
CREATE TABLE intervention_gaz (
    ID_intervention INT PRIMARY KEY,
    date_inter DATE
);

-- Table: Agence
CREATE TABLE agence (
    id_agence INT PRIMARY KEY,
    agence VARCHAR(100),
    adresse_agence VARCHAR(150)
);

-- Table: Traitement
CREATE TABLE traitement (
    ID_traitement INT PRIMARY KEY,
    traitement VARCHAR(150)
);



🔄 2. Requêtes d'insertion (exemples simples)
-- Ajout d'un abonné
INSERT INTO abonne VALUES
(1, 'Bensalah', 'Yacine', '0555123456', '12345678', 'Rue Khemisti', 'Oran', 'Oran', '31000');

-- Ajout d’un compteur électrique
INSERT INTO compteur_elec VALUES
(1001, 'B1', 4, 'Triphasé', 'Maison', 2019);

-- Ajout d’un agent
INSERT INTO agent VALUES
(10, 'Cherif', 'Hakim', 'Rue de la Liberté', 'Alger', 'Alger', '0555789564', 'M', 'Célibataire');


🔍 3. Requête SQL complexe (jointures + concaténation)
SELECT 
    CONCAT(a.nom_abn, ' ', a.prenom_abn) AS Nom_Complet,
    c.ref_compteur,
    c.type_compteur,
    g.groupe,
    g.tournee,
    g.circuit,
    r.date_releve,
    r.ancien_index,
    r.nouvel_index
FROM abonne a
JOIN possede p ON a.ref_abonne = p.ref_abonne
JOIN compteur_elec c ON p.ref_compteur = c.ref_compteur
JOIN se_situe s ON c.ref_compteur = s.ref_compteur
JOIN geolocalisation g ON s.id_geo = g.id_geo
JOIN releve_elec r ON c.ref_compteur = r.ref_compteur
ORDER BY r.date_releve DESC
LIMIT 1;


✅ 1. Création des utilisateurs & rôles SQL
-- Création des utilisateurs
CREATE USER 'admin_sonelgaz'@'localhost' IDENTIFIED BY 'admin2025!';
CREATE USER 'agent_releve'@'localhost' IDENTIFIED BY 'releve123';

-- Donner tous les droits à l'administrateur
GRANT ALL PRIVILEGES ON *.* TO 'admin_sonelgaz'@'localhost' WITH GRANT OPTION;

-- Donner des droits limités à l’agent de relève
GRANT SELECT, INSERT, UPDATE ON releve_elec TO 'agent_releve'@'localhost';
GRANT SELECT, INSERT, UPDATE ON releve_gaz TO 'agent_releve'@'localhost';

-- Appliquer les modifications
FLUSH PRIVILEGES;


🔁 2. Triggers utiles (automatisations)
🧠 a. Calcul automatique de consommation

-- D'abord, ajouter un champ "consommation" dans la table
ALTER TABLE releve_elec ADD consommation INT;

-- Création du trigger
DELIMITER $$
CREATE TRIGGER calcul_consommation_elec
BEFORE INSERT ON releve_elec
FOR EACH ROW
BEGIN
    SET NEW.consommation = NEW.nouvel_index - NEW.ancien_index;
END$$
DELIMITER ;

⚠️ b. Empêcher saisie d’un nouvel index < ancien index

DELIMITER $$
CREATE TRIGGER verif_index_elec
BEFORE INSERT ON releve_elec
FOR EACH ROW
BEGIN
    IF NEW.nouvel_index < NEW.ancien_index THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Erreur : Nouvel index < Ancien index !';
    END IF;
END$$
DELIMITER ;


📆 c. Log d’activité (journalisation)

CREATE TABLE log_interventions (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    id_intervention INT,
    date_action DATETIME,
    user_action VARCHAR(50),
    action_type VARCHAR(50)
);


// Et un trigger :

DELIMITER $$
CREATE TRIGGER log_new_intervention_elec
AFTER INSERT ON intervention_elec
FOR EACH ROW
BEGIN
    INSERT INTO log_interventions (id_intervention, date_action, user_action, action_type)
    VALUES (NEW.ID_intervention, NOW(), CURRENT_USER(), 'INSERT');
END$$
DELIMITER ;
