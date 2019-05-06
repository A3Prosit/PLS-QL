## Mots Clés

  

-   Contrainte d’intégrité *
    
-   Curseurs
    
-   DataSet .NET
    
-   Procédures
    
-   Fonction
    
-   Incohérences dans les données
    
-   Gestion des erreurs
    
-   Droit
    
-   Accès aux données
    
-   Ensemble d’enregistrement
    
-   Jeu de données
    

  

## Contexte

Quoi ?

Créer BDD sécurisée et ergonomique

Comment ?

Paramétrer les droits

En créant des procédures, des fonctions, curseurs

Gérer les erreurs et contraintes d’intégrité

Pourquoi ?

Réponde aux besoins

  

## Contraintes

  

Aucune contrainte

## Problématique

  

Comment créer des procédures et fonctions ?

Comment gérer les paquets et les exceptions ?

Comment gérer l’intégrité de la BDD ?

  

## Généralisation

  

Gestion de données

Automatisation

BDD

  

## Hypothèses

Peut faire procédure stockées avec PL/SQL

Pas besoin de curseur PL/SQL

On doit GRANT dans les procédures

Les contraintes d’intégrités répondent aux 3 FN

PL/SQL gère les exceptions

Try/catch avec PL/SQL

Contraintes d’intégrités = clés primaires/secondaires

  

## Plan d’Action

  

Etudes :

- Rappels BDD :

- Clés primaires

- Clés étrangères

- formes normales

- etc

- Contraintes d’intégrités

- Fonctions / procédures stockées

- curseurs / Dataset :

- explicite

- implicite

## Clés primaires / étrangères 

- Primaire : Contrainte d'unicité composée d'une ou plusieurs colonnes, qui permet d'identifier de manière unique chaque ligne de la table.
	- Contrainte d'unicité (index UNIQUE)
	- Composée d'une ou plusieurs colonnes (Les clés peuvent-êtres composites)
	- Permet d'identifier chaque ligne de manière unique (pas de NULL)

- Etrangères : Vérification de l'intégrité de la base. On place une **Contrainte** pour vérifier qu'une donnée que l'on rentre, correspond bien à un ID de l'autre côté.
	- Possibilité de compositve
	- Un index est automatiquement crée sur une clé étrangère
	- La clé étrangère doit faire référence à une primaire, respecter le même type...
	- "FOREING KEY" pour créer la clé, REFERENCES de l'autre côté.

## Formes normales 

Normaliser des modèles permet de vérifier la robustesse d'une conception et d'améliorer la modélisation, éviter la redondance {...}. On optimise ainsi le modèle relationnel.

- 6 niveaux de formes normales + deux complémentaires. Les trois premiers sont les plus importants.

**- 1FN** : Chaque attribut soit atomique.

![](https://image.noelshack.com/fichiers/2019/19/1/1557150259-capture.png)
![](https://image.noelshack.com/fichiers/2019/19/1/1557150339-capture2.png)

**- 2FN :** Ne concerne que les tables avec une clé primaire composite 
	- Être déjà en 1FN
	- "Aucun attribut ne faisant pas partie de la clé primaire ne doit dépendre que d'une partie de la clé primaire".

![](https://image.noelshack.com/fichiers/2019/19/1/1557150604-capture3.png)
==> Si on a une colonne qui est sur une table avec deux FK, mais qui ne se sert que d'une seule c'est faux .

- Il faut recréer une autre table pour y référencer la FK avec ce qui le concerne

**- 3FN :**  Comme la 2FN mais pour les attributs non clés
- Respecter la 2FN
- "Aucun attribut ne faisant pas partie de la clé primaire ne doit dépendre d'une partie des autres attributs ne faisant pas non plus partie de la clé primaire."
![](https://image.noelshack.com/fichiers/2019/19/1/1557152418-capture4.png)

## Fonctions et procédures stockées

- Procédure : Série instructions SQL désingée par un nom, enregistré dans la BDD. Contraitement aux requêtes préparées, qui ne sont gardées en mémire pour la session courante, les procédures stockées le sont de manières durables.
- Il faut prévoir un délimiteur avec un | 

``` SQL
DELIMITER | -- On change le délimiteur
CREATE PROCEDURE afficher_races()      
    -- toujours pas de paramètres, toujours des parenthèses
BEGIN
    SELECT id, nom, espece_id, prix
    FROM Race;  -- Cette fois, le ; ne nous embêtera pas
END|            -- Et on termine bien sûr la commande CREATE PROCEDURE par notre nouveau délimiteur


CALL afficher_races()|   -- le délimiteur est toujours | !!!
```

- On peut mettre des paramètres, il faut préciser le type :
	- IN : Valeur est fournie
	- OUT : La valeur sera établie lors de la procédure
	- INOUT : Valeur modifiée et renvoyée

```SQL
DELIMITER | -- Facultatif si votre délimiteur est toujours |
CREATE PROCEDURE afficher_race_selon_espece (IN p_espece_id INT)  
    -- Définition du paramètre p_espece_id
BEGIN
    SELECT id, nom, espece_id, prix 
    FROM Race
    WHERE espece_id = p_espece_id;  -- Utilisation du paramètre
END |
DELIMITER ;  -- On remet le délimiteur par défaut


CALL afficher_race_selon_espece(1);
SET @espece_id := 2;
CALL afficher_race_selon_espece(@espece_id);
```

- Delete :
``` SQL
DROP PROCEDURE afficher_races;
```

**Avantages** :
- Réduire allez-retour entre client/serveur. Une seule requête
- Sécuriser une BDD (choisir qui exécute quoi), finit les DELETE dangereux ou les UPDATE
- Traitement toujours de la même manière (un et un seul langage)

**Inconvénients** :
- Charge de données, moins de place pour stocker les données
- Diffère beaucoup d'un SGBD à un autre

## Curseur / Dataset 
Permet de parcourir une à une les résultats d'une requête :
### **Déclarations Explicite : (Assez lourd)**
1. On déclare le curseur (avec son SELECT)  `` CURSOR <<nomCurseur>> IS SELECT...``
2. On l'ouvre (on n'est pas obligé de l'ouvrir tout de suite) . ``OPEN <nom_du_curseur> ;``
3. On en récupère les lignes une à une, en commençant par la première  ``FETCH <nomCurseur>  into <var1> <var2>``
4. On le referme ``CLOSE <nom_du_curseur> ;``

Ex : Voici un script qui renvoie toutes les DORIS_KEY de la table BUDGET pour les budgets de l'exercice 2007 :
``` SQL
DECLARE  
  CURSOR mesbudgets IS  
   SELECT * FROM BUDGET  
   WHERE EXERCICE = '2007';  
  mon_budget BUDGET%ROWTYPE;  
BEGIN  
  OPEN mesbudgets;  
  LOOP  
    FETCH mesbudgets INTO mon_budget;  
    DBMS_OUTPUT.PUT_LINE(mon_budget.DORIS_KEY);  
    EXIT WHEN mesbudgets%NOTFOUND;  
  END LOOP;  
  CLOSE mesbudgets;  
END;
```

### **Déclarations implicite** :

-  Il existe une syntaxe qui permet de se passer des instructions OPEN, FETCH et CLOSE. On utilise toujours la boucle FOR :

```SQL
DECLARE
BEGIN
FOR curseur_EMPLOYE IN (SELECT * FROM EMP WHERE JOB = 'MANAGER')
LOOP

DBMS_OUTPUT.PUT_LINE('L''employé ' || curseur_EMPLOYE.ENAME ||
' a un salaire de ' || curseur_EMPLOYE.SAL ||
'. Le numéro de son Manager est ' || curseur_EMPLOYE.MGR);

END LOOP;
END;
```

### Attributs de curseur :

On peut récupérer des informations sur un curseur par l'intermédiaire des attributs suivants :  
- <nom_du_curseur>%ISOPEN : retourne TRUE ou FALSE.  
- <nom_du_curseur>%FOUND : retourne TRUE ou FALSE suivant que le dernier FETCH a retourné une ligne ou non.  
- <nom_du_curseur>%NOTFOUND : retourne TRUE ou FALSE à l'inverse de %FOUND  
- <nom_du_curseur>%ROWCOUNT : retourne le nombre de FETCH ayant renvoyé une ligne. Avant le premier FETCH, cet attribut renvoie 0.

### Updates 

On utilise "FOR UPDATE" pour faire des MAJ avec un curseur. Ceci verrouille les lignes de la BDD, le temps que la procédure s'éeffectue.
```SQL
DECLARE  
CURSOR moncurseur IS  
SELECT champ1,champ2 FROM table  
FOR UPDATE;
```

### UPDATE/Delete

On utilise "WHERE CURRENT OF <nom_du_curseur>" pour préciser que la MAJ ou la destruction  s'applique à la ligne en cours récupérée (pas besoin d'un Where).

``` SQL
DECLARE
 CURSOR moncurseur IS
 SELECT NUMERO FROM FASCICULE
 WHERE ABONNEMENT = 2500
 FOR UPDATE;
BEGIN
 FOR monfascicule IN moncurseur LOOP
 UPDATE FASCICULE
 SET NUMERO = NUMERO + 1
 WHERE CURRENT OF moncurseur;
 END LOOP;
COMMIT;
END;
```

### Passer des paramètres à un curseur 

```SQL

DECLARE  
CURSOR moncurseur (param table.champ3%TYPE := 10) IS // :=10 signifie que la valeur par défaut est 10
SELECT champ1,champ2 FROM table  
WHERE champ3 = param;

==> Utilisation : 
BEGIN  
OPEN moncurseur(valeur);
```

## PL/SQL

### Décomposition
PL/SQL est un langage qui intègre SQL et permet de programmer de manière procédurale.

Décomposé en bloc :
- DECLARE : Section déclarative
- BEGIN...END : Section éxécutable
- EXCEPTION : Traitement des exceptions

## Fonctions & cie
**Procédure** : Programme PL/SQL nommé, compilé et stocké dans la base.
**Fonction** : Procédure retournant une valeur.
**Package** : Module de programmes incluant procédures et / ou fonctions fonctionnellement dépendantes. Il est composé de deux parties :
- La spécification (introduite par CREATE PACKAGE), liste  des entêtes contenues dans le package
- Le corps du package (Introduit par CREATE PACKAGE BODY) qui  contient le code des procédures/fonctions

Entête : 
```
SQL> CREATE PACKAGE clients AS -- spécifications du package  
PROCEDURE insere_client (no INTEGER, nom VARCHAR2, ...);  
PROCEDURE supprime_client (no INTEGER);  
...  
END;
```
Body : 
```
SQL> CREATE PACKAGE BODY clients AS -- le corps du package  
PROCEDURE insere_client (no INTEGER, nom VARCHAR2, ...) IS  
BEGIN  
...  
INSERT INTO clients VALUES (no, nom, ...);  
END;
```

### Trigger

**Un trigger** est un morceau de PL/SQL stocké dans la base, que l'on peut déclencher lors d'évènements particuliers. Permet par exemple de synchroniser des tables : en mettant une à jour une table, un autre est mise à jour automatiquement.*

Ex :

```
Sur <évènement>
si <condition>
alors <action>
```
Nb : Les ordres du LDD et de la gestion de transaction (Create/alter/drop/commit/savepoint) sont interdits pour les triggers.

## Contraintes d'intégrité

Pour assurer la cohérence logique de la BDD, on va vérifier toutes les assertion.
- Un schéma d'une BDD relationnelle doit contenir des contraintes d'intégrité(CIs)
- Au cours de la vie de la BDD, il est adéquat d'en rajouter
- On les exprime à l'aide du SQL ou extension SQL
- Il en existe plusieurs types : 
	- Structurelle : Modèle relationnel / unicité
	- Comportementales : Liées aux applications (ex : moyenne salaire >=10 000)
	- Intra-Relationnelle : Une seule relation, non nullité d'un attribut
		- Domaine : (Valeur comprise entre X et Y)
		- Non nulle 
		- Unicité de clé : Pas de doublons
		- Dépendance de fonction : Une valeur sera utilisé dans une fonction
		- Temporelle : Ne peut que décroitre
		- Agregation : La moyenne ne doit dépasser X
	- Inter-Relationnelle : Plusieurs relations, intégrité référentielle
		- Référentielle : Notion de clé étrangère
		- Inclusion : Ensemble de valeur d'une colonne d'une relation est inclus dans l'ensemble des valeurs d'une colonne d'une autre relation
		- Genérale : La somme des quantités livrés <= quantité commandé
		

Il existe aussi les contraintes d'intégrité référentielle :
- Attribut ou groupe d'attribut d'une relation qui apparaît comme une clé pour une autre relation :

```JAVA
"Une récolte doit référencer un vin et un producteur existants"

VERIFICATION DE CETTE CI

**1/ Insertion d'une récolte**

**_-> le vin et le producteur doivent exister_**

**2/ Suppression d'un vin**

**_-> ce vin ne doit pas être récolté_**
```
**Exemples de CI :**

```
- Unique not nul
create table EMPLOYE (
NOM Char(20) unique not nul)

- primary key / check / references

create table EMPLOYE(
NOM(Char(20) primary key, 
SAL Integer check (SAL > 0)
DPT Char(20) references DEPART)

{...} DPT Char(20) references DEPART on delete cascade)

```

Il existe des SGBD avec un langage qui permet d'en faire simplement comme : 
INGRES : https://fr.wikipedia.org/wiki/Ingres_(base_de_donn%C3%A9es)

### Notion de transaction :

- Unité d’exécution atomique pour SGBD 
- Ensemble de requêtes SQL qui s'éxécute en ToR, vérifiant les CI avant de s'éxécuter. MAJ rejeté si une CI n'est pas vérifiée.

### Problèmes avec les CI :
- Redondance : Si l'on met deux règles, l'une peut écraser l'autre.

