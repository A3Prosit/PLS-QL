
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

# Rappels BDD :

- Clés primaires: l'id d'un enregistrement

- Clés étrangères: id d'une autre table

- formes normales: 5 règles 
- transaction:
ensemble e reuête s'exécutant en tout ou rien, permet de faire passer la base d'un état cohérent à un autre état cohérent
- trigger: action à exécutr sur la bdd quand une conition est vérifiée (update, delete, insert, etc) on [event] if [condition] DO [action]
- etc

# Contraintes d’intégrités
"assertion verifiée par la bdd à tout moment"
permettent d'assurer la cohérence de la bdd
2 mode de vérif: 
immédiates :elles sont vérif avant chaque requête d'update, si elles sont vérif  alors on update
différée: on fait l'update, on verif en fin de transaction et on rollback en cas de problème

différentes classifications:
- structurelles: liées au modèle relationnel. ex : unicité clé valeur
- comportementales: liées aux applications. ex: moyenne des salaires < 10k
- intra-relation: met en jeu 1 seul relation. ex: attribut non nul
- inter-relation: plusieursrelation. intégrité réferentielle

**CI Intra-relation**
- ci domaine: "le degré d'un vin vrie entre 5 et 15 degrés"
- ci non nullite: "toute valeur d'un numér de vin est connue"
- ci unicité de clé: "tous les numéro de vin sont différents"
- dépendance fonction: le numéro de vin détermine le cru
- temporelle: le degré d'un vin ne peut pas décroître
- agregat: la moyenne des quantités de vin bu par tt le monde ne doit pas dépacer 10 litres ( average {all quatities drank/person} < 10l )

**CI inter-relation**
- ci référentielle: clé étrangère
- ci inclusion: l'ensemble des valeurs d'une colonne d'une relation est inclus dans l'ensemble des valeurs d'une colonne d'une autre relation. ex: toute les ville d'un buveur et une ville d'un producteur (= si 2 clés étrangères référence la même colonne elles ont le même ensemble de valeur? normal)
- ci générale: ex: la somme des quantités livrées à un buveur est inférieur ou égale à la somme des quantités commandées par ce buveur ( sum(delivered) = sum(ordered) )

**CI integrité référentielle**
- un attribut ou groupe d'attributs d'une relation apparaît comme clé dans une autre relaion (ouai donc = y a une clé étrangère). ex: une récolte doit référencer un vin et un producteur existant

certaines CI sont offertent par les sgbd (non nullité, unicité des valeurs) mais la plupart doivent être faites par les programmes


**NORME SQL**

-> SQL 86 : unicité de valeur, non nullité,

vue avec "check option"

NOM Char(20) unique  not  null

-> SQL 89 : domaine, clé, intégrité référentielle avec "rejet"

create  table  EMPLOYE (
NOM Char(20) primary  key,
SAL Integer check  (SAL > 0),
DPT Char(20) references  DEPART )
create  table  DEPART (
NOM Char(20) primary  key,
NB_E Integer check  (NB_E > 0) )

-> SQL2 : intégrité référentielle
avec "cascade  delete  et  update"

create  table  EMPLOYE (
NOM Char(20) primary  key,
SAL Integer check  (SAL > 0),
DPT Char(20) references  DEPART
on  delete  cascade  )


**potentiels problèmes avec les CI**
cohérence: elles ne doivent pas être contradictoires
redondance: ex: degré <15 et degré <14
optimisation: il faut impliquer le moins de données possible pour la verif ou verif uniquement certains types d'updates


# Fonctions / procédures stockées

# curseurs / Dataset :

- explicite
CURSOR <nom_du_curseur> IS  
SELECT ... ;

OPEN <nom_du_curseur> ;
FETCH <nom_du_curseur> INTO <variable1>,<variable2> ;

%ROWCOUNT permet de savoir le numéro de ligne à laquelle on est

CLOSE <nom_du_curseur> ;

exemple 

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


On peut récupérer des informations sur un curseur par l'intermédiaire des attributs suivants :  
<nom_du_curseur>%ISOPEN : retourne TRUE ou FALSE.  
<nom_du_curseur>%FOUND : retourne TRUE ou FALSE suivant que le dernier FETCH a retourné une ligne ou non.  
<nom_du_curseur>%NOTFOUND : retourne TRUE ou FALSE à l'inverse de %FOUND  
<nom_du_curseur>%ROWCOUNT : retourne le nombre de FETCH ayant renvoyé une ligne. Avant le premier FETCH, cet attribut renvoie 0.


exemple d'un while avec un not found

OPEN moncurseur;
LOOP
 FETCH moncurseur INTO variable1,variable2;
 EXIT WHEN moncurseur%NOTFOUND;
END LOOP;
CLOSE moncurseur;


Il existe une syntaxe qui permet de se passer des instructions OPEN, FETCH et CLOSE, le for each :

DECLARE  
CURSOR moncurseur IS  
SELECT champ1,champ2 FROM table;  
variable1 table.champ1%TYPE;  
variable2 table.champ2%TYPE;  
BEGIN  
FOR maligne IN moncurseur LOOP  
...  
END LOOP;  
END;


on peut aussi créer un curseur implicitement en le déclarant dans la boucle

DECLARE  
variable1 table.champ1%TYPE;  
variable2 table.champ2%TYPE;  
BEGIN  
FOR maligne IN (SELECT champ1,champ2 FROM table) LOOP  
...  
END LOOP;  
END;

quand on update il est préférable d'utiliser la synthaxe suivante qui vérouille les lignes:

DECLARE  
CURSOR moncurseur IS  
SELECT champ1,champ2 FROM table  
FOR UPDATE OF champ1,champ2;

pour les updateet delete on peut utiliser WHERE CURRENT OF <\nom_du_curseur> qui permet de préciser que la maj ou delete s'applique à la ligne en cours du curseur


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

on peut passer des param au curseur :

DECLARE  
CURSOR moncurseur (param table.champ3%TYPE) IS  
SELECT champ1,champ2 FROM table  
WHERE champ3 = param;



- implicite

# exceptions

pour éviter un crash à la première erreur on prévoit des exceptions. Avec pl/sql on peut gérer des exception dans un bloc


# WS
